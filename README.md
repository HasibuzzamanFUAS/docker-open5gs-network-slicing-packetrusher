[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/ao4Zt6N9)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=17383441&assignment_repo_type=AssignmentRepo)
<h1 align="center">Slicing-UPF-AMF-Traffic</h1>

<!-- PROJECT LOGO -->
<img src="resources/images/FRA-UAS_Logo_rgb.jpg" width="150"/>

<h1 align="center">Team_STAMHS_5G</h1>
<p align="center">
  M.Eng. Information Technology <br>
  WS 24-25 / Prof. Dr. Armin Lehmann <br>
  Date: March 1, 2025
</p>

<div align="center">
 
| Name of the contributors   |  
|-----------------------|  
| Md. Ashraf Uddin      |   
| Tanvir Ahmed          |  
| Saleque Ahmed         |  
| Md. Sohel Rana        |  
| Hasibuzzaman          |   
| Md. Mosharraf Hossain | |  

</div>

## Table of Contents

*   [Introduction](#introduction)
*   [System Architecture](#system-architecture)
*   [Requirements](#requirements)
*   [Installation Process](#installation-process)
     - [Environment Setup](#step-1-environment-setup)
     - [Configure it](#step-2-configure-it)
     - [Build The Docker Images](#step-3-build-the-docker-images)
     - [Run The Containers](#step-4-run-the-containers)
*   [Testing](#testing)
    - [Route adding at upfs](#route-adding-at-upfs)
     - [Listening traffic to receive](#listening-traffic-to-receive)
     - [Generate traffic at UEs](#generate-traffic-at-ues)
     - [Dockerized Deployment Overview](#dockerized-deployment-overview)
     - [UEs Registration](#ue-registration)
     - [IP configurations of UPFs and Packetrusher](#ip-configurations-of-upfs-and-packetrusher)
     - [Listening to receive traffic](#listening-to-receive-traffic)
     - [Traffic Throughput for Each UE](#traffic-throughput-for-each-ue)
     - [Traffic Received at UPF](#traffic-received-at-upf)
     - [UPF Internet Connectivity Test](#upf-internet-connectivity-test)
*   [Limitations](#limitations)
*   [Conclusion](#conclusion)

## Introduction

The goal of this project is to use Open5GS to deploy Network Slicing in order to create a standalone 5G core network, 5G RAN based on Packetrusher, 5G RAN with multiple gNBs each using its own responsible AMF, 5G UE (User Equipment) based on Packetrusher using 

Docker and Docker Compose. And Implementation of a management and orchestration solution for Docker containers and 5G NFs. Also, to implement auto scale up of UPFs when the traffic threshold is  exceeded.

## System Architecture

![image](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/resources/images/System%20Architecture.jpg)

Further deployment description [here](https://github.com/MobileComputingWiSe24-25/mobcom-team_stamhs_5g/blob/main/documentation/opengs-network-slicing.md)

## Requirements

- Operating System: Ubuntu 20.04.06 LTS (Focal Fossa) (Linux recommended)
- gtp5g - 5G compatible GTP kernel module
- Docker & Docker Compose
- Open5GS (5G Core Implementation)
- Packetrusher (5G RAN Simulator)
- Prometheus (Monitoring)
- Grafana (Visualization)
- Traffic Generators (iperf3)

## Installation Process

### Step 1: Environment Setup
- Download and Install Ubuntu 20.04.6 LTS (Focal Fossa) [From here.](https://releases.ubuntu.com/focal/)

> Note: After Ubuntu Installation OS will provide a pop-up message to update. Do not update the version.

- Download and Install gtp5g - 5G compatible GTP kernel module [From here.](https://github.com/free5gc/gtp5g)

- Clone the Open5GS repository from the source link.

```bash
git clone https://github.com/HasibuzzamanFUAS/docker-open5gs-network-slicing-packetrusher.git
```
- Enter inside the cloned local repository
```bash
cd docker-open5gs-network-slicing-packetrusher
```
### Step 2: Configure It 

First, update the `.env` file with the desired values to use.

The `.env` file is used to build the images using Make or Docker Compose, as well as deploying in Docker Compose.

`OPEN5GS_VERSION` is the version of Open5GS to use.
- Accepted values are the tags, branches or commit IDs used in the Open5GS project
- Implemented value: v2.7.2
- Tested values: v2.5.5, v2.5.6, v2.5.8, v2.6.1, v2.6.2, v2.6.3, v2.6.4, v2.6.6, v2.7.0, v2.7.1, v2.7.2

`UBUNTU_VERSION` is the version of the ubuntu Docker image used as base for the containers.
- Accepted values are the tags used by Ubuntu in Docker Hub
- Implemented value: focal
- Tested values: focal, jammy

`MONGODB_VERSION` is the version of the mongo Docker image used as database for Open5GS.
- Accepted values are the tags used by MongoDB in Docker Hub
- Implemented value: The one specified in the `.env` file which is 6.0

`NODE_VERSION` is the version of Node.js being used to build the Open5GS WebUI.
- Accepted values are the tags used by Node in Docker Hub for its bookworm-slim image and the Node.js dependency of Open5GS WebUI
- Default value: 20

`DOCKER_HOST_IP` is the IP address of the host running Docker.

### Step 3: Build The Docker Images

> Tip: This is the recommended way to build the project, you can build the images all at once with a single command taking advantage of docker buildx parallelism.

> Note: This method uses the `docker-bake.hcl` file and requires `docker-buildx-plugin` which is integrated with the Operating System itself.

From the top level directory of the repository run:
```bash
docker buildx bake
```
### Step 4: Run The Containers

Select the appropriate deployment and from the top level directory of the repository run:

Run the network-slicing deployment
```bash
docker compose -f compose-files/network-slicing/docker-compose.yaml --env-file=.env up -d
```
Tear down the network-slicing deployment
```bash
docker compose -f compose-files/network-slicing/docker-compose.yaml --env-file=.env down
```

## Testing

### Route adding at upfs:

```bash
docker exec -it <CONTAINER_NAME> /bin/bash
```

```bash
ip route add <UE1_IP_SUBNET> via <UPF_IP> dev <UPF_Interface>
```

```bash
ip route del <UE2_IP_SUBNET> via <UPF_IP> dev <UPF_Interface>
```

### Listening traffic to receive:

```bash
iperf3 -s -i 1 -p <PORT1> & iperf3 -s -i 1 -p <PORT2> & iperf3 -s -i 1 -p <PORT3> &
```

### Generate traffic at UEs:

```bash
ip vrf exec <VRF_IF> iperf3 -c <IPERF_SERVER> -p <PORT> -t 9000 -b <BW-K/M/G>

# Or, to generate 10Gbps traffic

ip vrf exec <VRF_IF> iperf3 -c <IPERF_SERVER> -p <PORT> -t 9000 -b 10G

# Installing iperf3, (if not found):

apt-get install iperf3 -y
```

## Limitations

- Unable to integrate Multiple UEs under single gNB due to PacketRusher’s limitations.

- Auto scale up of UPFs during traffic overload couldn’t be implemented.

## Conclusion

In this project, we have created an independent 5G Core Network (5GC) with Open5GS that features support for network slicing, multiple User Plane Functions (UPFs), and a containerized architecture. This architecture efficiently routes traffic among UPFs, User Equipments (UEs), and gNodeBs (gNBs) efficiently. We have created an efficient 5G RAN simulation with Packetrusher support featuring multiple gNBs and UEs to support extensive traffic generation and testing. Besides this, iperf3 testing checks that traffic between different UEs is correctly divided between UPFs in accordance with S-NSSAI to ensure proper network slicing.
