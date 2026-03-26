# ROS 2 Pick and Place Project

## Project Overview
This project implements a basic pick-and-place pipeline in ROS 2 using the Academy Robotics workspace. The goal of the system is to detect an object from RGBD point cloud data, estimate its position in the environment, and send this information to a robot arm for motion planning using MoveIt. The project focuses on integrating perception, ROS 2 communication (topics, services, and actions), and motion planning within a simulated environment.

---
## System Architecture
The system is composed of three main components: perception, communication, and motion planning. These components are connected through ROS 2 topics, services, and actions to form a complete pipeline from object detection to robot motion execution.

---

## Implemented Components

### Object Detection
The object detection module is implemented in a node named `object_detector_node`.

This node subscribes to the point cloud topic:
`/front_rgbd_camera/depth/color/points`

It processes the incoming `PointCloud2` data by:
- filtering valid (finite) points  
- reducing noise and irrelevant data  
- computing the centroid of the detected object  

The detected pose is then published to:
`/detected_object_pose`

The pose is expressed in the `robot_base_link` frame, making it directly usable for motion planning.

---

### Detection Service
A custom service `DetectObject.srv` is implemented and exposed at:
`/detect_object`

This service allows other nodes to request the detected object pose when needed. It returns:
- a boolean indicating whether detection was successful  
- the estimated pose of the object  

This approach ensures that detection results can be accessed both continuously (via topic) and on-demand (via service).

---

### Pick and Place Action
A custom action `PickPlace.action` is implemented and available at:
`/pick_place_task`

This action provides a structured way to send a target pose from the client to the server. It allows asynchronous communication and supports feedback and result handling.

---

### Pick and Place Server
The pick-and-place server receives the target pose from the client and uses MoveIt to perform motion planning.

The server:
- sets the target pose for the robot arm  
- attempts to compute a valid trajectory  
- executes the trajectory in the simulation  

The planning is performed using the MoveIt group:
`ur_arm`

The server acts as the bridge between high-level commands (pose goals) and low-level robot motion.

---

### Pick and Place Client
The client coordinates the entire pipeline. It performs the following sequence:

1. Calls the `/detect_object` service  
2. Receives the detected object pose  
3. Sends this pose as a goal to `/pick_place_task`  

This connects perception and motion planning into a complete workflow.

---

### Launch File
A launch file named `pick_place.launch.py` is created to simplify execution.

This launch file starts:
- the object detector node  
- the pick-and-place server  
- the pick-and-place client  

Using this file, the system can be executed with a single command instead of multiple terminals.

---

## Build Instructions

The project is built inside the Docker environment using the following commands:

```bash
cd /home/user/ros2_ws
source /opt/ros/humble/setup.bash
colcon build --symlink-install
source install/setup.bash

Running the Project

1. Launch the simulation world
ros2 launch academy_robot_gazebo_ignition spawn_world.launch.py spawn_screw:=true
2. Spawn the robot

ros2 launch academy_robot_gazebo_ignition spawn_robot.launch.py robot:=academy_robot robot_model:=academy_robot_plus has_arm:=true

3. Run the project pipeline
ros2 launch pick_place_project pick_place.launch.py

Current Status

The system successfully integrates perception, service-based communication, and action-based coordination. The object detection pipeline is functional, and the client-server interaction is correctly established. The launch file allows the system to be executed in a structured and simplified manner.
