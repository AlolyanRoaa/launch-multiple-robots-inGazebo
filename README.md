# Launch Multiple Robots in Gazebo


This is documentation of how to launch multiple TurtleBot3 Burger robots in Gazebo simulator, and control them using teleop key. before following these instructions make sure that you have download, install, and build the downloaded package of TurtleBot3.


check [this repository](https://github.com/AlolyanRoaa/Creating-a-map-using-Turtlebot3-and-SLAM) to help you.


*Note: These instructions have been done on `Ubuntu 18.04.5 LTS bionic`, `ROS melodic`, and `TurtleBot3 robot`.*


## Outline


- Create a package.
- Move and control the robots using teleop key in Gazebo environment.
- Demo.
- Resources.


## 1-Create a package

start by opening a terminal in catkin_ws/src folder, then to create a package for this project run the following command where multi_robot is the package name

```bash
catkin_create_pkg multi_robot rospy gazebo_ros
```

![CreatePackage](https://github.com/AlolyanRoaa/launch-multiple-robots-inGazebo/blob/main/images/01-creatr%20package.PNG)


now create a folder called launch under the multi_robot package we just created, we will create 3 launch files in this folder.


![launchFolder](https://github.com/AlolyanRoaa/launch-multiple-robots-inGazebo/blob/main/images/02-create%20folder%20named%20launch.PNG)


in launch folder create the 3 launch files we need which are *one_robot.launch, robots.launch,* and *main.launch* using `touch ` command followed by the name of the file.


![launchFiles](https://github.com/AlolyanRoaa/launch-multiple-robots-inGazebo/blob/main/images/03-create%20launch%20files.PNG)


*one_robot.launch* is the file that will launch one robot in the simulation, which will contain the following script: 


```bash
<launch>
    <arg name="robot_name"/>
    <arg name="init_pose"/>

    <node name="spawn_minibot_model" pkg="gazebo_ros" type="spawn_model"
     args="$(arg init_pose) -urdf -param /robot_description -model $(arg robot_name)"
     respawn="false" output="screen" />

    <node pkg="robot_state_publisher" type="state_publisher" 
          name="robot_state_publisher" output="screen"/>

    <!-- The odometry estimator, throttling, fake laser etc. go here -->
    <!-- All the stuff as from usual robot launch file -->
</launch>
```

<img src="https://github.com/AlolyanRoaa/launch-multiple-robots-inGazebo/blob/main/images/04-one_robot.launch.PNG" width="600">


*robots.launch* which will open multiple robots. Note that you can operate more robots but make sure that the namespaces and tf_prefixes are different and the initial position for each robot is also different so that they will not overlap each other. this launch file will contain the following script:


```bash
<launch>
  <!-- No namespace here as we will share this description. 
       Access with slash at the beginning -->
  <param name="robot_description"
    command="$(find xacro)/xacro.py $(find turtlebot3_description)/urdf/turtlebot3_burger.urdf.xacro" />

  <!-- BEGIN ROBOT 1-->
  <group ns="robot1">
    <param name="tf_prefix" value="robot1_tf" />
    <include file="$(find multi_robot)/launch/one_robot.launch" >
      <arg name="init_pose" value="-x 1 -y 1 -z 0" />
      <arg name="robot_name"  value="Robot1" />
    </include>
  </group>

  <!-- BEGIN ROBOT 2-->
  <group ns="robot2">
    <param name="tf_prefix" value="robot2_tf" />
    <include file="$(find multi_robot)/launch/one_robot.launch" >
      <arg name="init_pose" value="-x -1 -y 1 -z 0" />
      <arg name="robot_name"  value="Robot2" />
    </include>
  </group>
</launch>
```


<img src="https://github.com/AlolyanRoaa/launch-multiple-robots-inGazebo/blob/main/images/05-robots.launch.PNG" width="600">


*main.launch* file is to launch the gazebo world simulation, you can choose any environment in worlds folder that is provided with TurtleBot3 robot.


![environments](https://github.com/AlolyanRoaa/launch-multiple-robots-inGazebo/blob/main/images/07-worlds.PNG)


```bash 
<launch>
  <param name="/use_sim_time" value="true" />

  <!-- start world -->
  <node name="gazebo" pkg="gazebo_ros" type="gazebo" 
   args="$(find turtlebot3_gazebo)/worlds/turtlebot3_world.world" respawn="false" output="screen" />

  <!-- start gui -->
  <!-- <node name="gazebo_gui" pkg="gazebo" type="gui" respawn="false" output="screen"/> -->

  <!-- include our robots -->
  <include file="$(find multi_robot)/launch/robots.launch"/>
</launch>
```


<img src="https://github.com/AlolyanRoaa/launch-multiple-robots-inGazebo/blob/main/images/06-main.launch.PNG" width="600">


to launch the simulation with the two robot use the following command


```bash
roslaunch multi_robot main.launch
```

![simulatedInEmpty](https://github.com/AlolyanRoaa/launch-multiple-robots-inGazebo/blob/main/images/08-two%20robot%20are%20simulated%20in%20empty.PNG)
*Note: in this picture we use empty.world environment*


![turtlebot3_world](https://github.com/AlolyanRoaa/launch-multiple-robots-inGazebo/blob/main/images/09-two%20robot%20are%20simulated%20in%20gazebo%20env.PNG)
*with turtlebot3_world.world file*


## Move and control the robots using teleop key in Gazebo environment


in new terminal check the topics using `rostopic list`then you will see 2 cmd_vel topics for each robots.


![rostopict](https://github.com/AlolyanRoaa/launch-multiple-robots-inGazebo/blob/main/images/10-rostopit%20list%20command.PNG)


to control them using teleop_twist, run the command below in a new terminal.


```bash 
rosrun teleop_twist_keyboard teleop_twist_keyboard.py /cmd_vel:=/robot1/cmd_vel
```

Note that if you want to control a different robot all you need is to remap cmd_vel to the robot you want.


## Demo


https://user-images.githubusercontent.com/85321139/127918971-5e0fabda-5226-41cd-9419-b79be3ba0224.mp4


## Resources


- https://www.theconstructsim.com/ros-qa-130-how-to-launch-multiple-robots-in-gazebo-simulator/
- https://answers.ros.org/question/41433/multiple-robots-simulation-and-navigation/













