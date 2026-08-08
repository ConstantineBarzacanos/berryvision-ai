# agrovision-ai
Agricultural Computer Vision &amp; AI Dataset Acquisition Platform
### Overview

**AgroVision AI** is an end-to-end agricultural computer vision and edge-AI platform designed to transform field imagery into structured, model-ready datasets.

The platform provides a complete acquisition workflow spanning **mobile and edge-camera capture, image and video collection, session management, persistent storage, rich metadata, AI-assisted annotation, and dataset preparation for computer vision model training**. It supports field acquisition through mobile devices such as the iPhone while providing an architecture designed to extend to **NVIDIA Jetson-based edge vision systems**.

AgroVision was developed as a working R&D platform rather than a UI-only demonstration. Captured images and videos are persisted as structured field sessions with associated contextual metadata, including crop and variety information, location, environmental conditions, acquisition source, timestamps, and computer-vision annotations. Datasets can subsequently be inspected, managed, and prepared for downstream training and export workflows such as **YOLO and COCO**.

The architecture is intentionally modular, separating **camera hardware, acquisition services, session management, storage, metadata, AI inference, and dataset management**, allowing the same platform to evolve from human-operated field data collection toward autonomous edge-AI acquisition and agricultural robotics.

**In short:**

**Field → Camera → Acquisition → AI/Metadata → Dataset → Model Training → Edge Intelligence**

### Architecture 
<img width="800" height="1000" alt="ChatGPT Image Aug 8, 2026, 02_39_18 PM" src="https://github.com/user-attachments/assets/ffe3ff50-e544-4686-b2c4-cafba1626751" />

### Application Screenshots

**The Dashboard**

<img width="1244" height="622" alt="Screen Shot 2026-08-08 at 2 56 34 PM" src="https://github.com/user-attachments/assets/ab133ba5-b6c8-470d-94e6-937fc7106264" />

**Field Survey Initiation Screen**

<img width="671" height="655" alt="Screen Shot 2026-08-08 at 2 57 54 PM" src="https://github.com/user-attachments/assets/36f9b5e1-c5e9-48be-ad75-e35f96df5e91" />

**Field Survey Metadata for downstream AI Processing**

<img width="1319" height="743" alt="Screen Shot 2026-08-08 at 3 14 55 PM" src="https://github.com/user-attachments/assets/4d77a0f7-7846-4d13-bb72-54f42f4fd15f" />

**Project Status**
AgroVision AI is an independently developed R&D prototype and demonstration platform for agricultural computer vision and edge-AI workflows.

© 2026 Constantine Barzacanos. All rights reserved.

