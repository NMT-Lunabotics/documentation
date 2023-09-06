# Lunabotics Technology Reference

This document is meant to be a rough summary of the technology used in
each year's competition run.

## 2022: DAVID

For the 2022 competition (NMT's first year at Lunabotics), the first
semester (i.e., Fall 2021) was primarily devoted to simulation of the
robot's interactions with its environment, because the CS team didn't
have access to a physical robot. After some consideration between
CoppeliaSim (https://www.coppeliarobotics.com/) and Webots
(https://www.cyberbotics.com/), we chose to use Webots because it had
marginally better documentation. At the time we intended to write the
robot software like a traditional software stack, implementing our
own networking and potentially outsourcing some mapping and path
planning work to libraries.

In the second semester, we decided to use ROS (http://wiki.ros.org/),
the Robot Operating System, which is not an operating system but
instead a communications protocol with a large surrounding suite of
tools and libraries. In general, ROS is characterized by the following
features:

 - ROS's tools are managed as an APT repository, and must be installed
   on an Ubuntu machine; and ROS versions are tied to specific Ubuntu
   versions, so running, e.g., ROS Melodic requires running Ubuntu
   18.04. (This fact has always caused major problems for the team,
   and resulted in the CS team thinning down to 3 members by the end
   of the year because not many people wanted to install a full,
   slightly outdated operating system on their computer just for the
   purposes of this project.)

 - Communication happens in two ways: using "topics", and "services".
   Only programs that register themselves as "nodes" can communicate
   on the ROS network.

   Topics are channels that nodes can "publish" or "subscribe" to; any
   message that is published to a topic is automatically broadcasted
   over the network (or locally using inter-process communication) to
   the complete list of every node that is subscribing to the same
   topic.

   Whereas topics follow a connection-oriented multiple-producer
   multiple-consumer model, services follow a connectionless
   multiple-producer single-consumer model. Any node can "advertise" a
   service, and other nodes can then "call" that service, which
   establishes a connection to the advertising node, sends a single
   data structure, and then disconnects.

   Topics and services are both located inside the ROS namespace,
   where each topic and service name begins with a slash. For example,
   `/cmd_vel` is the typical name for a topic that publishes motion
   commands, and which a motor controller will typically subscribe to.
   Specific hardware should generally place all its services and
   topics inside a single sub-namespace: for example, if a camera
   called "front" provides an image and a depth map, it should
   generally publish `/camera_front/image` and `/camera_front/depth`.

   Both features are strongly typed: ROS nodes define "service" and
   "topic" files, and these are processed by catkin (the ROS build
   tool) into data structure declarations in C and C++, Lisp data
   structures, JavaScript classes, et cetera.

 - Communication is mediated by a machine called the "ROS Master",
   which is a machine that is running the `roscore` program. The ROS
   master maintains a database of which nodes are publishing and
   subscribing to which topics, and which nodes are advertising which
   services; it is effectively a simple lookup database to determine
   which port ROS nodes should listen and connect to one another on.

   The ROS master _does not_ forward traffic under any circumstances,
   an it is not sufficient for nodes to be able to reach the ROS
   master for them to be able to communicate amongst themselves
   correctly; each node on a ROS network must be able to reach every
   other node directly for routing to work correctly. This can
   occasionally be a problem for robots with multiple computers
   onboard but only one antenna.

We used ROS because it provided automatic coding and routing of
network traffic, and also good visualization and processing tools
without a large effort investment into that kind of basic tooling. Our
setup ended up looking like this:

 - We used `rviz` (https://wiki.ros.org/rviz) to view the robot's
   tracking information and camera feeds for manual piloting.

 - `realsense2_camera` (https://wiki.ros.org/realsense2_camera)
   provided automatic ROS support for our Intel Realsense cameras.

 - `hector_mapping` (https://wiki.ros.org/hector_mapping) generated
   maps from the camera data on the robot.

 - `robot_state_publisher`
   (https://wiki.ros.org/robot_state_publisher) and `tf2` provided
   transformation matrices between the world and the robot's
   individual joints.

 - `move_base`, with a plug-in of `teb_local_planner`, provided actual
   navigation commands from the map. `dwa_local_planner` was also
   considered, but was apparently abandoned.

## 2023: DAVIID

