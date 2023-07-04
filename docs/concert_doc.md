# Concert robot

This tutorial is aimed at helping to get the concert robot and IK control ready (for dummies).

## Setup

- Clone the concert_description repository (use recursive clone):
```
git clone --recursive https://github.com/ADVRHumanoids/concert_description
```
- Open the `concert_description/docker/run-docker.bash` file in a file editor and remove the `--rm` flag. This ensures that the container will not be discarded on exit.
- Run this bash file. A new terminal will pop up (in a newly created docker container). All the following commands must be executed in this terminal. 

DO NOT CLOSE THE TERMINAL USED TO RUN THE CONTAINER.

## Gazebo

- Split the terminal in 6-8 parts (you can use the `Ctrl+Shift +E/+O` shortcut to split vertically/horizontally).
- You may want to install a file explorer GUI to navigate files inside of the container; `nautilus` is normally more than sufficient: `sudo apt-get install nautilus`.
- Run `roscore` in one terminal. If successful, it will start a ros core service and display a little summary.
- Run  `mon launch concert_gazebo concert.launch [rviz:=true]` in another terminal to launch the Gazebo world with the robot in it.

## IK

- Run `xbot2-gui` to launch the control panel. Press `ros_ctrl` and then `homing` to move the robot to a state from which it is ready to receive commands from the IK.
- Run `mon launch concert_cartesio concert.launch xbot:=true gui:=true` to launch the IK control in RViz. The robot model will have markers which can be used to control the robot in Gazebo. To use this markers, right click them and toggle the "Continuous Ctrl" on.
- Now you can move the markers in the RViz window (by dragging them with the mouse) and the robot will react accordingly in the Gazebo world (avoding self intersection).
- To be able to control the robot's arm in RViz, unfold the right panel there, press "Add" -> "Interactive markers". In the new drop-down menu, locate the "Update topic" option and choose "/tcp_cartesian_marker_server/update". New markers will appear. Similarly, toggle the "Continuous Ctrl" for them, too.

## MoveIt

- Install MoveIt and its requirements: `sudo apt install ros-noetic-moveit` (see more here https://moveit.ros.org/install).

## URDF file

- To create a package, you will first need a `.urdf` (Unified Robot Description Format) file describing the ConcertRobot. You can generate such file by running one python script located at `~/concert_ws/ros_src/concert_description/concert_examples/concert_example.py` (still inside of the container):
    ```
    python3 concert_example.py -o urdf -a gazebo_urdf:=true realsense:=true velodyne:=true use_gpu_ray:=true -r modularbot_gz >> concert_robot.urdf
    ```
    This will generate a `concert_robot.urdf` which you can then use in MoveIt Setup Assistant

### Setup Assistant

- Run `roslaunch moveit_setup_assistant setup_assistant.launch` to launch the Setup Assistant. There you can create a ros package by following the steps from https://ros-planning.github.io/moveit_tutorials/doc/setup_assistant/setup_assistant_tutorial.html

#### Creating a package: following the steps
Here is how I did it:
#### start
- launch the Setup Assistant using the command from above,
- choose "create new moveit configuration package",
- choose the `.urdf` file (supposedly, created by, some of the above steps) and 
- press "load files";
#### self-collisions
- set the "sampling density to high" (it doesn't really affect the computation speed, but may yield higher accuracy),
- press "generate collision matrix",
- select the "linear view" radio-button: this way you go through the pairs of joints; note that selected joints light up on the model to the right - this allows us to decypher the joint names (we will need them);
#### virtual joints (needs checking)
- create a new virual joint:
    ```
    name: virtual_joint
    child link: base_link
    parent frame name: mobile_base
    joint type: fixed
    ```
    Supposedly, this attaches the arm to the mobile base.

#### planning groups
- create a new planning group:
    ```
    name: controller
    kinematic solver: KDLKinematicsPlugin
    group default planner: RRTConnect
    ```
    Leave the rest at default.

- add the joints responsible for moving the arm and the base to this planning group (_I think_):
    ```
    arm joints:
        J1_E, J2_E, J3_E, J4_E, J5_E, J6_E,
        L1_E, L2_E, L3_E, L4_E, L5_E, L6_E,
    wheel joints:
        J1_A_stator, J1_B_stator, J1_C_stator, J1_D_stator (rotate the joints to which the wheels are attached)
        L1_A, L1_B, L1_C, L1_D
        (rotate the wheels)
    ```

#### robot poses
Add a few robot poses to plan between. I always add the "home" pose -- all joints rotations set to zero and the robot arm is as vertical as it can get.

#### controllers
Press the "auto add FollowJointsTrajectory Controllers for each planning group"

#### 3d perception
Choose the "depth map" option.

#### generate package
(Specify the maintainer first)

Create a new folder with the future package name and generate the package.

### Setting up catkin workspace
We need to reach the following file structure (_only the most important files are listed here_):
```
├── concert_ws/
│   ├── setup.bash
│   ├── ros_src/
│   |   ├──concert_description/     (this is the repo)
├── ws_moveit/
│   ├── devel/
│   |   ├──setup.bash               (generated by `catkin build`)
│   ├── src/
│   |   ├──concert_moveit_config/   (generated by the moveit setup assistant and placed here by us)
│   |   ├──concert_resources/       (copied from ros_src/concert_description by us)
│   |   ├──modular_resources/       (copied from ros_src/concert_description by us)
│   |   ├──concert_examples/        (copied from ros_src/modular/src/modular by us)
│   ├── build/                      (generated by `catkin build`)
```

To setup the catkin workspace, follow the steps from this "getting started" tutorial: https://ros-planning.github.io/moveit_tutorials/doc/getting_started/getting_started.html. 

Before running `catkin build`, place your moveit package (`concert_moveit_config`) as well as `concert_resources`, `modular_resources` and `concert_examples` inside the `ws_moveit/src/` directory.


### Using the package
Now we can use the `concert_moveit_config` package that we have just created. _Note_: we're still inside of the container.

- `cd ~/ws_moveit/src/`
- `catkin build`
- (_OPTIONAL_) `sudo apt-get update && sudo apt-get dist-upgrade` to prevent rviz packages from failing
- `source ~/concert_ws/setup.bash && source ~/ws_moveit/devel/setup.bash`
- `roslaunch concert_moveit_config demo.launch rviz_tutorial:=true`

_Note_: if the last command shows no errors, but gets stuck with the frozen rviz window saying "initializing", try restarting the **roscore** and rerun the last 5 bullet points.

A new Rviz window will open. There you can plan and execute "actions" you've created in the Setup Assistant.
