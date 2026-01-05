# ROS1–ROS2 Bridge on NVIDIA Jetson (Kernel 6.2)

This repository documents how to set up **ROS1 Noetic** and the **ROS1–ROS2 bridge (`ros1_bridge`)** using **Docker** on an NVIDIA Jetson platform.

The setup uses:

* ROS1 Noetic (Docker)
* ROS2 Humble (Jetson L4T Docker)
* `ros1_bridge` built via a dedicated Docker builder
* `rocker` for running ROS1 containers with GUI support

---

## Prerequisites

* Ubuntu (native or Jetson Ubuntu)
* Docker installed
* NVIDIA Jetson with JetPack / L4T
* X11 enabled (for GUI terminals)

---

## 1. Install Docker (Native Ubuntu)

If Docker is already installed, skip to **Build the ros-humble-ros1-bridge package**.

```bash
sudo apt update
sudo apt install -y docker.io
```

Add your user to the Docker group:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

> ⚠️ If you encounter permission errors later, **log out completely and log back in**.

### Verify Docker Installation

```bash
sudo service docker start
sudo service docker status

docker info
docker run -it --rm hello-world
```

---

## 2. Install Rocker

Rocker is used to run ROS containers with host integration (X11, networking, user mapping).

### Install Python Virtual Environment Support

```bash
sudo apt install -y python3-venv
```

### Create and Activate a Virtual Environment

```bash
mkdir -p ~/rocker_venv
python3 -m venv ~/rocker_venv
source ~/rocker_venv/bin/activate
```

### Install Rocker

```bash
pip install git+https://github.com/osrf/rocker.git
```

> 📌 **For every new terminal**, remember to activate the virtual environment:

```bash
source ~/rocker_venv/bin/activate
```

---

## 3. Build the ROS2 Humble – ROS1 Bridge Package

### Clone the Builder Repository

```bash
cd ~
git clone https://github.com/TommyChangUMD/ros-humble-ros1-bridge-builder.git
```

### Build the Docker Image

```bash
cd ros-humble-ros1-bridge-builder
docker build . -t ros-humble-ros1-bridge-builder
```

### Build the ros1_bridge Workspace

```bash
cd ~
docker run --rm ros-humble-ros1-bridge-builder | tar xvzf -
```

This will generate a `ros-humble-ros1-bridge` workspace in your home directory.

---

## 4. Run ROS1 Noetic (Terminal 1)

Start a ROS1 Noetic container using Rocker:

```bash
rocker --x11 --user --home --privileged \
  --volume /dev/shm:/dev/shm \
  --network=host \
  ros:noetic-ros-base-focal \
  'bash -c "sudo apt update && sudo apt install -y ros-noetic-rospy-tutorials tilix && tilix"'
```

Inside the container terminal:

```bash
source /opt/ros/noetic/setup.bash
roscore
```

---

## 5. Run ROS2 Humble (Terminal 2)

### Pull the ROS2 Humble Jetson Image

```bash
docker pull dustynv/ros:humble-desktop-l4t-r36.4.0
```

### Start the ROS2 Container

```bash
docker run -it --rm \
  --network=host \
  --env DISPLAY=$DISPLAY \
  -e RMW_IMPLEMENTATION=rmw_cyclonedds_cpp \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v $HOME:/home/user \
  dustynv/ros:humble-desktop-l4t-r36.4.0
```

---

## 6. Start the ROS1–ROS2 Bridge (Terminal 3)

Run the dynamic bridge:

```bash
docker run --rm -it \
  --network=host \
  -e RMW_IMPLEMENTATION=rmw_cyclonedds_cpp \
  -v $HOME:/home/user \
  dustynv/ros:humble-desktop-l4t-r36.4.0 \
  bash -c "source /home/user/ros-humble-ros1-bridge/install/local_setup.bash && \
           export ROS_MASTER_URI=http://10.81.162.71:11311 && \
           ros2 run ros1_bridge dynamic_bridge --bridge-all-topics"
```

> 🔧 **Note:**
> Replace `10.81.162.71` with the IP address of the machine running `roscore`.

---

## Architecture Overview

```
ROS1 Noetic (Docker) ──► roscore
        ▲
        │
   ros1_bridge
        │
        ▼
ROS2 Humble (Jetson Docker)
```

---

## Notes & Tips

* Use `--network=host` for all containers to simplify DDS and ROS networking.
* Ensure `RMW_IMPLEMENTATION=rmw_cyclonedds_cpp` is consistent across ROS2 containers.
* For debugging:

  ```bash
  ros2 topic list
  rostopic list
  ```

---

## References

* [https://github.com/TommyChangUMD/ros-humble-ros1-bridge-builder](https://github.com/TommyChangUMD/ros-humble-ros1-bridge-builder)
* [https://github.com/osrf/rocker](https://github.com/osrf/rocker)
* [https://github.com/dusty-nv/jetson-containers](https://github.com/dusty-nv/jetson-containers)


Just tell me 👍
