# 🤖 Transfer Robot 
> **Control System & Multi-Language AI Voice Assistant**

This repository contains the core software for an autonomous **Transfer Robot**, seamlessly combining a **ROS2** hardware control framework with a high-performance **Qt/QML** Graphical User Interface (GUI). 

The system coordinates real-time hardware risk management, local/cloud AI routing, an LLM-powered (Llama) voice assistant, and multi-modal vision processing.

---

## 🛠 Tech Stack

- **GUI Framework:** Qt 5 / Qt 6 (Core, Gui, Qml, Quick)
- **Robotics:** ROS2 (Humble / Jazzy core libraries - `rclcpp`)
- **AI & Audio:** Local Llama (Edge), Cloud AI Router, Google Speech STT, VAD
- **Vision & Hardware:** Luxonis OAK Depth/RGB Camera, LiDAR Sensors
- **Networking & System:** `libcurl`, Multi-threading (`std::thread`, `QThread`)

---

## ✨ Key Features

### 💻 Hybrid GUI & Cross-Platform Support
* **Qt/QML (Fusion Style) Interface:** Provides an intuitive, fluid, and highly responsive UI powered by high-speed C++ backend data binding via `QML Context Property`.
* **Multi-OS Compilation:** Designed with `#ifndef _WIN32` preprocessor directives to separate the production Linux environment (full ROS2/Hardware integration) from the Windows environment (for UI mocking and debugging).

### 🎯 ROS2 Autonomous Navigation & Hardware Interactivity
* **Native ROS2 Integration:** Connects directly to robot control components using `rclcpp::Node` and a dedicated `RosInterface`.
* **Multi-Threaded Executor:** Utilizes a standalone C++ thread (`std::thread`) for the background ROS spin (`exec.spin()`). This ensures real-time sensor data streaming without freezing or stuttering the main UI thread.
* **Sensor Management:** Processes immediate hazard feedback using an integrated `LidarManager` and obstacle detection bounding boxes.

### 🎙️ Intelligent Voice Assistant & TTS Pipeline
* **Hybrid Edge/Cloud AI:** Features a local `LlamaClient` for offline interaction and an `AiRouterClient` for cloud-based smart AI routing.
* **Real-Time Audio Pipeline:** Streams microphone input through `VoiceRecorder` (16kHz with VAD/Threshold detection) $\rightarrow$ `GoogleSpeechClient` (STT) $\rightarrow$ `LLM Inference` $\rightarrow$ `SoundManager` (TTS).
* **Flexible Exceptions:** Supports environment variables (e.g., `TRANSFER_ROBOT_DISABLE_INTERNAL_TTS`) to dynamically mute audio feedback during specific operation modes (e.g., shooting/media mode).

### 👁️ High-Performance Vision Pipeline (Depth & Skeleton)
* **Isolated Thread Execution:** The `CameraManager` worker is offloaded to a dedicated `QThread` (`moveToThread`) to completely prevent GUI lag.
* **OAK Camera & Skeleton Tracking:** Drives a low-power standby pipeline capable of human detection (`personFoundChanged`), distance tracking (`distanceUpdated`), and frame rendering via `VideoItem`.
* **Risk Evaluation Engine:** Feeds vision and LiDAR-based `ObstacleSummary` packages straight into the `TransferRobotAlertManager` to alter robot paths or issue emergency stops.

---

## 🏗 System Architecture & Data Flow

The application relies heavily on an **Event-Driven Architecture**. Thread-safe communication across different layers is handled natively via Qt's Signals & Slots mechanism using `Qt::QueuedConnection`.

```mermaid
graph TD
    LiDAR[LiDAR Sensors] -->|Obstacle Data| AlertManager[TransferRobotAlertManager]
    Camera[OAK Camera] -->|Vision Data| AlertManager
    AlertManager -->|Override / Emergency Stop| ROS2[ROS2 Navigation Node]
    AlertManager -->|Vocal Safety Warning| Sound[TtsManager / SoundManager]
    
    Mic[Microphone Input] -->|16kHz Audio| VoiceRec[VoiceRecorder + VAD]
    VoiceRec --> STT[GoogleSpeechClient]
    STT --> LLM[Llama Client / AI Router]
    LLM --> Sound

AppFlowState (State Machine): Tracks and broadcasts the current page and active execution steps of the robot.

TransferRobotAlertManager (Safety Core): Aggregates obstacle data from both cameras and LiDAR to issue immediate control overrides (controlCommandIssued) and vocal safety warnings.

TtsManager / SoundManager: Converts system alerts, navigation updates, and AI responses into audio playback.
<img width="1350" height="759" alt="image" src="https://github.com/user-attachments/assets/963ef787-28c3-4dec-b433-0fb2307e108f" />

<img width="875" height="656" alt="이승로봇_프로토타입" src="https://github.com/user-attachments/assets/50c592d6-2fa9-42e1-ad7f-f2505f0910ad" />
