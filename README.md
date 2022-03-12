Collection of ROS2 packages for different humanoid robots 
=========================================================

For each robot an URDF description and MoveIt configuration is provided. Additionally PROTO model files for the Webots simulator are provided.

The goal of this repo is to allow people to easily test their software on different robot platforms. This allows validating if the algorithm is generally applicable, as well as direct comparison between achieved performances on a platform. For example, walk algorithms can be tested on all platforms and the achieved speeds on each robot can be compared.


How to create a new robot based on a proto file (documentation WIP)
-------------------------------------------------------------------

1. Create a folder with `ROBOT_NAME_description/protos`
2. Place proto and all texture/mesh files in this folder
3. Copy worlds folder from another robot into `ROBOT_NAME_description`
4. Change the name of the robot in all the .wbt files to the one of the new proto
5. Run webots and open the world file to confirm that it is working. You should see the robot. Deal with any error messages (except not finding "hl_supervisor" that is okay).
6. Copy the folders `[config, launch, urdf]` and the files `[CMakeLists.txt, package.xml]` from `ANOTHER_ROBOT_NAME_description` to `ROBOT_NAME_description`
7. Replace all occurrences of `ANOTHER_ROBOT_NAME_description` with `ROBOT_NAME_description` in all the files (including filenames)
8. Link the ROBOT_NAME_description directory in your ROS2 workspace (go to `ROS2_WORKSAPCE/src` directory and run `ln -s SOME_PATH/ROBOT_NAME_description`)
9. Build the package (colcon build --packages-up-to `ROBOT_NAME_description`). Fix any compilation errors.
10. Commit your changes until here, as we will need to change something in the proto for a moment.
11. Change all HingeJointsWithBacklash to HingeJoints (the proto might already have a varialbe for this, otherwise use find/replace).
12. Open the world file again in webots to check if your model is still working. Fix any issues.
13. Source your ROS2 workspace in a terminal (`source SOME_PATH/ROS2_WORKSAPCE/install/setup.bash`)
14. Edit the `webots_robot_controller.py` in `wolfgang_webots_sim` and uncomment the lines (or add them if missing in `__init__()`)
    ``` 
    with open("urdf_export.xml", "w+") as f:
            f.writelines(self.robot_node.getUrdf())
    ``` 
15. Run `ros2 launch rl wolfgang_webots_sim simulation.launch robot_type:=ROBOT_NAME`. It should create an `urdf_export.xml` in your current directory. Even if there are errors.
16. Copy the content of the `urdf_export.xml` file to the `urdf/robot.urdf` file in the `ROBOT_NAME_description` folder.
17. Revert the changes to the proto file regarding the HingeJointsWithBacklash.
18. Make the model follow ROS standards: manually add `l_sole` and `r_sole` frames if missing. Make sure that the `base_link` frame is between the first leg joints.
19. Use the moveit_setup_assistant to create a `ROBOT_NAME_moveit_config` folder next to the `ROBOT_NAME_description` folder. See the MoveIt documentation for more information. You will need to create `LeftLeg`, `RightLeg` and `Legs` planning groups like in the other robots. At time of writing, the setup assistant is not working for ROS2, so you need to use the ROS1 version and change some files (compare to the other robots to see how).
20. Build the package (colcon build --packages-up-to `ROBOT_NAME_moveit_config`). Fix any compilation errors.
21. Source your ROS2 workspace again, as we have a new package (`source SOME_PATH/ROS2_WORKSAPCE/install/setup.bash`)
22. Run `ros2 launch ROBOT_NAME_description standalone.launch`. You should get an rviz2 view with your robot. Use the sliders to check if everythin is okay. You might need to fix the joint limits (especially in the knees). Just edit these values in the URDF. Validate that the `x_sole` and `imu_frame` are correctly positioned and rotated.
23. Edit the `webots_robot_controller.py` in `wolfgang_webots_sim` in the `__init__()` method by adding your robot to the other ones. Look into your proto to see what the names are.
24. You should now be able to run `ros2 launch wolfgang_webots_sim simulation.launch robot_type:=ROBOT_NAME` and get a simulation of your robot.


Using the bitbots_quintic_walk with your robot
----------------------------------------------
1. If you want to use the walking you need to add a config file in the `bitbots_quintic_walk/config` folder (see the other ones as examples).
2. Build the `bitbots_quinti_walk` package afterwards as it needs to copy the new config file. 
3. Then you should be able to start the walking with `ros2 launch bitbots_quintic_walk test.launch sim:=true robot_type:=ROBOT_NAME`.
4. Also start the simulation with `ros2 launch wolfgang_webots_sim simulation.launch camera:=false robot_type:=ROBOT_NAME`.
5. You can use teleop to control the robot `ros2 run bitbots_teleop teleop_keyboard`
6. If your robot is not moving, your parameters are probably wrong and lead to unsolvable IK goals. Try to change the `trunk_height` parameter.
7. Automatically optimize the parameters (TODO more documentation on this)
