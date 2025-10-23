## Brief Intro and Technical Details

Here is a quick overview of the software:

* Scanning Mode &rarr; Scans all objects that need to be sorted, with a list of unique objects stored in a .csv file.
* Sorting Mode &rarr; Sorts the objects in a binary fashion, which means that system looks for a target object in a set number of passes. Refer to video for demo of this.

<div><br></div>

Here is a quick overview of the key features:

* Object Detection &rarr; Done by utilizing webcam and YOLO; users can select a PyTorch model to be used
* Object Logging &rarr; Stores list of unique objects in csv to be read by Arduino; objects also stored in Arduino's EEPROM in case system powers off
* User Control &rarr; User dictates when the software does what
* Arduino Recognition &rarr; Software knows when Arduino is connected and will not start scanning or sorting until then
* Arduino Echoing &rarr; Arduino send an echo of messages to log to provide updates to user, so that Serial Monitor doesn't have to be open
* Error Handling &rarr; More information below

<div><br></div>

Error Handling:

* User must select PyTorch model to be able to enter sorting/scanning mode
* User must connect to an Arduino AND serial communication must be established to be able to enter sorting/scanning mode
* Sorting can only take place if a model has been selected and a list of objects has been sent
* etc...

<div><br></div>

What is missing with this version?

* Motors &rarr; Motor selection not 100% finalized at this stage, values will be inputted once motors are finalized

Note that software does not perform 100% perfectly. However, for the purposes of this project, it is enough to be able to proceed, as it performs basic functions up to a standard acceptable for a prototype. The best example of this is the fact that a sorting session has to be started BEFORE actually engaging sorting mode, which will take a bit of getting used to for an end user.

<div><br></div>

How does this compare to the requirements laid out on the milestones page?


| Requirement from Milestone 3 | Status | Notes (if needed) |
|----------|----------|----------|
| Be intuitive to use, and look somewhat polished  | ✅  | Used customtkinter to achieve a more modern look; mostly polished appearance  |
| Allow user to perform basic functions  | ✅  | System can be used to detect and scan, with framework in place to sort  |
| Allow user control over the process  | ✅  | User dictates entire process, minus the actual object detection  |
| At the minimum, recognize when an Arduino is connected via USB to laptop  | ✅  | System can recognize AND communicate with Arduino  |
| Leverage webcam to detect objects | ✅  | Utilizes YOLO and webcam to detect objects, with bounding boxes serving as visual confirmation  |
| Have some error handling in place and prevent user from going too far out of order  | ✅  | Error handling talked about above  |

## Code

```py title="appv2.py" linenums="1"
import customtkinter as ctk
from tkinter import filedialog, messagebox
from customtkinter import CTkImage
# import tkinter as tk
# from tkinter import ttk, filedialog, messagebox
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
# ⚙️ CONFIGURATION
# ---------------------------
CAMERA_INDEX = 0
WINDOW_TITLE = "YOLO-Based Binary Object Sorting System V2"
WINDOW_BG_COLOR = "#1e1e1e"
FRAME_RATE_MS = 15
LOG_DIR = "logs"

# ---------------------------
# 📁 Ensure Logs Folder Exists
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
    """
    Send object list to Arduino for EEPROM storage
    Much simpler than SerialTransfer!
    """
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
# 🖼 GUI Class
# ---------------------------
class YOLOApp:
    def __init__(self, root):
        self.root = root
        self.root.title(WINDOW_TITLE)
        self.root.configure(bg_color=WINDOW_BG_COLOR)
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

        # Binary Sorting Algorithm Variables
        self.sort_classes = []  # List of classes to sort through
        self.current_sort_index = 0  # Which class we're currently sorting
        self.current_target_class = None  # The class we're looking for in this pass
        self.sorting_in_progress = False  # Whether we're actively sorting
        self.waiting_for_next_pass = False  # Whether we're waiting for user to start next pass

        # Layout frames
        main_frame = ctk.CTkFrame(root, bg_color=WINDOW_BG_COLOR)
        main_frame.pack(fill=ctk.BOTH, expand=True)

        left_frame = ctk.CTkFrame(main_frame, bg_color=WINDOW_BG_COLOR)
        left_frame.pack(side=ctk.LEFT, padx=10, pady=10)

        right_frame = ctk.CTkFrame(main_frame, bg_color=WINDOW_BG_COLOR)
        right_frame.pack(side=ctk.RIGHT, padx=10, pady=10, fill=ctk.BOTH, expand=True)

        # Video Frame
        self.video_frame = ctk.CTkLabel(left_frame)
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

        self.placeholder_img = CTkImage(light_image=placeholder, dark_image=placeholder, size=(640, 480))
        self.video_frame.configure(image=self.placeholder_img, text="")  # text="" prevents text overlay
        self.video_frame.imgtk = self.placeholder_img  # Keep a reference

        # Control Buttons
        btn_frame = ctk.CTkFrame(left_frame, bg_color=WINDOW_BG_COLOR)
        btn_frame.pack(pady=10)

        self.choose_model_btn = ctk.CTkButton(btn_frame, text="Choose PyTorch Model", command=self.choose_model)
        self.choose_model_btn.grid(row=0, column=0, padx=5)

        self.arduino_btn = ctk.CTkButton(btn_frame, text="Connect to Arduino", command=self.connect_to_arduino)
        self.arduino_btn.grid(row=0, column=1, padx=5)

        self.scan_btn = ctk.CTkButton(btn_frame, text="📸 Start Scanning Mode", command=self.start_scanning_mode)
        self.scan_btn.grid(row=0, column=2, padx=5)

        self.sort_btn = ctk.CTkButton(btn_frame, text="⚙️ Start Sorting Mode", command=self.start_sorting_mode)
        self.sort_btn.grid(row=0, column=3, padx=5)

        self.stop_btn = ctk.CTkButton(btn_frame, text="⏹ Stop", command=self.stop_detection)
        self.stop_btn.grid(row=0, column=4, padx=5)

        # Second row of buttons - MERGED FUNCTIONALITY
        self.list_objects_btn = ctk.CTkButton(btn_frame, text="📋 List Stored Objects", command=self.list_stored_objects)
        self.list_objects_btn.grid(row=1, column=0, padx=5)

        self.clear_objects_btn = ctk.CTkButton(btn_frame, text="🗑️ Clear Objects", command=self.clear_stored_objects)
        self.clear_objects_btn.grid(row=1, column=1, padx=5)

        # NEW: Merged sorting button that handles everything
        self.start_sorting_session_btn = ctk.CTkButton(btn_frame, text="🚀 Start Sorting Session", 
                                                   command=self.start_sorting_session)
        self.start_sorting_session_btn.grid(row=1, column=2, columnspan=2, pady=10, padx=5)

        exit_button = ctk.CTkButton(btn_frame, text="Exit", command=self.exit_app)
        exit_button.grid(row=1, column=4, padx=5)

        # Third row - Binary Sorting Controls (simplified)
        sort_control_frame = ctk.CTkFrame(btn_frame, bg_color=WINDOW_BG_COLOR)
        sort_control_frame.grid(row=2, column=0, columnspan=5, pady=10)

        self.next_pass_btn = ctk.CTkButton(sort_control_frame, text="➡️ Next Pass", command=self.start_next_pass, state=ctk.DISABLED)
        self.next_pass_btn.grid(row=0, column=0, padx=5)

        self.finish_sort_btn = ctk.CTkButton(sort_control_frame, text="✅ Finish Sorting", command=self.finish_sorting, state=ctk.DISABLED)
        self.finish_sort_btn.grid(row=0, column=1, padx=5)

        # Sorting Status Display
        self.sort_status_label = ctk.CTkLabel(
            sort_control_frame,
            text="Arduino not connected",
            text_color="red",
            fg_color="transparent",
            bg_color=WINDOW_BG_COLOR,
            font=("Segoe UI", 10, "bold")
        )
        self.sort_status_label.grid(row=1, column=0, columnspan=2, pady=5)

        # Output Log
        self.log_text = ctk.CTkTextbox(
            right_frame,
            width=400,         # Increased width
            height=300,        # Increased height
            fg_color="black",  # Background color
            text_color="white",# Fix white-on-white issue
            wrap="word"
        )       
        self.log_text.pack(fill=ctk.BOTH, expand=True, padx=5, pady=5)
        self.log("🟢 GUI Initialized")

        # ---------------------------
        # 🎨 Button Styling
        # ---------------------------
        # style = ttk.Style()
        # style.theme_use("default")
        # style.configure("TButton", background="#444", foreground="white", padding=6, font=("Segoe UI", 10, "bold"))
        # style.map("TButton", background=[("active", "#777")])

    # ---------------------------
    # 🧾 Log Output
    # ---------------------------
    def log(self, message):
        timestamp = datetime.now().strftime("[%H:%M:%S]")
        self.log_text.insert(ctk.END, f"{timestamp} {message}\n")
        self.log_text.see(ctk.END)

    # ---------------------------
    # 🚀 NEW: Merged Sorting Session Function
    # ---------------------------
    def start_sorting_session(self):
        """
        Merged function that handles:
        1. Model validation
        2. File selection (single dialog)
        3. Sending objects to Arduino
        4. Setting up binary sort
        5. Starting camera/sorting mode
        """
        # Validation checks
        if self.model is None:
            self.log("❌ No model selected. Please choose a PyTorch model first.")
            messagebox.showerror("Error", "Please select a PyTorch model before starting sorting session.")
            return
            
        if self.arduino is None or not self.arduino.is_open:
            self.log("❌ Arduino not connected. Please connect Arduino first.")
            messagebox.showerror("Error", "Please connect Arduino before starting sorting session.")
            return

        # Single file dialog for CSV with detected objects
        filepath = filedialog.askopenfilename(
            title="Select CSV with detected objects for sorting",
            filetypes=[("CSV Files", "*.csv")]
        )
        if not filepath:
            self.log("⚠️ No file selected - sorting session cancelled")
            return

        classes = get_unique_classes_from_csv(filepath)
        if not classes:
            self.log("⚠️ No valid classes found in CSV")
            messagebox.showerror("Error", "No valid object classes found in the selected CSV file.")
            return

        self.log(f"📂 Processing file: {os.path.basename(filepath)}")
        self.log(f"📊 Found {len(classes)} unique classes: {', '.join(classes)}")

        # Send objects to Arduino EEPROM
        self.log(f"📤 Sending {len(classes)} objects to Arduino EEPROM...")
        if not send_objects_to_eeprom(classes, self.arduino):
            self.log("❌ Failed to send objects to Arduino")
            messagebox.showerror("Error", "Failed to send object list to Arduino.")
            return

        self.log(f"✅ Successfully sent objects to Arduino")
        
        # Wait for Arduino to confirm it's ready
        self.log("⏳ Waiting for Arduino to process objects...")
        
        # Setup binary sorting
        self.sort_classes = classes
        self.current_sort_index = 0
        self.current_target_class = self.sort_classes[0] if self.sort_classes else None
        self.sorting_in_progress = False
        self.waiting_for_next_pass = False
        self.arduino_ready_to_sort = True

        self.log(f"🎯 Binary sort setup complete!")
        self.log(f"🔄 Total passes needed: {len(self.sort_classes)}")
        
        # Enable sorting controls
        self.update_sort_status()
        self.next_pass_btn.configure(state=ctk.NORMAL)
        self.finish_sort_btn.configure(state=ctk.NORMAL)
        
        # Start sorting mode automatically
        self.start_sorting_mode()
        
        self.log("🚀 Sorting session ready! Click 'Next Pass' to begin first sorting pass.")

    # ---------------------------
    # 🎯 Binary Sorting Functions (simplified)
    # ---------------------------
    def update_sort_status(self):
        if not self.arduino_ready_to_sort:
            self.sort_status_label.configure(text="❌ Arduino not ready", text_color="red")
            return

        if not self.sort_classes:
            self.sort_status_label.configure(text="✅ Ready to start sorting session", text_color="green")
            return

        if self.current_sort_index >= len(self.sort_classes):
            self.sort_status_label.configure(text="🎉 All sorting passes complete!", text_color="green")
            return

        # Status for in-progress session
        status = f"Pass {self.current_sort_index + 1}/{len(self.sort_classes)} - Target: '{self.current_target_class}'"
        if self.sorting_in_progress:
            status += " (ACTIVE)"
        elif self.waiting_for_next_pass:
            status += " (WAITING)"
        else:
            status += " (READY)"

        self.sort_status_label.configure(text=status, text_color="white")

    def start_next_pass(self):
        if not self.sort_classes or self.current_sort_index >= len(self.sort_classes):
            self.log("✅ All sorting passes completed!")
            self.finish_sorting()
            return

        self.current_target_class = self.sort_classes[self.current_sort_index]
        self.sorting_in_progress = True
        self.waiting_for_next_pass = False

        self.log(f"▶️ Starting Pass {self.current_sort_index + 1}: Sorting '{self.current_target_class}'")
        self.log(f"📝 Instructions: '{self.current_target_class}' → Target pile, All others → Other pile")
        
        # Send the target class to Arduino
        self.send_to_arduino(f"SET_TARGET:{self.current_target_class}")
        
        self.update_sort_status()
        self.next_pass_btn.configure(text="⏸️ Pause Pass", command=self.pause_current_pass)

    def pause_current_pass(self):
        if self.sorting_in_progress:
            self.sorting_in_progress = False
            self.waiting_for_next_pass = True
            self.current_sort_index += 1
            
            self.log(f"⏸️ Pass paused. Ready for next pass.")
            
            if self.current_sort_index < len(self.sort_classes):
                self.log(f"📋 Next pass will sort: '{self.sort_classes[self.current_sort_index]}'")
                self.next_pass_btn.configure(text="➡️ Next Pass", command=self.start_next_pass)
            else:
                self.log("🎉 All passes completed!")
                self.next_pass_btn.configure(state=ctk.DISABLED)
            
            self.update_sort_status()
            self.send_to_arduino("PAUSE_SORT")

    def finish_sorting(self):
        self.sorting_in_progress = False
        self.waiting_for_next_pass = False
        self.current_sort_index = 0
        self.sort_classes = []
        self.current_target_class = None
        
        self.log("✅ Sorting session finished")
        self.update_sort_status()
        
        self.next_pass_btn.configure(text="➡️ Next Pass", command=self.start_next_pass, state=ctk.DISABLED)
        self.finish_sort_btn.configure(state=ctk.DISABLED)
        
        self.send_to_arduino("FINISH_SORT")

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
        self.scan_btn.configure(text="⏹ Stop Scanning Mode", state=ctk.NORMAL)
        self.sort_btn.configure(state=ctk.DISABLED)
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
            self.log("❌ Arduino is not ready. Start sorting session first.")
            return
        if not self.sort_classes:
            self.log("❌ No sorting session active. Use 'Start Sorting Session' first!")
            return
        
        self.stop_detection()
        self.mode = "sort"
        self.sort_btn.configure(text="⏹ Stop Sorting Mode", state=ctk.NORMAL)
        self.scan_btn.configure(state=ctk.DISABLED)
        self.log("⚙️ Sorting mode active")
        self.is_running = True
        self.update_frame()

    def stop_detection(self):
        self.is_running = False
        was_sorting = (self.mode == "sort")
        self.mode = None
        self.video_frame.configure(image=self.placeholder_img)
        self.video_frame.imgtk = self.placeholder_img
        self.scan_btn.configure(text="📸 Start Scanning Mode", state=ctk.NORMAL)
        self.sort_btn.configure(text="⚙️ Start Sorting Mode", state=ctk.NORMAL)
        
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
    # 🔁 Frame Processing
    # ---------------------------
    def update_frame(self):
        if not self.is_running:
            return

        ret, frame = self.cap.read()
        if not ret:
            self.log("❌ Failed to grab frame - stopping detection")
            self.stop_detection()
            return

        results = self.model(frame)[0]
        annotated = results.plot()

        if self.mode == "scan":
            self.handle_scanning_mode(results)
        elif self.mode == "sort":
            self.handle_sorting_mode(results)

        rgb_frame = cv2.cvtColor(annotated, cv2.COLOR_BGR2RGB)
        img = Image.fromarray(rgb_frame)
        img_size = img.size  # This returns (width, height)
        ctk_img = CTkImage(light_image=img, dark_image=img, size=img_size)  # dynamic size
        self.video_frame.imgtk = ctk_img
        self.video_frame.configure(image=ctk_img, text="")

        self.root.after(FRAME_RATE_MS, self.update_frame)

    # ---------------------------
    # 📸 Scanning Mode Logic
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
    # ⚙️ Enhanced Sorting Mode Logic
    # ---------------------------
    def handle_sorting_mode(self, results):
        if not self.sorting_in_progress or not self.current_target_class:
            return

        self.send_to_arduino("sort")
        
        # Process all detected objects
        for box in results.boxes:
            cls_id = int(box.cls[0])
            class_name = self.model.names[cls_id]
            
            if class_name == self.current_target_class:
                self.log(f"🎯 TARGET FOUND: {class_name} → Target pile")
                self.send_to_arduino(f"SORT_TARGET:{class_name}")
            else:
                self.log(f"📦 OTHER: {class_name} → Other pile")
                self.send_to_arduino(f"SORT_OTHER:{class_name}")
            
            # Only process the first detection to avoid spam
            break

    # ---------------------------
    # 🔌 Arduino Communication
    # ---------------------------

    def connect_to_arduino(self):
        port = find_arduino_port()
        self.update_sort_status()
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
                    self.arduino_btn.configure(text="Arduino Connected!", state=ctk.DISABLED)
                    
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
        self.update_sort_status()
    
    def exit_app(self):
        if self.arduino and self.arduino.is_open:
            port = self.arduino.port
            self.arduino.close()
            self.clear_stored_objects()
            reset_arduino_before_exit(port)
        self.root.destroy()

# ---------------------------
# 🚀 Run GUI
# ---------------------------
if __name__ == "__main__":
    root = ctk.CTk()
    app = YOLOApp(root)
    root.mainloop()

    if app.cap.isOpened():
        app.cap.release()
    cv2.destroyAllWindows()
```

```cpp title="arduino_v2.ino" linenums="1"
#include <Arduino.h>
#include <EEPROM.h>

// ===== MOTOR CONTROL INCLUDES =====
// TODO: Add your servo/stepper motor library includes here
// Example: #include <Servo.h>
// Example: #include <Stepper.h>

// ===== MOTOR CONTROL OBJECTS =====
// TODO: Initialize your motor control objects here
// Example: Servo sortingServo;
// Example: Stepper conveyorMotor(stepsPerRevolution, motorPin1, motorPin2, motorPin3, motorPin4);

// ===== MOTOR CONTROL PINS =====
// TODO: Define your motor control pins here
// Example: const int SERVO_PIN = 9;
// Example: const int CONVEYOR_ENABLE_PIN = 8;
// Example: const int CONVEYOR_DIR_PIN = 7;
// Example: const int CONVEYOR_STEP_PIN = 6;

// Storage constants
const uint8_t MAX_ITEMS = 30;
const uint8_t MAX_STR_LEN = 50;
const int EEPROM_START_ADDR = 0;
const int NUM_ITEMS_ADDR = 0;
const int ITEMS_START_ADDR = 1;

// Object storage
String storedItems[MAX_ITEMS];
uint8_t numStoredItems = 0;

// Mode tracking
String currentMode = "";
String lastModeMessage = "";
unsigned long lastModeTime = 0;
const unsigned long MODE_MESSAGE_COOLDOWN = 5000; // 5 seconds between mode messages
bool itemsLoaded = false;

// Binary Sorting Variables
String currentTargetClass = "";
bool binarySortActive = false;
String lastSortedItem = "";
unsigned long lastSortTime = 0;
const unsigned long SORT_COOLDOWN = 2000; // 2 seconds between same item sorts

// ===== MOTOR CONTROL POSITIONS =====
// TODO: Define your sorting positions/angles here
// Example: const int TARGET_POSITION = 90;    // Servo angle for target pile
// Example: const int OTHER_POSITION = -90;    // Servo angle for other pile
// Example: const int CENTER_POSITION = 0;     // Servo center/neutral position
// Example: const int CONVEYOR_SPEED = 100;    // Conveyor belt speed

// ===== FUNCTION DECLARATIONS =====
void performTargetSortAction();
void performOtherSortAction();
void handleBinarySortTarget(String itemClass);
void handleBinarySortOther(String itemClass);
void processObjectList(String objectListString);
void storeObjectsInEEPROM(String objects[], uint8_t count);
void loadObjectsFromEEPROM();

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
      Serial.print(" ");
      Serial.println(storedItems[i]);
    }
    Serial.println("READY_TO_SORT");
  }
}

void handleBinarySortTarget(String itemClass) {
  unsigned long currentTime = millis();
  
  // Only process if it's a different item or enough time has passed
  if (itemClass != lastSortedItem || (currentTime - lastSortTime) > SORT_COOLDOWN) {
    Serial.print("🎯 TARGET SORT: ");
    Serial.println(itemClass);
    
    // Perform target sorting action
    performTargetSortAction();
    
    lastSortedItem = itemClass;
    lastSortTime = currentTime;
  }
}

void handleBinarySortOther(String itemClass) {
  unsigned long currentTime = millis();
  
  // Only process if it's a different item or enough time has passed  
  if (itemClass != lastSortedItem || (currentTime - lastSortTime) > SORT_COOLDOWN) {
    Serial.print("📦 OTHER SORT: ");
    Serial.println(itemClass);
    
    // Perform other sorting action
    performOtherSortAction();
    
    lastSortedItem = itemClass;
    lastSortTime = currentTime;
  }
}

void performTargetSortAction() {
  // ===== TARGET PILE MOTOR CONTROL =====
  // TODO: Add your motor control code here for moving items to target pile
  
  Serial.println("→ Moving to TARGET pile");
  
  // Example servo control:
  // sortingServo.write(TARGET_POSITION);  // Move to target pile position
  // delay(1000);                          // Wait for movement
  // sortingServo.write(CENTER_POSITION);  // Return to center
  
  // Example stepper motor control:
  // digitalWrite(CONVEYOR_DIR_PIN, HIGH); // Set direction to target pile
  // for(int i = 0; i < STEPS_TO_TARGET; i++) {
  //   digitalWrite(CONVEYOR_STEP_PIN, HIGH);
  //   delayMicroseconds(STEP_DELAY);
  //   digitalWrite(CONVEYOR_STEP_PIN, LOW);
  //   delayMicroseconds(STEP_DELAY);
  // }
  
  // Example pneumatic actuator control:
  // digitalWrite(PNEUMATIC_TARGET_PIN, HIGH);  // Activate target actuator
  // delay(500);                                // Hold position
  // digitalWrite(PNEUMATIC_TARGET_PIN, LOW);   // Release
  
  // TODO: Add any additional mechanical actions needed for target sorting
  // Examples: conveyor belt control, pusher mechanisms, etc.
}

void performOtherSortAction() {
  // ===== OTHER PILE MOTOR CONTROL =====
  // TODO: Add your motor control code here for moving items to other pile
  
  Serial.println("→ Moving to OTHER pile");
  
  // Example servo control:
  // sortingServo.write(OTHER_POSITION);   // Move to other pile position  
  // delay(1000);                          // Wait for movement
  // sortingServo.write(CENTER_POSITION);  // Return to center
  
  // Example stepper motor control:
  // digitalWrite(CONVEYOR_DIR_PIN, LOW);  // Set direction to other pile
  // for(int i = 0; i < STEPS_TO_OTHER; i++) {
  //   digitalWrite(CONVEYOR_STEP_PIN, HIGH);
  //   delayMicroseconds(STEP_DELAY);
  //   digitalWrite(CONVEYOR_STEP_PIN, LOW);
  //   delayMicroseconds(STEP_DELAY);
  // }
  
  // Example pneumatic actuator control:
  // digitalWrite(PNEUMATIC_OTHER_PIN, HIGH);   // Activate other actuator
  // delay(500);                                // Hold position
  // digitalWrite(PNEUMATIC_OTHER_PIN, LOW);    // Release
  
  // TODO: Add any additional mechanical actions needed for other sorting
  // Examples: conveyor belt control, pusher mechanisms, etc.
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
  
  Serial.println("🤖 Arduino Binary Object Sorter Starting...");
  
  // ===== MOTOR CONTROL INITIALIZATION =====
  // TODO: Initialize your motors and actuators here
  
  // Example servo initialization:
  // sortingServo.attach(SERVO_PIN);
  // sortingServo.write(CENTER_POSITION);  // Start at center position
  
  // Example stepper motor initialization:
  // pinMode(CONVEYOR_ENABLE_PIN, OUTPUT);
  // pinMode(CONVEYOR_DIR_PIN, OUTPUT);
  // pinMode(CONVEYOR_STEP_PIN, OUTPUT);
  // digitalWrite(CONVEYOR_ENABLE_PIN, HIGH);  // Enable stepper driver
  
  // Example pneumatic initialization:
  // pinMode(PNEUMATIC_TARGET_PIN, OUTPUT);
  // pinMode(PNEUMATIC_OTHER_PIN, OUTPUT);
  // digitalWrite(PNEUMATIC_TARGET_PIN, LOW);  // Ensure actuators start retracted
  // digitalWrite(PNEUMATIC_OTHER_PIN, LOW);
  
  // TODO: Add any sensor initialization here
  // Example: proximity sensors, limit switches, etc.
  
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
      
    } else if (input.startsWith("SET_TARGET:")) {
      // Set the target class for binary sorting
      currentTargetClass = input.substring(11); // Remove "SET_TARGET:" prefix
      binarySortActive = true;
      Serial.print("🎯 Target class set to: ");
      Serial.println(currentTargetClass);
      Serial.println("🔄 Binary sort mode activated");
      
      // ===== MOTOR CONTROL: SORTING SESSION START =====
      // TODO: Add any initialization needed when starting a new sorting session
      // Examples: move servos to ready position, start conveyor belt, etc.
      
    } else if (input.startsWith("SORT_TARGET:")) {
      // Handle target object detection
      if (binarySortActive) {
        String detectedClass = input.substring(12); // Remove "SORT_TARGET:" prefix
        handleBinarySortTarget(detectedClass);
      }
      
    } else if (input.startsWith("SORT_OTHER:")) {
      // Handle non-target object detection  
      if (binarySortActive) {
        String detectedClass = input.substring(11); // Remove "SORT_OTHER:" prefix
        handleBinarySortOther(detectedClass);
      }
      
    } else if (input == "PAUSE_SORT") {
      Serial.println("⏸️ Sorting pass paused");
      
      // ===== MOTOR CONTROL: PAUSE ACTIONS =====
      // TODO: Add pause actions here
      // Examples: stop conveyor belt, move servos to safe position, etc.
      
      currentTargetClass = "";
      lastSortedItem = ""; // Reset to allow immediate sorting when resumed
      
    } else if (input == "FINISH_SORT") {
      binarySortActive = false;
      currentTargetClass = "";
      lastSortedItem = "";
      Serial.println("✅ Binary sorting session finished");
      
      // ===== MOTOR CONTROL: SESSION END =====
      // TODO: Add cleanup actions here
      // Examples: return all servos to home position, stop conveyor, etc.
      
    } else if (input == "LOAD_OBJECTS") {
      loadObjectsFromEEPROM();
      
    } else if (input == "LIST_OBJECTS") {
      if (itemsLoaded && numStoredItems > 0) {
        Serial.println("📋 Current object list:");
        for (uint8_t i = 0; i < numStoredItems; i++) {
          Serial.print(" ");
          Serial.print(i + 1);
          Serial.print(": ");
          Serial.println(storedItems[i]);
        }
        if (binarySortActive) {
          Serial.print("🎯 Current target: ");
          Serial.println(currentTargetClass);
        }
      } else {
        Serial.println("📭 No objects stored");
      }
      
    } else if (input == "CLEAR_OBJECTS") {
      EEPROM.write(NUM_ITEMS_ADDR, 0);
      numStoredItems = 0;
      itemsLoaded = false;
      binarySortActive = false;
      currentTargetClass = "";
      Serial.println("🗑️ Object list cleared");
      
    } else if (input == "scan") {
      currentMode = "scan";
      unsigned long currentTime = millis();
      if (lastModeMessage != "scan" || (currentTime - lastModeTime) > MODE_MESSAGE_COOLDOWN) {
        Serial.println("📸 Scan mode active");
        lastModeMessage = "scan";
        lastModeTime = currentTime;
      }
      
      // ===== MOTOR CONTROL: SCAN MODE =====
      // TODO: Add scan mode motor actions here
      // Examples: position camera, start conveyor for scanning, etc.
      
    } else if (input == "sort") {
      currentMode = "sort";
      unsigned long currentTime = millis();
      if (itemsLoaded) {
        if (lastModeMessage != "sort" || (currentTime - lastModeTime) > MODE_MESSAGE_COOLDOWN) {
          if (binarySortActive) {
            Serial.println("⚙️ Binary sort mode active");
          } else {
            Serial.println("❌ Binary sort not activated - use SET_TARGET first");
          }
          lastModeMessage = "sort";
          lastModeTime = currentTime;
        }
      } else {
        Serial.println("❌ Cannot sort - no objects loaded");
      }
      
      // ===== MOTOR CONTROL: SORT MODE =====
      // TODO: Add sort mode motor actions here  
      // Examples: start conveyor belt, position sorting mechanisms, etc.
      
    } else if (input == "stop") {
      currentMode = "";
      Serial.println("⏹️ Stopped");
      lastModeMessage = ""; // Reset so next mode change shows message
      lastSortedItem = ""; // Reset sort cooldown
      
      // ===== MOTOR CONTROL: STOP ALL =====
      // TODO: Add emergency stop actions here
      // Examples: stop all motors, return servos to safe positions, etc.
      
    } else {
      Serial.print("❓ Unknown command: ");
      Serial.println(input);
    }
  }
  
  // ===== MOTOR CONTROL: MAIN LOOP TASKS =====
  // TODO: Add any continuous motor control tasks here
  // Examples: sensor monitoring, position feedback, safety checks, etc.
}
```
<div><br></div>

## Results

<div class="video-wrapper">
  <iframe 
    src="https://www.youtube.com/embed/U8tS1O6jC3M?mute=1&rel=0&playsinline=1"
    title="Software Version 2 Demo"
    frameborder="0"
    allow="autoplay; encrypted-media"
    allowfullscreen>
  </iframe>
</div>

Note that there is a part in the video where a message pops up to show that the Arduino is not connected. This is another form of error checking that occurs because the Arduino was not plugged in at that point.

## Conclusion

Overall, it works as expected, and theoretically, any changes that have to be made will now be towards integrating the prototype, Arduino, and user interface. 