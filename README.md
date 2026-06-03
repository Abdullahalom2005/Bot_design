Ah, I see exactly what happened! When you highlight and copy the text directly from the chat screen, some platforms strip out all the invisible line breaks and formatting, turning it into a giant wall of text.To fix this so you can copy it perfectly, I have placed the entire README inside a code block below.All you need to do is click the small "Copy" button that appears in the top-right corner of the block below, and then paste it into your GitHub repository. It will keep all the tables, fonts, and spacing perfectly intact!Markdown# 🤖 Line Following Bot with Vision Integration

> **Autonomous line-following robot with real-time object detection, obstacle avoidance, and a live web dashboard.**

## 📋 Project Overview

| Feature | Details |
| :--- | :--- |
| **Line Following** | 6-sensor IR array, smooth + sharp + memory-assisted correction |
| **Obstacle Avoidance** | 3x ultrasonic — dynamic 180° front turn, servo sweep on sides |
| **Object Detection** | YOLOv8, 18 classes, Raspberry Pi camera |
| **Live Dashboard** | MJPEG stream + detection panel, Flask port 5000 |
| **Run Recording** | `.avi` video + timestamped `.txt` detection log |

---

## 📁 Repository Structure

```text
keus_ROBOTICS/
├── Esp_code/
│   └── main.ino          # ESP32: line following + obstacle avoidance
├── ML_train_code/
│   ├── dataset/          # Images and labels (YOLO format)
│   ├── dataset.yaml      # Ultralytics config
│   ├── train_boxes.py    # Train YOLOv8
│   └── test_boxes.py     # Test on webcam
└── Rasberry_pi_code/
    └── main.py           # Flask: stream, inference, recording
🛠️ HardwareComponentDetailsMicrocontrollerESP32-WROOM-32SBCRaspberry Pi 4CameraRaspberry Pi Camera v3 or HQMotor DriverL298N dual H-bridgeDC Motors4x differential driveIR Sensors6-channel digital arrayUltrasonic3x HC-SR04 (front, left, right)ServoStandard hobby servo, pin 15PowerLiPo / battery pack💻 ESP32 Firmware (Esp_code/main.ino)Pin AssignmentsFunctionPinsLeft motor (IN1, IN2, ENA)22, 23, 18Right motor (IN3, IN4, ENB)19, 21, 5IR sensors (S0–S5)34, 35, 32, 33, 25, 26Front ultrasonic (trig / echo)27 / 14Left ultrasonic (trig / echo)12 / 13Right ultrasonic (trig / echo)2 / 4Servo15Line Following — IR Sensor States(1 = sees line, 0 = no line)ForwardS0S1S2S3S4S5Action001100Forward001000Forward000100Forward011110ForwardTurn LeftS0S1S2S3S4S5Action011000Gentle left010000Gentle left011100Gentle left110000Moderate left111000Moderate left101000Moderate left100000Sharp left111100Sharp left111110Sharp leftTurn RightS0S1S2S3S4S5Action000110Gentle right000010Gentle right001110Gentle right000011Moderate right000111Moderate right000101Moderate right000001Sharp right001111Sharp right011111Sharp rightSpecial StatesS0S1S2S3S4S5Action111111Crossroad — brake000000Line lost — recoveryLost-Line RecoveryPhaseTriggerActionMemoryAll sensors zeroTurn in last direction for 100 ms (straight) or 700 ms (turn)Active scanAfter memory timeoutSpin until any sensor sees blackFailsafeAfter 10 sHard brakeObstacle AvoidanceSensorThresholdActionFront< 15 cmBrake → spin right → re-acquire line → freeze all sensorsLeft< 20 cmBrake → servo 180° → wait 3 s → recenter → freeze 2 sRight< 20 cmBrake → servo 0° → wait 3 s → recenter → freeze 2 sNote: Front freeze duration = 2 x (runTime - 3s)🧠 ML Training (ML_train_code/)Dataset StructurePlaintextdataset/
├── train/
│   ├── images/
│   └── labels/    (class cx cy w h)
└── valid/
    ├── images/
    └── labels/
Train the ModelBashcd ML_train_code
python train_boxes.py
Output: runs/detect/trainN/weights/best.pt(Use epochs=50+ for real accuracy — default epochs=2 is for testing only).Test on WebcamBashpython test_boxes.py
Press q to quit.Raise conf above 0.0032 to reduce false positives.🌐 Raspberry Pi Server (Rasberry_pi_code/main.py)ComponentDetailCameraPicamera2 640x480, RGB to BGRInferenceEvery 3rd frame, imgsz=320, conf=0.5RecordingMJPG .avi, 10 FPS, 10-min cap, pause/resumeLoggingrun_timestamp_log.txt, one line per detectionAPI EndpointsRouteDescriptionGET /Web dashboardGET /video_feedMJPEG streamGET /detectionsJSON detected objectsGET /startStart / resume recordingGET /stopPause recordingModel Weights⚠️ Not included — configure before running.Download yolov8n.pt from Ultralytics → place in ML_train_code/Train → copy best.pt to Pi → update path in main.pyPythonmodel = YOLO("runs/detect/train/weights/best.pt")
🚀 Installation & Running1. ESP32Open Arduino IDE and ensure ESP32 board support is installed.Install the ESP32Servo library via Library Manager.Open main.ino, select your board and port, and click Upload.Usage: Power on → 3 s delay → starts automatically. Serial at 9600 baud.2. ML Training (PC/GPU)Bashpip install ultralytics opencv-python
3. Raspberry PiBashpip install flask picamera2 ultralytics opencv-python
cd Rasberry_pi_code
python main.py
Usage: Open http://<pi-ip>:5000 in your browser.🏷️ Dataset & Classes#ClassCategory0Brandlogo_MaybechlogoBrand Logo1Brandlogo_TeslalogoBrand Logo2Brandlogo_applelogoBrand Logo3Brandlogo_keuslogoBrand Logo4QRcodeCode5chairFurniture6faces_RogerFedererFace7faces_henryCavillFace8faces_keanureevesFace9numberplateVehicle10parcelboxObject11pets_catAnimal12pets_dogAnimal13smartswitchElectronics14tableFurniture15vehicle_bicycleVehicle16vehicle_carVehicle17vehicle_motorbikeVehicle⚙️ TuningESP32 Speed Variables (main.ino)C++const int baseSpeed         = 85;
const int turnSpeed         = 135;
const int innerWheelSpeed   = 75;
const int sharpTurnSpeed    = 160;
const int sharpReverseSpeed = 100;
ESP32 Recovery TimeoutsC++const unsigned long sweepTimeoutStraight = 100; // ms
const unsigned long sweepTimeoutTurn     = 700; // ms
Pi Inference SettingsSettingDefaultEffectimgsz320Lower = faster, 640 = more accurateconf0.5Raise to cut false positivesframe skip% 3Raise for smoother stream🔧 TroubleshootingProblemFixRobot driftsTune baseSpeed / innerWheelSpeedUltrasonic reads 999Check wiring, ground loops on echo pin180 turn overshootsTune sharpTurnSpeed / sharpReverseSpeedYOLO slow on PiSet imgsz=160, skip every 5th frame, use INT8 modelpicamera2 failsEnable in raspi-config, test with libcamera-hellobest.pt not foundUse absolute path: YOLO("/home/pi/weights/best.pt")Stream blankRun Flask on 0.0.0.0, allow port 5000 in firewallSide sensor false triggerCheck freezeDuration constant
