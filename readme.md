# Jetson Orin Nano – YOLOv8 Pose Estimation in Docker

This repo documents how to run **Ultralytics YOLOv8-Pose** on an **NVIDIA Jetson Orin Nano** inside Docker with a USB webcam.

Tested on **JetPack 6.2.x (L4T r36.4.x)**.

---

## ✅ Requirements
- Jetson Orin Nano running JetPack 6.2.x (L4T r36.4.x)
- Docker with NVIDIA runtime
- USB/UVC webcam (e.g., `/dev/video0`)
- Internet access inside the container

> If `nvcr.io` pulls fail, either login with your NGC API key or use the fallback image `dustynv/l4t-pytorch:r36.4.0`.

---

## 🚦 Step-by-Step Guide

### 1. Allow X clients on the host
```bash
xhost +local:root
```

### 2. Run the container
```bash
sudo docker run -it --rm \
  --runtime nvidia --gpus all \
  --network host --ipc=host \
  -e DISPLAY=$DISPLAY -e QT_X11_NO_MITSHM=1 \
  -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
  --device /dev/video0:/dev/video0 \
  --group-add video \
  --name jetson-pose \
  nvcr.io/nvidia/l4t-jetpack:r36.4.0 bash
```
### 3. Inside the container

#### 3.1 Update and install pip
```bash
apt-get update
apt-get install -y python3-pip
```
#### 3.2 Install Ultralytics
```bash
python3 -m pip install ultralytics
```

---

### 4. Run pose estimation (headless save)
```bash
python3 -m ultralytics pose predict \
  model=yolov8n-pose.pt source=0 \
  show=False save=True imgsz=640 conf=0.4 iou=0.6
```
- Results are saved under `/workspace/runs/pose/predict*` inside the container.
- If you mount `~/yolo_results:/workspace/runs` when starting the container, results appear on the host at `~/yolo_results/pose/predict*`.

Stop with **Ctrl+C**.

---

### 5. (Optional) Live preview window
Install GUI libraries inside the container:
```bash
apt-get update
apt-get install -y \
  python3-opencv \
  libgl1 libglib2.0-0 libgtk-3-0 \
  libxkbcommon-x11-0 libxrender1 libxrandr2 libxi6 libxfixes3 \
  libxcb1 libxcb-icccm4 libxcb-image0 libxcb-keysyms1 \
  libxcb-render-util0 libxcb-shape0 libxcb-xinerama0 libxcb-xfixes0 \
  libqt5gui5 qtwayland5
```
Run with preview:
```bash
python3 -m ultralytics pose predict \
  model=yolov8n-pose.pt source=0 \
  show=True save=True imgsz=640 conf=0.4 iou=0.6
```
If window issues persist:
```bash
export QT_QPA_PLATFORM=xcb
```

---

## 🔧 Tips for Better Performance
- Max out Jetson clocks (on host):
  ```bash
  sudo nvpmodel -m 0
  sudo jetson_clocks
  ```
- Reduce input size:
  ```bash
  ... imgsz=480
  ```
- Use FP16:
  ```bash
  ... half=True device=0
  ```
- TensorRT export (faster inference):
  ```bash
  python3 -m ultralytics export model=yolov8n-pose.pt format=engine half=True
  python3 -m ultralytics pose predict model=yolov8n-pose.engine source=0 show=True
  ```

---

## 📂 Where to Find Results
- Inside container: `/workspace/runs/pose/predict*`

---

## 🛑 Common Fixes
- **`yolo: command not found`** → use `python3 -m ultralytics ...`
- **`No module named pip`** → `apt-get install -y python3-pip`
- **PyTorch missing CUDA** → install Jetson wheel from step 3.2
- **Camera not found** → check with `v4l2-ctl --list-devices`, try `source=1`

---

## 📝 License
MIT

