# Concert robot

This tutorial is aimed at helping to get the concert robot and IK control ready (for dummies).

## Setup

- Clone the https://github.com/ADVRHumanoids/concert_description repository (use recursive clone).
- Open the `concert_description/docker/run-docker.bash` file in a file editor and remove the `--rm` flag. This ensures that the container will not be discarded on exit.
- Run this bash file. A new terminal will pop up (in a newly created docker container). All the following commands must be executed in this terminal. 

DO NOT CLOSE THE TERMINAL USED TO RUN THE CONTAINER.

## Gazebo

- Split the terminal in 8-10 parts (you can use the `Ctrl+Shift +E/+O` shortcut to split vertically/horizontally).
- Run `roscore` in one terminal. If successful, it will start a ros core service and show a little summary.
- Run  `mon launch concert_gazebo concert.launch [rviz:=true]` in another terminal to launch the Gazebo world with the robot in it.

## IK

- Run `xbot2-gui` to launch the control panel. Press `ros_ctrl` and then `homing` to move the robot to a state from which it is ready to receive commands from the IK.
- Run `mon launch concert_cartesio concert.launch xbot:=true gui:=true` to launch the IK control in RViz. The robot model will have markers which can be used to control the robot in Gazebo. To use this markers, right click them and toggle the "Continuous Ctrl" on.
- Now you can move the markers in the RViz window (by dragging them) and the robot will react accordingly in the Gazebo world (avoding self intersection).
- To be able to control the robot's arm in RViz, unfold the right panel there, press "Add" -> "Interactive markers". In the new drop-down menu, locate the "Update topic" option and choose "/tcp_cartesian_marker_server/update". New markers will appear. Similarly, toggle the "Continuous Ctrl" for them, too.

## MoveIt

- Install MoveIt and its requirements: `sudo apt install ros-noetic-moveit` (see more here https://moveit.ros.org/install).


## URDF file

- To create a package, you will first need a `.urdf` (Unified Robot Description Format) file describing the ConcertRobot. You can generate such file by running one python script located at `~/concert_ws/ros_src/concert_description/concert_examples/concert_example.py` (still inside of the container):
    ```
    python3 concert_example.py -o urdf -a gazebo_urdf:=true realsense:=false velodyne:=false use_gpu_ray:=false -r modularbot_gz >> concert_robot.urdf
    ```
    This will generate a `concert_robot.urdf` which you can then use in MoveIt Setup Assistant

### Setup Assistant

- Run `roslaunch moveit_setup_assistant setup_assistant.launch` to launch the Setup Assistant. There you can create a ros package by following the steps from https://ros-planning.github.io/moveit_tutorials/doc/setup_assistant/setup_assistant_tutorial.html

