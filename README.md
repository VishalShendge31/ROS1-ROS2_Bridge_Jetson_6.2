# ROS1-ROS2_Bridge_Jetson_6.2
### Set up ROS1 Noetic and ros1_bridge package using docker.

Now let's set the ROS1 Noetic and ros1_bridge using docker. If you have Docker installed you can skip to the *Build the ros-humble-ros1-bridge package* step.

**Install Docker (for native Ubuntu)**
sudo apt install docker.io 

sudo usermod  -aG docker $USER   # add yourself to the "docker" group
newgrp docker  # or logout and re-login completely (not just opening a new terminal)

**Verify installation**
sudo service docker start
sudo service docker status

# If you get the permission error below you will need to log out of Linux completely 
# (not just exit the terminal) and re-login to have the docker group take effect.
docker info    # verify docker is running
docker run -it --rm hello-world

**Install Rocker**
sudo apt-get install python3-venv

#Create a venv
mkdir -p ~/rocker_venv
python3 -m venv ~/rocker_venv

#Install rocker
cd ~/rocker_venv
. ~/rocker_venv/bin/activate
pip install git+https://github.com/osrf/rocker.git

For any new terminal re activate the venv before trying to use it.
. ~/rocker_venv/bin/activate

**Build the ros-humble-ros1-bridge package**
Clone the ros-humble-ros1-bridge package,
cd
git clone https://github.com/TommyChangUMD/ros-humble-ros1-bridge-builder.git

and build the docker image.
cd ros-humble-ros1-bridge-builder/
docker build . -t ros-humble-ros1-bridge-builder

Now, build the ros-humble-ros1-bridge package.
cd ~/
docker run --rm ros-humble-ros1-bridge-builder | tar xvzf -

Use ros-humble-ros1-bridge
Terminal-1: Run the ROS1 Noetic roscore using the container, 
rocker --x11 --user --home --privileged \
  --volume /dev/shm /dev/shm \
  --network=host -- ros:noetic-ros-base-focal \
  'bash -c "sudo apt update && sudo apt install -y ros-noetic-rospy-tutorials tilix && tilix"'
  
This will open a new terminal for the container, start the roscore in that terminal,
source /opt/ros/noetic/setup.bash
roscore

Install ros2 image:
docker pull dustynv/ros:humble-desktop-l4t-r36.4.0

start thr ros2 docker:
docker run -it --rm \
		  --network=host \
		  --env DISPLAY=$DISPLAY \
		  -e RMW_IMPLEMENTATION=rmw_cyclonedds_cpp \
		  -v /tmp/.X11-unix:/tmp/.X11-unix \
		  -v $HOME:/home/user \
		  dustynv/ros:humble-desktop-l4t-r36.4.0


start the bridge:
docker run --rm -it \
		  --network=host \
		  -e RMW_IMPLEMENTATION=rmw_cyclonedds_cpp \
		  -v $HOME:/home/user \
		  dustynv/ros:humble-desktop-l4t-r36.4.0 \
		  bash -c "source /home/user/ros-humble-ros1-bridge/install/local_setup.bash && \
			   export ROS_MASTER_URI=http://10.81.162.71:11311 && \
			   ros2 run ros1_bridge dynamic_bridge --bridge-all-topics"
