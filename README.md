## General information
This is the dataset used in the Paper *SignAlign: A Smartphone-Based Visual Positioning System for Enhanced Bicycle Safety*. It contains data from an instrumented bicycle at three different locations. The dataset can be used to validate bicycle positioning approaches. It contains (among other data) both relative position tracking (from visual inertial odometry), global position information and object detection annotations of road signs.

Each location contains multiple rides and 2 shared files:

**RTK ground truth:** The reference trajectory is recorded with a professional [GNSS RTK device](https://www.alberding.eu/en/A08_RTK_overview.html) and serves as centimeter-level ground truth, especially when `rtkFix = RTK Fixed`. The measurements are stored in `RTK.csv` (5 Hz) and include `Latitude`/`Longitude`/`altitude` plus quality indicators such as `hdop` and the fix status (`rtkFix`) for filtering/weighting.

**Road sign reference points**: At each locations the position and dimensions of all available road signs were precisvly recorded and stored in `signs.csv`. These signs can be used as reference points for positioning. 


# SignAlign – Measurement Data Folder Structure

The following describes the folder structure and the data types/formats found at each of the locations

## Top-level layout

```text
Top-Level-Folder/
  RTK.csv
  signs.csv
  Rides/
    <ride>/
      color.mp4
      notes.txt                  (optional)
      GNSS/0.csv
      accelerometer/0.csv
      angles/0.csv
      gyroscope/0.csv
      heading/0.csv
      intrinsics/0.csv
      position/0.csv
      trackingState/0.csv
      rawDepth/depth/<Timestamp>.bin
      yolo_detections.json
```

`<ride>` folders are named by an UUID (`2F1973B8-24C0-4A8F-AA75-1039A8A54E01`).

## Common conventions

### CSV files
- Header row is always present (first line).
- Decimal separator: `.` (dot).
- Time values containing a comma are quoted with `"` in the CSV (example: `"2025-06-16, 12:47:47.2840"`).

### Time fields (shared across ride CSVs)
| Field | Type | Format | Example | Description |
|---|---|---|---|---|
| `NTP` | String | `YYYY-MM-DD, HH:MM:SS.SSSS` | `"2025-06-16, 12:47:47.2840"` | Network time protocol synchronized timestamp |
| `Timestamp` | String | `YYYY-MM-DD, HH:MM:SS.SSSS` | `"2025-06-16, 12:47:48.2900"` | Local device timestamp |
| `Video` | String | `HH:MM:SS.mmm` | `00:00:00.080` | Relative time since starting the recording |

The NTP timestamp should be used to synchronize the `RTK.csv` file with files of a ride. 


## Root files

### `RTK.csv`
- Type: CSV (comma-separated).
- Purpose: GNSS‑RTK ground truth recorded with a professional RTK device, used as reference in the centimeter range (best quality when `rtkFix = RTK Fixed`).
- Columns:
  - `Date`: String (timestamp)
  - `NTP`: String 
  - `GNSS-Time`: Integer as string 
  - `Latitude`: Float (decimal degrees)
  - `Longitude`: Float (decimal degrees)
  - `altitude`: Float (meters)
  - `hdop`: Float
  - `rtkFix`: String 
- Notes:
  - `rtkFix` is the quality indicator based on the GNGGA standard

### `signs.csv`
- Type: CSV (semicolon-separated `;`).
- Columns:
  - `Label`: String | Unique identifier per sign per locatiopn | (example: `fs15`)
  - `Class_type`: String |Road sign type| (example: `forb_stopping`)
  - `Latitude`: Float (decimal degrees)
  - `Longitude`: Float (decimal degrees)
  - `Height`: Float (meters) | Height of the center of the sign above ground
  - `Circumference`: Float (meters) | Circumference for round signs or height for rectangular signs
  - `Curb`: Float (meters) | The height of the curb at the sign position
  - `Remark`: String 

## Per-ride data (`Rides/<ride>/`)
All csv files of a ride have, as first three columns, `NTP`, `Timestamp`, `Video`

### `Rides/<ride>/color.mp4`
- Type: MP4 container (`.mp4`) with the recorded color video stream. (60Hz)

### `Rides/<ride>/notes.txt` (optional)
- Type: Text file (free-form notes).

### `Rides/<ride>/GNSS/0.csv`
- Type: CSV (comma-separated).
- Description: GNSS data of the smartphone (1 Hz)
- Columns:
  - `Latitude`: Float (decimal degrees)
  - `Longitude`: Float (decimal degrees)
  - `Altitude`: Float (meters)
  - `Speed`: Float (m/s, `-1.0` if “unknown/unavailable”)
  - `Accuracy`: Float (m, radius of horizontal uncertainty for the location)

### `Rides/<ride>/accelerometer/0.csv`
- Type: CSV (comma-separated).
- Description: accelerometer data of the smartphone (100 Hz)
- Columns:
  - `X`: Float (G)
  - `Y`: Float (G)
  - `Z`: Float (G)

### `Rides/<ride>/gyroscope/0.csv`
- Type: CSV (comma-separated).
- Description: gyroscope data of the smartphone (100 Hz)
- Columns:
  - `X`: Float (rad/s)
  - `Y`: Float (rad/s)
  - `Z`: Float (rad/s)

### `Rides/<ride>/angles/0.csv`
- Type: CSV (comma-separated).
- Description: The orientation of the camera (relative to starting position) (60Hz)
- Columns:
  - `X`: Float (rad) | roll
  - `Y`: Float (rad) | pitch 
  - `Z`: Float (rad) | yaw

### `Rides/<ride>/heading/0.csv`
- Type: CSV (comma-separated).
- Description: Heading from the smartphones compass (10Hz)
- Columns:
  - `Heading`: Float (degrees)
  - `Accuracy`: Float (maximum deviation in degrees)

### `Rides/<ride>/intrinsics/0.csv`
- Type: CSV (comma-separated).
- Description: Calibrated camera instrinsics from the smartphone camera (60Hz)
- Columns:
  - `fx`: Float (px) | camera focal length in x
  - `fy`: Float (px) | camera focal length in y
  - `cx`: Float (px) | principal point x
  - `cy`: Float (px) | principal point y

### `Rides/<ride>/position/0.csv`
- Type: CSV (comma-separated).
- Description: Relativ position to starting point (60Hz)
- Columns:
  - `X`: Float (m) | lateral (+ right, - left)
  - `Y`: Float (m) | Vertical (+ up, - down)
  - `Z`: Float (m) | longitudinal (+ backwards, - forwards)

### `Rides/<ride>/trackingState/0.csv`
- Type: CSV (comma-separated).
- Description: State of the relativ position tracking. Only recorded at changes. If no value is present, tracking was normal
- Columns:
  - `state`: String ("Not Available"/"Limited"/"Normal")
  - `reason`: String (Description why tracking is limited)

### `Rides/<ride>/rawDepth/depth/<Timestamp>.bin`
- Type: Binary depth map.
- Filename: `<Timestamp>.bin` where `<Timestamp>` matches the `Timestamp` format `YYYY-MM-DD, HH:MM:SS.SSSS` (example: `2025-06-16, 12:47:48.2920.bin`).
- Description: Depthmap sampled by the smartphones LiDAR. Synchronized to the `color.mp4`. (60Hz)

- Content:
  - Data type: little-endian `float32` (`<f4`)
  - Values: depth in meters
  - Element count: `49152` values per file
  - Shape: `256 x 192` (width x height; row-major)

The following python code can be used to extract the grayscale depth image from the depthmap
```python
import array 
import numpy as np
import matplotlib.pyplot as plt

def binary_to_depth(path: str):
    with open(path, "rb") as f:
        data = f.read()
    float_array = array.array("f")
    float_array.frombytes(data)
    return float_array

def show_depthmap(path: str):
    distances = np.array(binary_to_depth(path))
    mat = distances.reshape((192,256))
    mat = np.rot90(mat, k=-1)
    plt.imshow(mat)
    plt.show()
```

### `Rides/yolo_detections.bin`
Contains for each frame of `color-mp4` the object detections of a yolo model fine tuned to detect road signs. Frames can be accessed by their timestamp (NTP). Each frame contains a dictionary of at least 1 confidence value. Only detection with higher confidence are stored. Each detection contains pixel coordinates (`x`,`y`), bbox dimensions (`width`,`height`), the class type (`class_name`), `confidence`, and the `track_id`. The track_id can be used to identify the same road sign in consecutive frames. Example:
```json
"2025-07-15 14:59:12.633000": {
    "0.80": [
      {
        "x": 1303,
        "y": 462,
        "width": 171,
        "height": 159,
        "class_name": "forb_stopping",
        "confidence": 0.9541036486625671,
        "track_id": 0
      }
    ]
  }
```
