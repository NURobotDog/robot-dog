# ros-docker
- [Introduction](#introduction)
- [Quick Start](#quick-start)
# Introduction: 
This repo contain a communication framework that will a part of robot dog project in this repo:  
https://github.com/NEURoboticsClub/NU-Dog/tree/main  

Our motor controller (MC) program moteus_test.py is a program that will control the 12 motors that are running the robot's movement. We will be sending ROS commands to MC from our laptop using the script in cpu.py.  
So we need another ROS node inside our Raspberry pi to receive these commands and pass it to MC. However, there is an incompatibility issue with Raspberry Pi used in this project and the ROS version that we want to use which is ROS melodic.  

So the objective is to set up a communication framework so that a ROS node in our machine (cpu.py) can communicate with the motor controller program (moteus_test.py) in Raspberry Pi.  

This framework will solve the incompatibility issue by leveraging Docker containers.  

### How does this work?  
We will run what we will call the “bridge” nodes which are ROS nodes that will be running in the Docker environment with ROS package and other dependencies installed. These nodes do not do anything other than passing messages back and forth between MC and CPU. They are multi-threaded so that they can listen and publish messages either using Python sockets or use ROS built-in publisher-subscriber framework concurrently. 

### What actually happens when we start running these nodes?  

Take a look at the diagram below which illustrates the high level overview of the messages that are passed.  

<img width="1366" alt="Screenshot 2023-07-23 at 4 44 35 PM" src="https://github.com/freecode23/ros-docker/assets/67333705/6915a31d-66f0-4a2f-8ae0-7b906279474f">


1. The `champ_teleop` module enables us to dictate the desired movements of the robotic dog via keyboard input. It accomplishes this by publishing the input data on multiple topics, which include `cmd_vel` and `body_pose`. 

2. Some nodes in `champ_config` are designed to subscribe to these topics. They then publish the necessary pose and velocity details that the twelve Motor Controllers (MCs) need to execute. Before these details are passed onto the MCs, the Central Processing Unit (CPU) node processes this information. 

3. The CPU node communicates the command to the MCs using the ROS publisher framework. The `cpu_sub` node, which is one of the bridge nodes, subscribes to the topic published by the CPU, thereby receiving this message. 

4. The `cpu_sub` node then forwards the message to the MCs via a socket. Upon receiving these byte messages, the MCs convert them into JSON format and set the attributes of the twelve motors. These attributes include the motor ID, velocity, position, and torque. 

5. The MCs call the `getParsedResults()` method, which fetches the real-time attributes of the motors and sends them to the `mc_sub` bridge node via a socket.

6. The `mc_sub` bridge node then publishes messages on the MC topic. Consequently, its subscriber (the CPU node) can listen to these messages, invoke its callback function, and perform necessary actions with this new information about motor attributes. Specifically, in our setup, the CPU node takes this information along with the data received from `champ_config` packages to adjust the new command that it should send to the MCs.

# Quick Start
## Prerequites:  
1. Docker installed in your machine:  
https://docs.docker.com/desktop/install/linux-install/  

2. Make sure Docker compose is already installed on your machine:  
   ```
   docker-compose --version
   ``` 
   - We will be using version 2  
   - If it's not yet installed, go to:  
   https://docs.docker.com/compose/install/linux/#install-using-the-repository  

3. If you need to run Raspberry Pi:  
   - Make sure Python3 is installed in Pi  
   - Docker and docker compose installed  



## EXTRA INFO: 
### 1. To run any ros command in a new terminal
1. Open new terminal in your laptop and go into `cpu-catkins` directory. 
2. Run:  
```
docker exec -it cpu-catkins-cpu-node-1 bash    
source /app/devel/setup.bash
```
3. run any ros command such as:  
```
rostopic echo /joint_states/position  
rostopic info /cmd_vel
```
   
## 2. To stop any running containers:  
```
docker compose down
```  

## 3. What happens when they all start running:    

https://github.com/freecode23/ros-docker/assets/67333705/48e9ce1a-7318-4152-bdc5-fee3295179fc








