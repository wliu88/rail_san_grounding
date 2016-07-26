# Seeding Situated Affordance Network (SAN) From Vision Pipeline

This document introduces seeding SAN from object recognition using Stanley's Vector robot. Previously on tablebot, a planner for jaco arm is used in model training process to move the gripper to a candidate grasp pose. This pose along with the closest point cloud is used for contructing the object model. However, this planner is not availble for Vector robot for now. The easy way around this is to modify the model training code and control the arm manually using the joystick. After the object models are trained correctly, object recognition can be used. The next problem is to seed the recognized object names to SAN in ROS.

## Preparations
In order to use object recognition, ros package `rail_segmentation` and `rail_pick_and_place` need to be installed. Detailed documentation about these two packages can be found in [rail_pick_and_place wiki](http://wiki.ros.org/rail_pick_and_place), [rail_recognition wiki](http://wiki.ros.org/rail_recognition), and [rail_segmentation wiki](http://wiki.ros.org/rail_segmentation).

1. Install `rail_segmentation` and `rail_pick_and_place` packages in catkin workspace. I installed them in my computer's catkin workspace, not in vector1 or vector2.

2. Configure `rail_segmentaion`
  * Modify frame ids in `rail_segmentation/config/zones.yaml`. I used following for Vector robot:
  ```
  parent_frame_id: "base_footprint"
  child_frame_id: "camera_depth_frame"
  bounding_frame_id: "base_footprint"
  segmentation_frame_id: "base_footprint"
  ```
  * Modify point cloud topic in `rail_segmentation/src/Segmenter.cpp`. I used `/camera/depth_registered/points` for Vector robot.

3. Configure `rail_pick_and_place`
  *  Set up grasp database according to instruction in [graspdb wiki](http://wiki.ros.org/graspdb?distro=indigo). I encountered some problems     in this step. There are more tutorials on how to setup PostgreSQL online. 
  *  Connect to grasp database by modifying all relevant launch files in `rail_pick_and_place` following the same procedures in *Connecting the Collection Node, your Database, and your Robot* section [here](http://wiki.ros.org/rail_pick_and_place/Tutorials/Collecting%20object%20model%20data%20and%20grasp%20demonstrations).
  *  Comment out the code section requesting a grasp from arm (*around line 110*) in `rail_pick_and_place/rail_grasp_collection/src/GraspCollector.cpp`. This allows model training without using jaco arm planner.

## Training Object Models
1. Bring up Vector robot. Launch joystick teleop. Make sure that jaco arm can be controlled by joystick. If not, ssh to vector1 and run the following: `rosrun vector_ros jaco_cartesian_vel_ctr` after commenting out this command from the vector1 launch file. This is only necessary if the jaco arm is not initialized correctly upon bringup of Vector robot.

2. Run `rosrun rail_segmentation rail_segmentation` to start the segmentation node. Segmentation can be called from the command line with `rosservice call /rail_segmentation/segment "{}"`, which will then automatically trigger recognition, which will automatically update the world model, etc.

3. Train 2D image recognizer according to *Setting up the 2D Image Recognizer* section in [rail_recognition wiki](http://wiki.ros.org/rail_recognition).

4. Train object models according to *Object Model Training* section in [rail_pick_and_place tutorials](http://wiki.ros.org/rail_pick_and_place/Tutorials) except use the joystick to move the jaco arm to candidate grasp pose. Remember to segment the scene between grasp demonstations so that each grasp model will comprise different point clouds. 

## Test Object Recognition
Running `roslaunch rail_recognition object_recognition_listener` should print out recognized object names.

## Seeding Recognized Objects to SAN
Work to be continued. Migrating `ProbCog` package to ROS is a problem now.
