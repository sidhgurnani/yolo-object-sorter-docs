# Troubleshooting and FAQ

## Troubleshooting Guide

### Hardware Issues

#### Arduino Not Detected
**Symptoms:** "No Arduino detected" error during setup

**Solutions:**

1. Verify Arduino Leonardo is connected via USB
2. Install Arduino drivers if on Windows
3. Check Device Manager for COM port
4. Try different USB cable/port
5. Restart application after connecting Arduino

#### Camera Not Working
**Symptoms:** Black screen or "Camera test FAILED"

**Solutions:**

1. Close other applications using camera (Zoom, Teams, etc.)
2. Try different camera index in wizard
3. Check camera permissions in Windows/macOS settings
4. Verify camera works in another application first
5. Use USB 2.0 port instead of USB 3.0 (sometimes more stable)
6. Unplug and replug the webcam

#### Servo Not Moving
**Symptoms:** Objects detected but not physically sorted

**Solutions:**

1. Check servo is connected to correct pin (default: pin 3)
2. Verify 5V power supply to servo
3. Test servo with simple Arduino sketch first
4. Check TARGET_ANGLE and NOT_TARGET_ANGLE values
5. Ensure servo isn't mechanically blocked

### Software Issues

#### Model Not Loading
**Symptoms:** Error when selecting .pt file

**Solutions:**

1. Verify file is a valid YOLO .pt model
2. Check Python has ultralytics package installed
3. Ensure model is trained for object detection (not classification)
4. Try re-training model with latest Ultralytics YOLO

#### Objects Not Detected
**Symptoms:** Camera shows feed but no detections

**Solutions:**

1. Lower YOLO_CONF_THRESHOLD (try 0.50)
2. Verify model is trained on your object classes
3. Improve lighting conditions
4. Ensure objects are in frame and clearly visible
5. Check object names match exactly (case-sensitive)

#### Communication Timeouts
**Symptoms:** "FATAL: No pass data received" error

**Solutions:**

1. Increase timeout value (currently 3s)
2. Check Arduino Serial Monitor for errors
3. Verify Arduino code uploaded correctly
4. Reset Arduino and reconnect
5. Check for loose USB connection

### Performance Issues

#### Slow Detection
**Symptoms:** Long delays between object detections

**Solutions:**

1. Reduce FRAME_RATE_MS to 100ms
2. Reduce detection_confirmation_frames to 2
3. Lower YOLO_IMG_SIZE to 320 (faster inference)
4. Use CPU with better single-core performance
5. Close other applications

#### High CPU Usage
**Symptoms:** Computer fans loud, system laggy

**Solutions:**

1. Increase FRAME_RATE_MS to 200ms
2. Reduce YOLO_IMG_SIZE to 320
3. Close unnecessary applications
4. Use dedicated GPU if available (CUDA)

#### False Positives
**Symptoms:** Shadows or wrong objects being detected

**Solutions:**

1. Increase YOLO_CONF_THRESHOLD to 0.70
2. Increase detection_confirmation_frames to 4
3. Improve consistent lighting
4. Retrain model with more diverse examples
5. Add negative examples to training data

## FAQ

### General Questions

**Q: How many unique object types can the system sort?**
A: Theoretically unlimited, but practical limit is ~10-15 unique object types due to EEPROM storage and sorting time. Each additional type requires one more pass.

**Q: How accurate is the sorting?**
A: Accuracy depends on model quality, lighting, and object distinctiveness. A properly trained model and a controlled testing and deployment environment can yield an accuracy above 95%.

**Q: How fast can it sort?**
A: Speed heavily depends on servo movement time and detection settings, and partially dependent on the quality of the model being used.

### Technical Questions

**Q: Why does Arduino send PASS_COMPLETE 3 times?**
A: Redundancy to ensure Python receives the message despite potential serial buffer issues.

**Q: What is the detection cooldown for?**
A: Prevents the same object from being counted multiple times as it passes through the camera view.

**Q: Can I use a different Arduino board?**
A: Yes, but must support Serial (not just USB CDC). Leonardo, Mega, and Uno work. Nano has limited EEPROM.

**Q: Why is the first pass total used as ground truth?**
A: The first pass counts ALL objects (target + non-target), providing the true total count before any are removed.