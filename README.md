# Cartographer Documentation

## Step 1 - Installation and Compiling
___

Follow the tutorial below to setup Cartographer on your PC. \
``
NOTE: IT IS RECOMMENDED TO CREATE A SEPARATE WORKSPACE SPECIALLY FOR CARTOGRAPHER
``


[Compiling Cartographer ROS](https://google-cartographer-ros.readthedocs.io/en/latest/compilation.html#building-installation)

NOTE: If you encounter this problem when installing:\
``
 ERROR: the following packages/stacks could not have their rosdep keys resolved to system dependencies: cartographer: [libabsl-dev] defined as "not available" for OS version [focal] 
``

[Check this thread to solve it](https://github.com/cartographer-project/cartographer_ros/issues/1726)



## Step 2 - Getting Ready to use Cartographer
___

#### Some important Information about Cartographer
### __Configuration Files__

1. Alongside Launch files, Cartographer also makes use of Config files
2. The config and launch files can be in a different workspace. 
3. It is **recommended to keep them in you main workspace** since the cartographer workspace can take significantly longer to make
2. Config files are used to set the parameters Cartographer needs to run
3. In config files, every distance is defined in **meters** and every time is in **seconds**
4. Ideally, a .lua configuration should be robot-specific. e.g. a config for each base (mecanum, omni) and different sensors configuration (LIDAR only, LIDAR+kinect, LIDAR+Odom)
5. In the repo you will find multiple configurations already pre-made for our use cases. 

 For more details on the parameters and what they do check the [Config Reference Documentation](https://google-cartographer-ros.readthedocs.io/en/latest/configuration.html)
    
### __Topics__
1. To work, Cartographer needs to subscribe to a range sensor E.g. LIDAR. the name of the topic can be specified in the .lua config file
2. Additionally, Carographer also accepts IMU and Odomotery data which will be used to improve the algorithm.
    - To use IMU data set TRAJECTORY_BUILDER_2D.use_imu_data to true
    - To use Odometery data, set use_odomotery to true
    - NOTE: if use_odomotery is true, then provide_odom_frame **MUST** be false and vice versa
3. If you are going to provide Laser scan and IMU data to Cartographer, the topics **MUST** be named **"scan"** and **"imu"**


### __Frames and Transforms__
1. Alongside the sensor topics, you probably have to provide the TF frame IDs of your environment and robot in **map_frame**, **tracking_frame**, **published_frame** and **odom_frame**.
2. You can do this either through defining it in a **.urdf** file or through the robot's TF tree through a **/tf** topic.
3. The names of the frames can be defined in the .lua configuration file 
4. The proper TF tree should be **WIP -> WIP -> WIP -> WIP**. Check the image below for how it looks.

IMAGE PLACE HOLDER 

- To do this, one static transform is defined in the rplidar launch file that transforms from base_link to laser
    - **MAKE SURE YOU CHANGE THE X and Y VALUES IF YOU CHANGE LIDAR POSITION OR CHANGE BASES**
- The second transofrm is defined in the cartographer launch file 
- Finally, robot localization takes care of the last transform\
    ``
    IF YOU ENCOUNTER ANY PROBLEMS RELATED TO TRANSFORMS MAKE SURE YOUR TF TREE AND FRAMES ARE DEFINED PROPERLY
    ``
    
    ``
    Aditionally, If you are using odomotery data but the /odom Topic is not publishing or has a different name, you may encounter a TF error message
    ``

## Step 3 - Running Cartographer
___

### **Running SLAM**
 * **To setup a launch file for Cartographer, you need to launch two nodes.**
 * **Before running the launch file, make sure to run all other launch files necessary (LIDAR, Odomotery, IMU).

**First is crtogrpaher_node.**
* The node takes a few arguments to set up.
* The main arguments are:
    *    ``-configuration_directory`` which specifies the directory of the config file
    * ``-configuration_basename`` which specifies the name of the conffig file you are going to use
* For more information about the arguments of thi node, refer to the [ API Documentation](https://google-cartographer-ros.readthedocs.io/en/stable/ros_api.html)
* Here is a basic example of how it can be written in a launch file
    ```XML
    <node name="cartographer_node" pkg="cartographer_ros"
        type="cartographer_node" args="
            -configuration_directory
                $(find config_directory)/config
            -configuration_basename sample.lua"
        output="screen">
    </node>
    ```
**Second is crtogrpaher_occupancy_grid_node.**

* This node is very important to us as it provides us with the map of the occupancy grid. 
* We can then visualize this map in RViz
    ```XML
    <node name="cartographer_occupancy_grid_node" pkg="cartographer_ros"
            type="cartographer_occupancy_grid_node" args="-resolution 0.05" />
    ```
* **You will find some launch files related to our use cases in launch/cartographer in our repo.**


### **Running Localization Only**

* **To use Cartographer in localization only mode, you need a ``.pbstream`` file of your map**
* Next is to run cartographer_node with a ```-load_state_filename``` argument and by defining the following line in your lua config:

    ```lua
    TRAJECTORY_BUILDER.pure_localization_trimmer = {
        max_submaps_to_keep = 3,
    }
    ```
