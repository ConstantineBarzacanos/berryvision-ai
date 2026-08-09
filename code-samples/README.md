# Overall Software Architecture Structure 

1. Camera Hardware Abstraction
   - architectural purpose
   - AbstractCamera code excerpt

2. Camera Factory / Dependency Injection
   - architectural purpose
   - factory code excerpt

3. Mobile / WebRTC Acquisition
   - architectural purpose
   - representative implementation excerpt

4. NVIDIA Jetson Edge Camera
   - architectural purpose
   - representative implementation excerpt

5. Acquisition Session Lifecycle
   - ACTIVE → PAUSED → RESUMED → COMPLETED
   - representative code

6. Metadata & AI Data Contract
   - GPS
   - image/video metadata
   - detections / bounding boxes
   - maturity / density
   - representative schema

7. Dataset / MLOps Boundary
   - acquisition → dataset → YOLO / COCO → training
   - representative export code

8. Architecture Notes
   - what is intentionally omitted
   - complete operational repository remains private

# Representative Implementation Samples

These selected code samples illustrate architectural patterns implemented in the **BerryVision AI** personal R&D platform.

The purpose of this directory is to demonstrate architectural depth rather than publish the complete application. The samples focus on the engineering seams that matter most to an AI / MLOps / enterprise architect:

- Hardware abstraction between mobile and edge cameras
- Dependency injection through a camera factory
- Consistent camera lifecycle and telemetry contracts
- Portability from browser/mobile acquisition to NVIDIA Jetson edge hardware
- Separation of field-acquisition concerns from higher-level application logic

The complete development repository, operational configuration, credentials, field datasets, local filesystem structure, and deployment-specific implementation are intentionally not publicly distributed.

## Camera Architecture

```text
Application / Acquisition Services
              |
              v
       CameraFactory
              |
              v
       AbstractCamera
          /       \
         /         \
        v           v
 MobileCamera   JetsonCamera
 Browser/WebRTC   CSI/GStreamer
```

The same acquisition workflow can operate against different camera implementations without coupling session or dataset logic to a specific device.

## Files

- `camera/camera_models.py` — device types, status model, and capture configuration.
- `camera/abstract_camera.py` — hardware abstraction contract.
- `camera/mobile_camera.py` — representative mobile/browser adapter.
- `camera/jetson_camera.py` — representative NVIDIA Jetson CSI adapter.
- `camera/camera_factory.py` — dependency-injection factory selecting the appropriate driver.

## Scope

These are representative, sanitized implementation samples derived from the working BerryVision AI R&D platform. They are intentionally scoped to communicate system design patterns, not to provide a complete deployable distribution.

© 2026 Constantine Barzacanos. All rights reserved.
```

---

## `code-samples/camera/camera_models.py`

```python
"""
BerryVision AI — representative camera domain models.

This public sample illustrates the device-independent configuration contract used
by the acquisition layer. Deployment-specific configuration is intentionally
omitted from the public showcase.
"""

from dataclasses import dataclass
from enum import Enum


class CameraType(str, Enum):
    """Supported acquisition-device categories."""

    MOBILE_BROWSER = "MOBILE_BROWSER"
    JETSON_CSI = "JETSON_CSI"
    USB_WEBCAM = "USB_WEBCAM"
    DRONE_GIMBAL = "DRONE_GIMBAL"


class CameraStatus(str, Enum):
    """Operational state exposed consistently by all camera drivers."""

    OFFLINE = "OFFLINE"
    INITIALIZING = "INITIALIZING"
    READY = "READY"
    STREAMING = "STREAMING"
    RECORDING = "RECORDING"
    ERROR = "ERROR"


@dataclass
class CameraConfig:
    """Device-neutral acquisition parameters."""

    camera_type: CameraType = CameraType.MOBILE_BROWSER
    resolution_width: int = 1920
    resolution_height: int = 1080
    framerate: int = 30
    exposure_compensation: float = 0.0
    autofocus_enabled: bool = True
    hdr_enabled: bool = True
```

---

## `code-samples/camera/abstract_camera.py`

```python
"""
BerryVision AI — camera hardware abstraction contract.

The abstraction deliberately separates session and dataset logic from physical
camera implementations. A mobile browser, NVIDIA Jetson CSI camera, USB webcam,
or future robotic sensor can satisfy the same acquisition contract.
"""

from abc import ABC, abstractmethod
from typing import Any, Dict, Optional, Tuple

from .camera_models import CameraConfig, CameraStatus


class AbstractCamera(ABC):
    """Device-independent camera interface used by acquisition services."""

    def __init__(self, config: Optional[CameraConfig] = None) -> None:
        self.config = config or CameraConfig()
        self._status = CameraStatus.OFFLINE
        self._is_previewing = False
        self._is_recording = False

    @property
    def status(self) -> CameraStatus:
        """Current operational status of the camera."""
        return self._status

    @abstractmethod
    def initialize(self) -> bool:
        """Initialize sensors, browser streams, or hardware pipelines."""
        raise NotImplementedError

    @abstractmethod
    def shutdown(self) -> bool:
        """Release camera resources safely."""
        raise NotImplementedError

    @abstractmethod
    def start_preview(self) -> bool:
        """Start the live viewfinder stream."""
        raise NotImplementedError

    @abstractmethod
    def stop_preview(self) -> bool:
        """Stop the live viewfinder stream."""
        raise NotImplementedError

    @abstractmethod
    def start_recording(self) -> bool:
        """Begin video acquisition."""
        raise NotImplementedError

    @abstractmethod
    def stop_recording(self) -> Tuple[bool, str]:
        """Stop video acquisition and return status plus media reference."""
        raise NotImplementedError

    @abstractmethod
    def capture_image(self) -> Tuple[bytes, Dict[str, Any]]:
        """Capture a still frame and device-level metadata."""
        raise NotImplementedError

    @abstractmethod
    def get_status(self) -> Dict[str, Any]:
        """Return normalized device diagnostics and telemetry."""
        raise NotImplementedError
```

---

## `code-samples/camera/mobile_camera.py`

```python
"""
BerryVision AI — representative mobile/browser camera adapter.

The production R&D platform uses the browser media layer for mobile acquisition.
This public sample intentionally omits networking, upload endpoints, and
deployment-specific browser integration.
"""

from typing import Any, Dict, Optional, Tuple

from .abstract_camera import AbstractCamera
from .camera_models import CameraConfig, CameraStatus, CameraType


class MobileCamera(AbstractCamera):
    """Representative adapter for browser/WebRTC-based field acquisition."""

    def __init__(self, config: Optional[CameraConfig] = None) -> None:
        super().__init__(
            config or CameraConfig(camera_type=CameraType.MOBILE_BROWSER)
        )
        self._last_frame: Optional[bytes] = None

    def initialize(self) -> bool:
        self._status = CameraStatus.INITIALIZING
        # Browser/WebRTC initialization occurs at the client integration layer.
        self._status = CameraStatus.READY
        return True

    def shutdown(self) -> bool:
        self._is_previewing = False
        self._is_recording = False
        self._status = CameraStatus.OFFLINE
        return True

    def start_preview(self) -> bool:
        if self._status == CameraStatus.OFFLINE:
            self.initialize()
        self._is_previewing = True
        self._status = CameraStatus.STREAMING
        return True

    def stop_preview(self) -> bool:
        self._is_previewing = False
        self._status = CameraStatus.READY
        return True

    def start_recording(self) -> bool:
        self._is_recording = True
        self._status = CameraStatus.RECORDING
        return True

    def stop_recording(self) -> Tuple[bool, str]:
        self._is_recording = False
        self._status = CameraStatus.READY
        return True, "mobile-video-reference"

    def set_uploaded_frame(self, frame_bytes: bytes) -> None:
        """Accept a frame supplied by the browser/mobile integration layer."""
        self._last_frame = frame_bytes

    def capture_image(self) -> Tuple[bytes, Dict[str, Any]]:
        if self._last_frame is None:
            raise RuntimeError("No mobile frame is currently available.")

        metadata = {
            "device_type": CameraType.MOBILE_BROWSER.value,
            "resolution": (
                f"{self.config.resolution_width}x"
                f"{self.config.resolution_height}"
            ),
            "fps": self.config.framerate,
        }
        return self._last_frame, metadata

    def get_status(self) -> Dict[str, Any]:
        return {
            "device_type": CameraType.MOBILE_BROWSER.value,
            "status": self._status.value,
            "is_previewing": self._is_previewing,
            "is_recording": self._is_recording,
            "resolution": (
                f"{self.config.resolution_width}x"
                f"{self.config.resolution_height}"
            ),
            "fps": self.config.framerate,
        }
```

---

## `code-samples/camera/jetson_camera.py`

```python
"""
BerryVision AI — representative NVIDIA Jetson CSI camera adapter.

The production R&D architecture targets NVIDIA Jetson-class edge hardware.
Pipeline strings, sensor tuning, and deployment-specific configuration are
intentionally simplified in this public sample.
"""

from typing import Any, Dict, Optional, Tuple

from .abstract_camera import AbstractCamera
from .camera_models import CameraConfig, CameraStatus, CameraType


class JetsonCamera(AbstractCamera):
    """Representative CSI/GStreamer adapter for NVIDIA Jetson edge systems."""

    def __init__(self, config: Optional[CameraConfig] = None) -> None:
        super().__init__(config or CameraConfig(camera_type=CameraType.JETSON_CSI))
        self._sensor_id = 0

    def _build_pipeline(self) -> str:
        """Return a simplified hardware-accelerated capture pipeline."""
        width = self.config.resolution_width
        height = self.config.resolution_height
        fps = self.config.framerate

        return (
            f"nvarguscamerasrc sensor-id={self._sensor_id} ! "
            f"video/x-raw(memory:NVMM),width={width},height={height},"
            f"framerate={fps}/1 ! nvvidconv ! appsink"
        )

    def initialize(self) -> bool:
        self._status = CameraStatus.INITIALIZING
        self._pipeline = self._build_pipeline()
        # Production implementation binds the CSI/GStreamer pipeline here.
        self._status = CameraStatus.READY
        return True

    def shutdown(self) -> bool:
        self._is_previewing = False
        self._is_recording = False
        self._status = CameraStatus.OFFLINE
        return True

    def start_preview(self) -> bool:
        if self._status == CameraStatus.OFFLINE:
            self.initialize()
        self._is_previewing = True
        self._status = CameraStatus.STREAMING
        return True

    def stop_preview(self) -> bool:
        self._is_previewing = False
        self._status = CameraStatus.READY
        return True

    def start_recording(self) -> bool:
        self._is_recording = True
        self._status = CameraStatus.RECORDING
        return True

    def stop_recording(self) -> Tuple[bool, str]:
        self._is_recording = False
        self._status = CameraStatus.READY
        return True, "jetson-video-reference"

    def capture_image(self) -> Tuple[bytes, Dict[str, Any]]:
        # Production implementation retrieves a frame from the CSI pipeline.
        raise NotImplementedError(
            "Frame retrieval is intentionally omitted from the public sample."
        )

    def get_status(self) -> Dict[str, Any]:
        return {
            "device_type": CameraType.JETSON_CSI.value,
            "status": self._status.value,
            "is_previewing": self._is_previewing,
            "is_recording": self._is_recording,
            "sensor_id": self._sensor_id,
            "resolution": (
                f"{self.config.resolution_width}x"
                f"{self.config.resolution_height}"
            ),
            "fps": self.config.framerate,
        }
```

---

## `code-samples/camera/camera_factory.py`

```python
"""
BerryVision AI — camera factory / dependency-injection example.

Higher-level acquisition code requests an AbstractCamera and remains unaware of
the physical driver selected at runtime.
"""

from typing import Optional

from .abstract_camera import AbstractCamera
from .camera_models import CameraConfig, CameraType
from .jetson_camera import JetsonCamera
from .mobile_camera import MobileCamera


class CameraFactory:
    """Create the camera implementation appropriate for the deployment target."""

    @staticmethod
    def create_camera(
        camera_type: CameraType = CameraType.MOBILE_BROWSER,
        config: Optional[CameraConfig] = None,
    ) -> AbstractCamera:
        config = config or CameraConfig(camera_type=camera_type)

        if camera_type == CameraType.MOBILE_BROWSER:
            return MobileCamera(config)

        if camera_type == CameraType.JETSON_CSI:
            return JetsonCamera(config)

        raise ValueError(
            f"Camera type '{camera_type.value}' is not enabled "
            "in this representative public sample."
        )
```

---

## `code-samples/camera/__init__.py`

```python
"""BerryVision AI representative camera architecture samples."""
```

## Publication guidance

These files are deliberately **representative and sanitized**. They preserve the architectural patterns present in the working BerryVision R&D codebase while omitting operational endpoints, credentials, local filesystem details, temporary tunnel configuration, deployment-specific settings, captured field data, and the complete private source tree.

For the first public release, publish only this camera/HAL package. It demonstrates:

- Hardware abstraction
- Dependency injection
- Mobile-to-edge portability
- Normalized lifecycle/state handling
- Separation of business/acquisition logic from physical devices

Do **not** publish the private `olive-scanner` repository wholesale.

After the camera package is live and reviewed, the next useful public samples would be:

1. Session lifecycle/state machine
2. Structured metadata/data contracts
3. Dataset export boundary
4. Representative tests

© 2026 Constantine Barzacanos. All rights reserved.
