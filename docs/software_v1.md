## Brief Intro and Technical Details

This page outlines the first attempt at developing a python app to be controlled by the user via a GUI. 

Here is a quick overview of the software:

* Scanning Mode &rarr; Scans all objects that need to be sorted, with a list of unique objects stored in a .csv file.
* Sorting Mode &rarr; Sorts the objects

<div><br></div>

Here is a quick overview of the key features:

* Object Detection &rarr; Done by utilizing webcam and YOLO; users can select a PyTorch model to be used
* Object Logging &rarr; Stores list of unique objects in csv to be read by Arduino; objects also stored in Arduino's EEPROM in case system powers off
* User Control &rarr; User dictates when the software does what
* Arduino Recognition &rarr; Software knows when Arduino is connected and will not start scanning or sorting until then
* Error Handling &rarr; Some error checking is in place

<div><br></div>

How does this compare to the requirements laid out on the milestones page?


| Requirement from Milestone 3 | Status | Notes (if needed) |
|----------|----------|----------|
| Be intuitive to use, and look somewhat polished  | ❌  | GUI is intuitive, but very dated  |
| Allow user to perform basic functions  | 🔄  | Detection and scanning good, but sorting framework not set in place  |
| Allow user control over the process  | ✅  | User dictates entire process, minus the actual object detection  |
| At the minimum, recognize when an Arduino is connected via USB to laptop  | 🔄  | System can recognize AND communicate with Arduino, but framework to sort is not in place  |
| Leverage webcam to detect objects | ✅  | Utilizes YOLO and webcam to detect objects, with bounding boxes serving as visual confirmation  |
| Have some error handling in place and prevent user from going too far out of order  | ✅  | Error handling present  |

<div><br></div>

What is missing with this version?

* Object Sorting Theory &rarr; Sorting Mode can be engaged but beyond that, there is not much structure for how the objects will be sorted. Binary sorting makes the most sense in this application and will be developed in the next version.
* User Interface &rarr; Interface using tkinter for Python was a great idea, but has a very dated appearance 
* Arduino Functionality &rarr; Arduino is equipped to identify when sorting has started, but does not have the proper framework in place to be able to control motors.

## Code

Since this is an intermediate version of the software, with a better version in place, the code for the GUI and Arduino will be provided here as well as screenshots of the appearance of the software. A video of the software working will be provided on the next version's page, with error handling, user control, etc on full display. 

Python Code:
``` py linenums="1"
import tkinter as tk
from tkinter import ttk, filedialog
from PIL import Image, ImageTk
import cv2
from ultralytics import YOLO
import csv
import os
import serial
import threading
from datetime import datetime
import serial.tools.list_ports
import time
import csv
from PIL import ImageDraw, ImageFont

# ---------------------------
# CONFIGURATION
# ---------------------------
CAMERA_INDEX = 0
WINDOW_TITLE = "YOLO-Based Binary Object Sorting System V1"
WINDOW_BG_COLOR = "#1e1e1e"
FRAME_RATE_MS = 15
LOG_DIR = "logs"

# ---------------------------
# Ensure Logs Folder Exists
# ---------------------------
os.makedirs(LOG_DIR, exist_ok=True)

def generate_csv_name():
    now = datetime.now()
    return f"{LOG_DIR}/scan_log_{now.strftime('%Y-%m-%d_%H-%M-%S')}.csv"

# ---------------------------
# Connection to Arduino
# ---------------------------

def find_arduino_port():
    ports = serial.tools.list_ports.comports()
    for port in ports:
        if "Leonardo" in port.description or (port.vid == 0x2341 and port.pid == 0x8036):
            return port.device
    return None

def reset_arduino_before_exit(port):
    try:
        ser = serial.Serial(port, 1200)
        ser.close()
        print("🛑 Arduino reset triggered")
        time.sleep(2)  # wait for Arduino to fully reset
    except Exception as e:
        print(f"⚠️ Could not reset Arduino: {e}")

def get_unique_classes_from_csv(path):
    unique_classes = set()
    try:
        with open(path, newline='') as f:
            reader = csv.reader(f)
            for row in reader:
                if len(row) >= 2:
                    unique_classes.add(row[1])
    except Exception as e:
        print(f"Error reading CSV: {e}")
    return list(unique_classes)

def send_objects_to_eeprom(item_list, arduino_serial):
    try:
        # Create comma-separated string
        object_string = ",".join(item_list[:20])  # Limit to 20 items
        
        # Send the command
        command = f"STORE_OBJECTS:{object_string}\n"
        arduino_serial.write(command.encode())
        
        print(f"✅ Sent {len(item_list)} objects to Arduino EEPROM")
        return True
        
    except Exception as e:
        print(f"❌ Failed to send objects: {e}")
        return False

# ---------------------------
# GUI Class
# ---------------------------
class YOLOApp:
    def __init__(self, root):
        self.root = root
        self.root.title(WINDOW_TITLE)
        self.root.configure(bg=WINDOW_BG_COLOR)
        self.arduino = None

        self.cap = cv2.VideoCapture(CAMERA_INDEX)
        if not self.cap.isOpened():
            raise RuntimeError("❌ Cannot open webcam")

        self.is_running = False
        self.mode = None  # 'scan' or 'sort'
        self.csv_file = None
        self.detected_classes = set()
        self.model_path = ""
        self.model = None
        self.arduino_ready_to_sort = False

        # Layout frames
        main_frame = tk.Frame(root, bg=WINDOW_BG_COLOR)
        main_frame.pack(fill=tk.BOTH, expand=True)

        left_frame = tk.Frame(main_frame, bg=WINDOW_BG_COLOR)
        left_frame.pack(side=tk.LEFT, padx=10, pady=10)

        right_frame = tk.Frame(main_frame, bg=WINDOW_BG_COLOR)
        right_frame.pack(side=tk.RIGHT, padx=10, pady=10, fill=tk.Y)

        # Video Frame
        self.video_frame = tk.Label(left_frame)
        self.video_frame.pack()

        # Create a black placeholder with text "Camera Offline"
        placeholder = Image.new("RGB", (640, 480), (0, 0, 0))
        draw = ImageDraw.Draw(placeholder)

        try:
            font = ImageFont.truetype("arial.ttf", 36)
        except:
            font = ImageFont.load_default()

        text = "CAMERA OFFLINE"
        bbox = draw.textbbox((0, 0), text, font=font)
        text_width = bbox[2] - bbox[0]
        text_height = bbox[3] - bbox[1]
        position = ((640 - text_width) // 2, (480 - text_height) // 2)
        draw.text(position, text, fill=(200, 200, 200), font=font)

        self.placeholder_img = ImageTk.PhotoImage(placeholder)
        self.video_frame.configure(image=self.placeholder_img)
        self.video_frame.imgtk = self.placeholder_img

        # Control Buttons
        btn_frame = tk.Frame(left_frame, bg=WINDOW_BG_COLOR)
        btn_frame.pack(pady=10)

        self.choose_model_btn = ttk.Button(btn_frame, text="Choose PyTorch Model", command=self.choose_model)
        self.choose_model_btn.grid(row=0, column=0, padx=5)

        self.arduino_btn = ttk.Button(btn_frame, text="Connect to Arduino", command=self.connect_to_arduino)
        self.arduino_btn.grid(row=0, column=1, padx=5)

        self.scan_btn = ttk.Button(btn_frame, text="📸 Start Scanning Mode", command=self.start_scanning_mode)
        self.scan_btn.grid(row=0, column=2, padx=5)

        self.sort_btn = ttk.Button(btn_frame, text="⚙️ Start Sorting Mode", command=self.start_sorting_mode)
        self.sort_btn.grid(row=0, column=3, padx=5)

        self.stop_btn = ttk.Button(btn_frame, text="⏹ Stop", command=self.stop_detection)
        self.stop_btn.grid(row=0, column=4, padx=5)

        # Second row of buttons
        self.send_list_btn = ttk.Button(btn_frame, text="📂 Send Object List to Arduino", command=self.send_object_list)
        self.send_list_btn.grid(row=1, column=0, columnspan=2, pady=10, padx=5)

        self.list_objects_btn = ttk.Button(btn_frame, text="📋 List Stored Objects", command=self.list_stored_objects)
        self.list_objects_btn.grid(row=1, column=2, padx=5)

        self.clear_objects_btn = ttk.Button(btn_frame, text="🗑️ Clear Objects", command=self.clear_stored_objects)
        self.clear_objects_btn.grid(row=1, column=3, padx=5)

        exit_button = ttk.Button(btn_frame, text="Exit", command=self.exit_app)
        exit_button.grid(row=1, column=4, padx=5)

        # Output Log
        self.log_text = tk.Text(right_frame, width=40, height=30, bg="black", fg="white", wrap=tk.WORD)
        self.log_text.pack(fill=tk.BOTH, expand=True)
        self.log("🟢 GUI Initialized")

        # ---------------------------
        # 🎨 Button Styling
        # ---------------------------
        style = ttk.Style()
        style.theme_use("default")
        style.configure("TButton", background="#444", foreground="white", padding=6, font=("Segoe UI", 10, "bold"))
        style.map("TButton", background=[("active", "#777")])

    # ---------------------------
    # 🧾 Log Output
    # ---------------------------
    def log(self, message):
        timestamp = datetime.now().strftime("[%H:%M:%S]")
        self.log_text.insert(tk.END, f"{timestamp} {message}\n")
        self.log_text.see(tk.END)

    # ---------------------------
    # 🚦 MODE SWITCHING
    # ---------------------------
    def start_scanning_mode(self):
        if self.mode == "scan":
            self.stop_detection()
            return

        if self.model is None:
            self.log("⚠️ No model selected")
            return
        if self.arduino is None or not self.arduino.is_open:
            self.log("⚠️ Arduino not connected")
            return

        self.stop_detection()
        self.mode = "scan"
        self.scan_btn.config(text="⏹ Stop Scanning Mode", state=tk.NORMAL)
        self.sort_btn.config(state=tk.DISABLED)
        self.csv_file = generate_csv_name()
        self.detected_classes.clear()
        self.log(f"📄 Logging to: {self.csv_file}")
        self.is_running = True
        self.update_frame()

    def start_sorting_mode(self):
        if self.mode == "sort":
            self.stop_detection()
            return
        if self.model is None:
            self.log("⚠️ No model selected")
            return
        if self.arduino is None or not self.arduino.is_open:
            self.log("⚠️ Arduino not connected")
            return
        if not self.arduino_ready_to_sort:
            self.log("❌ Arduino is not ready. Send object list first.")
            return
        self.stop_detection()
        self.mode = "sort"
        self.sort_btn.config(text="⏹ Stop Sorting Mode", state=tk.NORMAL)
        self.scan_btn.config(state=tk.DISABLED)
        self.log("⚙️ Sorting mode active")
        self.is_running = True
        self.update_frame()

    def stop_detection(self):
        self.is_running = False
        was_sorting = (self.mode == "sort")  # Remember if we were sorting
        self.mode = None
        self.video_frame.configure(image=self.placeholder_img)
        self.video_frame.imgtk = self.placeholder_img
        self.scan_btn.config(text="📸 Start Scanning Mode", state=tk.NORMAL)
        self.sort_btn.config(text="⚙️ Start Sorting Mode", state=tk.NORMAL)
        
        # Send stop command to Arduino
        self.send_to_arduino("stop")
        
        if was_sorting:
            self.log("🛑 Detection stopped - Sorting mode disabled")
        else:
            self.log("🛑 Detection stopped")

    def choose_model(self):
        if self.is_running:
            self.log("⚠️ Stop detection before loading a new model")
            return
        path = filedialog.askopenfilename(filetypes=[("PyTorch Model", "*.pt")])
        if path:
            self.model_path = path
            self.model = YOLO(self.model_path)
            self.log(f"✅ Model loaded: {self.model_path}")

    # ---------------------------
    # Frame Processing
    # ---------------------------
    def update_frame(self):
        if not self.is_running:
            return

        ret, frame = self.cap.read()
        if not ret:
            self.log("❌ Failed to grab frame - stopping detection")
            self.stop_detection()  # Auto-stop if camera fails
            return

        results = self.model(frame)[0]
        annotated = results.plot()

        if self.mode == "scan":
            self.handle_scanning_mode(results)
        elif self.mode == "sort":
            self.handle_sorting_mode(results)

        rgb_frame = cv2.cvtColor(annotated, cv2.COLOR_BGR2RGB)
        img = Image.fromarray(rgb_frame)
        imgtk = ImageTk.PhotoImage(image=img)
        self.video_frame.imgtk = imgtk
        self.video_frame.configure(image=imgtk)

        self.root.after(FRAME_RATE_MS, self.update_frame)

    # ---------------------------
    # Scanning Mode Logic
    # ---------------------------
    def handle_scanning_mode(self, results):
        self.send_to_arduino("scan")
        for box in results.boxes:
            cls_id = int(box.cls[0])
            class_name = self.model.names[cls_id]
            if class_name not in self.detected_classes:
                self.detected_classes.add(class_name)
                with open(self.csv_file, 'a', newline='') as f:
                    writer = csv.writer(f)
                    writer.writerow([datetime.now().isoformat(), class_name])
                self.log(f"📝 Logged: {class_name}")

    # ---------------------------
    # Sorting Mode Logic
    # ---------------------------
    def handle_sorting_mode(self, results):
        self.send_to_arduino("sort")
        for box in results.boxes:
            cls_id = int(box.cls[0])
            class_name = self.model.names[cls_id]
            self.log(f"📦 Detected for sorting: {class_name}")
            # Send the detected class to Arduino
            self.send_to_arduino(class_name)
            break

    # ---------------------------
    # Arduino Communication
    # ---------------------------

    def connect_to_arduino(self):
        port = find_arduino_port()
        if port:
            try:
                self.arduino = serial.Serial(port, 9600, timeout=2)
                self.log(f"🟡 Connecting to Arduino on {port}...")

                time.sleep(2)  # Allow time for Arduino to reset and send handshake

                # Wait for "READY"
                start_time = time.time()
                ready_line = ""
                while time.time() - start_time < 5:
                    if self.arduino.in_waiting > 0:
                        ready_line = self.arduino.readline().decode().strip()
                        if ready_line == "READY":
                            break
                        elif ready_line:  # Any other message from Arduino
                            self.log(f"Arduino: {ready_line}")

                if ready_line == "READY":
                    self.log("✅ Arduino is ready!")
                    self.arduino_btn.config(text="Arduino Connected!", state=tk.DISABLED)
                    
                    # Start listening thread for Arduino messages
                    self.start_arduino_listener()
                else:
                    self.log(f"❌ Unexpected handshake message: {ready_line}")
            except serial.SerialException as e:
                self.log(f"❌ Connection Failed: {str(e)}")
        else:
            self.log("⚠️ No Arduino found!")

    def start_arduino_listener(self):
        """Start a background thread to listen for Arduino messages"""
        def listen():
            while self.arduino and self.arduino.is_open:
                try:
                    if self.arduino.in_waiting > 0:
                        message = self.arduino.readline().decode().strip()
                        if message:
                            if message == "READY_TO_SORT":
                                self.arduino_ready_to_sort = True
                                self.log("🟢 Arduino ready to sort!")
                            else:
                                self.log(f"Arduino: {message}")
                    time.sleep(0.1)
                except Exception as e:
                    if self.arduino and self.arduino.is_open:
                        self.log(f"Error reading from Arduino: {e}")
                    break
        
        listener_thread = threading.Thread(target=listen, daemon=True)
        listener_thread.start()

    def send_to_arduino(self, message):
        try:
            if self.arduino and self.arduino.is_open:
                self.arduino.write((message + "\n").encode())
        except Exception as e:
            self.log(f"❌ Error sending to Arduino: {e}")
    
    def send_object_list(self):
        """Send object list to Arduino EEPROM"""
        if self.arduino is None or not self.arduino.is_open:
            self.log("⚠️ Arduino must be connected before sending list")
            return

        filepath = filedialog.askopenfilename(filetypes=[("CSV Files", "*.csv")])
        if not filepath:
            self.log("⚠️ No file selected")
            return

        classes = get_unique_classes_from_csv(filepath)
        if not classes:
            self.log("⚠️ No valid objects found in CSV")
            return

        self.log(f"📤 Sending {len(classes)} objects to Arduino EEPROM...")
        
        if send_objects_to_eeprom(classes, self.arduino):
            self.log(f"✅ Sent {len(classes)} objects: {', '.join(classes)}")
        else:
            self.log("❌ Send failed")

    def list_stored_objects(self):
        """Ask Arduino to list stored objects"""
        if self.arduino and self.arduino.is_open:
            self.send_to_arduino("LIST_OBJECTS")
        else:
            self.log("⚠️ Arduino not connected")

    def clear_stored_objects(self):
        """Clear objects from Arduino EEPROM"""
        if self.arduino and self.arduino.is_open:
            self.send_to_arduino("CLEAR_OBJECTS")
            self.arduino_ready_to_sort = False
            self.log("🗑️ Cleared stored objects")
        else:
            self.log("⚠️ Arduino not connected")
    
    def exit_app(self):
        if self.arduino and self.arduino.is_open:
            port = self.arduino.port
            self.arduino.close()
            self.clear_stored_objects()
            reset_arduino_before_exit(port)
        self.root.destroy()

# ---------------------------
# Run GUI
# ---------------------------
if __name__ == "__main__":
    root = tk.Tk()
    app = YOLOApp(root)
    root.mainloop()

    if app.cap.isOpened():
        app.cap.release()
    cv2.destroyAllWindows()
```

Arduino Code:
``` cpp linenums="1"
#include <Arduino.h>
#include <EEPROM.h>

const uint8_t MAX_ITEMS = 30;
const uint8_t MAX_STR_LEN = 50;
const int EEPROM_START_ADDR = 0;
const int NUM_ITEMS_ADDR = 0;
const int ITEMS_START_ADDR = 1;

String storedItems[MAX_ITEMS];
uint8_t numStoredItems = 0;
String currentMode = "";
String lastModeMessage = "";
unsigned long lastModeTime = 0;
const unsigned long MODE_MESSAGE_COOLDOWN = 5000; // 5 seconds between mode messages
bool itemsLoaded = false;

void storeObjectsInEEPROM(String objects[], uint8_t count) {
  // Store number of items first
  EEPROM.write(NUM_ITEMS_ADDR, count);
  
  int addr = ITEMS_START_ADDR;
  
  for (uint8_t i = 0; i < count; i++) {
    String item = objects[i];
    
    // Store length of string
    uint8_t len = min(item.length(), (unsigned int)(MAX_STR_LEN - 1));
    EEPROM.write(addr, len);
    addr++;
    
    // Store the string characters
    for (uint8_t j = 0; j < len; j++) {
      EEPROM.write(addr, item[j]);
      addr++;
    }
    
    // Fill remaining space with zeros
    for (uint8_t j = len; j < MAX_STR_LEN - 1; j++) {
      EEPROM.write(addr, 0);
      addr++;
    }
  }
  
  Serial.println("✅ Objects stored in EEPROM");
}

void loadObjectsFromEEPROM() {
  numStoredItems = EEPROM.read(NUM_ITEMS_ADDR);
  
  // Sanity check
  if (numStoredItems > MAX_ITEMS) {
    numStoredItems = 0;
    Serial.println("⚠️ Invalid EEPROM data, resetting");
    return;
  }
  
  if (numStoredItems == 0) {
    Serial.println("📭 No objects stored in EEPROM");
    return;
  }
  
  int addr = ITEMS_START_ADDR;
  
  for (uint8_t i = 0; i < numStoredItems; i++) {
    uint8_t len = EEPROM.read(addr);
    addr++;
    
    if (len >= MAX_STR_LEN) len = 0; // Corrupted data
    
    String item = "";
    for (uint8_t j = 0; j < len; j++) {
      char c = EEPROM.read(addr);
      if (c != 0) item += c;
      addr++;
    }
    
    // Skip remaining bytes for this item
    addr += (MAX_STR_LEN - 1 - len);
    
    storedItems[i] = item;
  }
  
  if (numStoredItems > 0) {
    itemsLoaded = true;
    Serial.println("📥 Objects loaded from EEPROM:");
    for (uint8_t i = 0; i < numStoredItems; i++) {
      Serial.print("  ");
      Serial.println(storedItems[i]);
    }
    Serial.println("READY_TO_SORT");
  }
}

String lastSortedItem = "";
unsigned long lastSortTime = 0;
const unsigned long SORT_COOLDOWN = 2000; // 2 seconds between same item sorts

void handleSorting(String itemClass) {
  unsigned long currentTime = millis();
  
  // Only process if it's a different item or enough time has passed
  if (itemClass != lastSortedItem || (currentTime - lastSortTime) > SORT_COOLDOWN) {
    Serial.print("🔄 Sorting item: ");
    Serial.println(itemClass);

    if (itemClass == "bottle") {
      // trigger servo for bottle
      Serial.println("→ Sorting bottle");
    } else if (itemClass == "can") {
      // another action for can
      Serial.println("→ Sorting can");
    } else if (itemClass == "apple") {
      // action for apple
      Serial.println("→ Sorting apple");
    } else {
      Serial.println("❓ Unknown class, default action");
    }
    
    lastSortedItem = itemClass;
    lastSortTime = currentTime;
  }
}

void processObjectList(String objectListString) {
  // Parse comma-separated string
  String tempItems[MAX_ITEMS];
  uint8_t count = 0;
  
  int startIdx = 0;
  int commaIdx = objectListString.indexOf(',');
  
  while (commaIdx != -1 && count < MAX_ITEMS) {
    tempItems[count] = objectListString.substring(startIdx, commaIdx);
    tempItems[count].trim();
    count++;
    
    startIdx = commaIdx + 1;
    commaIdx = objectListString.indexOf(',', startIdx);
  }
  
  // Get the last item (or the only item if no commas)
  if (startIdx < objectListString.length() && count < MAX_ITEMS) {
    tempItems[count] = objectListString.substring(startIdx);
    tempItems[count].trim();
    count++;
  }
  
  if (count > 0) {
    storeObjectsInEEPROM(tempItems, count);
    loadObjectsFromEEPROM(); // Reload to confirm storage worked
  }
}

void setup() {
  Serial.begin(9600);
  while (!Serial);
  delay(1000);
  
  Serial.println("🤖 Arduino Object Sorter Starting...");
  
  // Try to load existing objects from EEPROM
  loadObjectsFromEEPROM();
  
  Serial.println("READY");
}

void loop() {
  if (Serial.available()) {
    String input = Serial.readStringUntil('\n');
    input.trim();

    if (input.startsWith("STORE_OBJECTS:")) {
      // Extract the object list from the command
      String objectList = input.substring(14); // Remove "STORE_OBJECTS:" prefix
      Serial.print("📝 Storing objects: ");
      Serial.println(objectList);
      processObjectList(objectList);
      
    } else if (input == "LOAD_OBJECTS") {
      loadObjectsFromEEPROM();
      
    } else if (input == "LIST_OBJECTS") {
      if (itemsLoaded && numStoredItems > 0) {
        Serial.println("📋 Current object list:");
        for (uint8_t i = 0; i < numStoredItems; i++) {
          Serial.print("  ");
          Serial.print(i + 1);
          Serial.print(": ");
          Serial.println(storedItems[i]);
        }
      } else {
        Serial.println("📭 No objects stored");
      }
      
    } else if (input == "CLEAR_OBJECTS") {
      EEPROM.write(NUM_ITEMS_ADDR, 0);
      numStoredItems = 0;
      itemsLoaded = false;
      Serial.println("🗑️ Object list cleared");
      
    } else if (input == "scan") {
      currentMode = "scan";
      unsigned long currentTime = millis();
      if (lastModeMessage != "scan" || (currentTime - lastModeTime) > MODE_MESSAGE_COOLDOWN) {
        Serial.println("📸 Scan mode active");
        lastModeMessage = "scan";
        lastModeTime = currentTime;
      }
      
    } else if (input == "sort") {
      currentMode = "sort";
      unsigned long currentTime = millis();
      if (itemsLoaded) {
        if (lastModeMessage != "sort" || (currentTime - lastModeTime) > MODE_MESSAGE_COOLDOWN) {
          Serial.println("⚙️ Sort mode active");
          lastModeMessage = "sort";
          lastModeTime = currentTime;
        }
      } else {
        Serial.println("❌ Cannot sort - no objects loaded");
      }
      
    } else if (input == "stop") {
      currentMode = "";
      Serial.println("⏹️ Stopped");
      lastModeMessage = "";  // Reset so next mode change shows message
      
    } else if (currentMode == "sort" && itemsLoaded) {
      // This would be a detected object class from Python
      handleSorting(input);
      
    } else {
      Serial.print("❓ Unknown command: ");
      Serial.println(input);
    }
  }
}
```

## Results

Image shown below, with appearance and some error handling shown:

![Results Software V1](./assets/Screenshot%202025-08-10%20125031.png)