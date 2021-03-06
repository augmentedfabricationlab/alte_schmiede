# Robot Simulation Environment @Alte Schmiede

**Quick links:** [compas docs](https://compas-dev.github.io/main/) | [compas_fab docs](https://gramaziokohler.github.io/compas_fab/latest/) | [urdf and moveit tutorials](https://gramaziokohler.github.io/compas_fab/latest/examples/03_backends_ros/07_ros_create_urdf_ur5_with_measurement_tool.html) | [simulation playground](#Alte-Schmiede-Simulation-Playground) | [troubleshooting](#docker-troubleshooting)

## Requirements

* Operating System: **Windows 10 Pro** <sup>(1)</sup>.
* [Rhinoceros 3D 6.0](https://www.rhino3d.com/): Focus on Rhino 6.0 only. [See here if you use Rhino 5.0](#rhino-50)
* [Anaconda Python Distribution](https://www.anaconda.com/download/): 3.x
* [Docker Community Edition](https://www.docker.com/get-started): Download it for [Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows). Leave "switch Linux containers to Windows containers" disabled.
* X11 Server: On Windows use [XMing](https://sourceforge.net/projects/xming/), on Mac use [XQuartz](https://www.xquartz.org/) (see details [here](https://medium.com/@mreichelt/how-to-show-x11-windows-within-docker-on-mac-50759f4b65cb)).
* Git: [official command-line client](https://git-scm.com/) or visual GUI (e.g. [Github Desktop](https://desktop.github.com/) or [SourceTree](https://www.sourcetreeapp.com/))
* [ABB RobotStudio](https://new.abb.com/products/robotics/robotstudio/downloads): 6.08 (only available for Windows). After install, **make sure you add RobotWare 6.03.02** (`Add-Ins` -> `RobotApps` -> `RobotWare` and add `6.03.02`).
* [VS Code](https://code.visualstudio.com/) with the following `Extensions`:
  * `Python` (official extension)
  * `EditorConfig for VS Code` (optional)
  * `Docker` (official extension, optional)

<sup>(1): Windows 10 Home does not support running Docker.</sup>


## Getting started

### Compas and Compas Fab Installation
    
    (base)  conda config --add channels conda-forge
    (base)  conda create -n your_env_name python=3.8 compas=0.15.6 compas_fab=0.11.0 --yes
    (base)  conda activate your_env_name
    (your_env_name) python -m compas_rhino.install -v 6.0
    (your_env_name) python -m compas_fab.rhino.install -v 6.0
    
### Verify Installation

    (your_env_name) python
    >>> import compas_fab
    >>> compas_fab.__version__
    '0.11.0'
    >>> exit()
    

### Repository Cloning
Then, clone [this repository](https://github.com/augmentedfabricationlab/alte_schmiede) into your workspace/project folder.  

## Loading Robot Model

* For starting the simulation, open the [rhino/robotic_setup_alte_schmiede.3dm](rhino/robotic_setup_alte_schmiede.3dm) and [rhino/robotic_setup_alte_schmiede_artist.ghx](rhino/robotic_setup_alte_schmiede_artist.ghx) file.
* First, you need to load a specified robot model by pressing the `load` button (you can choose the model from a list of urdf files).
* Once, the model is loaded, you can add and remove a tool in the `Tool` cluster and manipulate the joints with the sliders in the `Configuration` cluster.

## Simulation Playground

* Once you opened the file [rhino/robotic_setup_alte_schmiede.3dm](rhino/robotic_setup_alte_schmiede.3dm) and [rhino/robotic_setup_alte_schmiede.ghx](rhino/robotic_setup_alte_schmiede.ghx) abd loaded the correct robot model, you can start the Docker ROS moveit simulation environment and connect your ROS client to it.
* For starting the __Docker ROS moveit simulation environment__, go to VS code and start the docker containers by:
  * __Only once__: If you do this the first time, you have to build the local [Dockerfile](docker\docker-images\Dockerfile) via 
    * right-click and `Build` or 
    * in the Terminal (cd to folder) via<br/> `docker build --rm -f Dockerfile -t augmentedfabricationlab/ros-abb-planner-alte-schmiede .` 
    * (Building without cache:<br/> `docker build --no-cache --rm -f Dockerfile -t augmentedfabricationlab/ros-abb-planner-alte-schmiede .`)
    * Tag your Docker Image as: augmentedfabricationlab/ros-abb-planner-alte-schmiede:latest
  * __Always__: Run the docker image that matches your robot model, e.g. [`abb_linear_axis_floor/docker-compose.yml`](docker/ros-systems/abb_linear_axis_floor/docker-compose.yml) via 
    * right-click on the file `Compose-up` or 
    * type `docker-compose up -d` in the Terminal (cd to folder) to start it.
    * when ending, don't forget to stop the image via `docker-compose down -d`
* For access to the __web UI of the docker image__, start your browser and go to:<br/>
`http://localhost:8080/vnc.html?resize=scale&autoconnect=true`
* In Rhino, you can now __connect the ROS client to the rosbridge__ and query all moveit related services (compute_ik, trajectory planning, etc.)


## Docker Setup Information
* Docker user name: augmentedfabricationlab
* The [robotic_setups_description](https://github.com/augmentedfabricationlab/robotic_setups_description.git) repository contains the robot descriptions files for the Dockerfile building.
* The [robotic_setups](https://github.com/augmentedfabricationlab/robotic_setups.git) repository contains the catkin workspace for the urdf models and moveit packages for various robotic setups of our lab, for setting up the systems in Linux as described in [this tutorial](https://gramaziokohler.github.io/compas_fab/latest/examples/03_backends_ros/07_ros_create_urdf_ur5_with_measurement_tool.html).
* The `ros-base` and `novnc` images are remote images and drawn from the [gramaziokohler docker hub organization](https://hub.docker.com/u/gramaziokohler).

## Moveit on Linux Notes

when ROS should be connected between 2 machines via Ethernet (and not via localhost), the ROS MASTER and IPP should be changed via

    nano ~/.bashrc
    
and set accordingly:

    export ROS_MASTER_URI=http://10.10.0.1:11311
    export ROS_IP=10.10.0.1
    
 after pulling a new urdf description into robotic_setups repository run
    
    catkin_make
    source devel/setup.bash
 
 then you can configure a new moveit package via
 
    roslaunch moveit_setup_assistant setup_assistant.launch
    
 Careful: Moveit Collision Matrix for ABB lacks closeness of link 4 and 6, therefore these lines have to be added manually to the srdf file:
 
     <disable_collisions link1="robotA_link_4" link2="robotA_link_6" reason="User"/>
     <disable_collisions link1="robotB_link_4" link2="robotB_link_6" reason="User"/>
 
 
 Then run again `catkin_make` and `source devel/setup.bash` and run your moveit demo pacakge:
 
    roslaunch your_package_name_moveit_config  demo.launch rviz_tutorial:=true
    
 And push the files to the remote repository.

## Docker Troubleshooting

Sometimes things don't go as expected. Here are some of answers to the most common issues you might bump into:

> Q: Docker does not start. It complains virtualization not enabled in BIOS.

This is vendor specific, depending on the manufacturer of your computer, there are different ways to fix this, but usually, pressing a key (usually `F2` for Lenovo) before Windows even start will take you to the BIOS of your machine. In there, you will find a `Virtualization` tab where this feature can be enabled.

> Q: Cannot start containers, nor do anything with Docker. Error message indicates docker daemon not accessible or no response.

Make sure docker is running. Especially after a fresh install, docker does not start immediately. Go to the start menu, and start `Docker for Windows`.


