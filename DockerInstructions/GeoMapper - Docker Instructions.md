# GeoMapper - Docker Instructions

- [[#Introduction|Introduction]]
- [[#General points|General points]]
- [[#Installing the docker stack|Installing the docker stack]]
- [[#Setting up the system|Setting up the system]]
- [[#Setting up Portainer|Setting up Portainer]]
- [[#Triggering stack - CLI|Triggering stack - CLI]]
- [[#Triggering Stack - Portainer|Triggering Stack - Portainer]]
- [[#Data output|Data output]]
- [[#Live Visualisation|Live Visualisation]]
- [[#LiDAR temperature monitoring|LiDAR temperature monitoring]]
- [[#Mapping health and pose covariance|Mapping health and pose covariance]]


## Introduction

This markdown file provides information regarding the setup and usage of the GeoMapper OEM docker image.

## General points

- IP address:
  - The IP address of your systems Ethernet end point MUST be `192.168.5.150`
  - Should you require a different IP address, please contact Osprey Systems Engineering Ltd for a custom installation.

- Architecture:
  - The user will be provided with the docker image compiled for the target architecture, ARM64 or AMD64.
  - The target architecture must be communicated to Osprey Systems Engineering at PO.

## Installing the docker stack

Docker must be installed on the target machine, the installation of which is beyond the scope of this document. However, straightforward and comprehensive instructions can be found here:

https://docs.docker.com/engine/install/

The docker image will be provided as a `.tar` file, for example:

```text
geo_mapper_r1-4_sn-2-147-194_x64.tar
```

Transfer this file to the target computer. Then open a terminal session and load the image with the following commands:

```bash
sudo chmod +777 geo_mapper_r1-4_sn-2-147-194_x64.tar
sudo docker image load -i geo_mapper_r1-4_sn-2-147-194_x64.tar
```

Once this has been completed, the `.tar` file can be removed to save disk space.

## Setting up the system

A `docker-compose.yml` file will be provided along with the tar file, which is used to bring up the GeoMapper docker container.

A data folder is specified in the `docker-compose.yml` file, which will create a docker volume for output data storage. This is specified at the bottom of the `docker-compose.yml` file. Should you wish to change the path, this is where it should be changed.

```yaml
volumes:
  - "/dev:/dev"
  - "/home/cm5/GeoMapper/data:/opt/lidar_ws/src/FL/PCD"
```

Adjust the `/home/cm5/GeoMapper/data:` portion to the desired path.

## Setting up Portainer

Installation of Portainer is beyond the scope of this document, however the online instructions are both straightforward and comprehensive:

https://docs.portainer.io/start/install-ce/server/docker

Once the docker image is uploaded to the system, as described earlier, the user can set up a container through the Portainer interface. This will give a GUI which can be used to start, stop and inspect the GeoMapper logs.

To set up Portainer for this, open the Portainer web browser at:

```text
https://localhost:9443
```

A local account will need to be set up the first time Portainer is used.

Once set up, open Portainer and navigate to:

```text
Local → Stacks
```

![Portainer Stacks](images/1_Portainer_Stacks.png)

![Portainer Stacks|207](images/2_Portainer_stacks.png)

Click on the `+ Add stack` icon on the top right.

![New Stack](images/3NewStack.png)

Give the stack an appropriate name, for example:

```text
geomapper_portainer
```

![Stack Name](images/4StackName.png)

Copy the contents of the `docker-compose.yml` file into the text box.

![Paste Compose File](images/5Paste.png)

De-select Access Control and click the `Deploy the stack` button.

![Deploy Stack|292](images/6Deploy.png)

The GeoMapper container will start when the stack is first deployed. See the section titled `Triggering stack - Portainer` for instructions to bring it down.

## Triggering stack - CLI

To trigger GeoMapper through the CLI, run the following command from the path of the `docker-compose.yml` file:

```bash
sudo docker compose up
```

Should the user wish to run this command detached, run the following:

```bash
sudo docker compose up -d
```

When a survey has been completed, and the user wishes to bring GeoMapper down and save the output data, either press `Ctrl+C` in the terminal session if not running in detached mode, or run the following command in the location of the `docker-compose.yml` file:

```bash
sudo docker compose down
```

## Triggering Stack - Portainer

In order to start and stop the GeoMapper docker container using Portainer, log into Portainer and navigate to `Stacks`.

![Portainer Stacks|211](images/2_Portainer_stacks.png)

Here, the stack that was created previously will be visible.

![Stacks View](images/1Stacks.png)

Double click on the appropriate stack to move to container view. Select the container and click the `Start` button.

GeoMapper is now active. Ensure the device is steady for at least 3 seconds to establish good IMU offset calibration and a good initial starting point cloud, then the user is free to move around the environment.

![Stacks Container View](images/3Stacks.png)

When finished, select the container again and click the stop icon.

## Data output

When the GeoMapper docker container is brought down, it will save the output point cloud to the volume specified in the `docker-compose.yml` file.

The file will be titled:

```text
scans.pcd
```

This file will overwrite on subsequent scans.

## Live Visualisation

Once a mapping session is running, the live output can be visualised through Foxglove Studio.

GeoMapper utilises ROS for the middleware, so it is also possible to visualise this data using RViz, however this is outside the scope of this document.

Foxglove Studio is a robotics and data visualisation tool compatible with Linux, macOS and Windows. It can be downloaded from:

https://foxglove.dev/download

Once installed, and a mapping session is active, open Foxglove Studio.

If this is the first time the application has been opened, a free account will need to be created. It is possible to set up foxglove without an account on the device, this can be done following the instructions here: 

https://docs.foxglove.dev/docs/standalone-license

Layouts for Foxglove can also be created and transferred to the device in question, instructions provided here:

https://docs.foxglove.dev/docs/visualization/layouts

Once set up:
1. Open the application
2. Click `Open Connection` in the top left
3. Select `ROS1`
4. Edit the localhost text to the LAN address used by the GeoMapper LiDAR host

```text
http://192.168.5.150:11311
```

![Foxglove Connection](images/1Foxglove.png)

![Foxglove Visualisation](images/2Foxglove.png)

Once connected, click the 3D visualisation icon on the top left / middle of the screen. This will open a panel for 3D visualisation and should enable the correct TF frames.

In the drop down menus on the left hand side:
1. Click the eye icon of `cloud_registered`
2. Click the text `cloud_registered`
3. Enter an appropriate decay time

`100 seconds` is recommended, but this number is theoretically unlimited.

It should be noted that visualising many millions of points in real time with larger decay times may cause the visualising computer to slow down.

The user can drag, rotate and zoom in and out on the 3D visualisation.

When finished, exit the application as normal. Settings will remain persistent.

## LiDAR temperature monitoring

GeoMapper publishes the internal LiDAR core temperature as a ROS topic while the system is running. This can be used to monitor sensor thermal condition during operation. It should be noted that this is the internal temperature, the temperature of the external heat sink will lag this up until equilibrium. 

The temperature topic is:

```text
/livox/lidar_temperature
```

The topic uses the standard ROS message type:

```text
sensor_msgs/Temperature
```

The temperature value is reported in degrees Celsius.

A typical message will appear similar to:

```yaml
header:
  seq: 123
  stamp:
    secs: 1781793983
    nsecs: 123456789
  frame_id: "livox_frame"
temperature: 42.35
variance: 0.0
```

## Mapping health and pose covariance

GeoMapper publishes odometry information during operation. The odometry message includes a pose covariance matrix, which can be used by downstream systems to estimate localisation uncertainty and provide an indication of mapping health.

The odometry topic is:

```text
/Odometry
```

The message type is:

```text
nav_msgs/Odometry
```

The pose covariance is contained within:

```text
/Odometry.pose.covariance
```

The covariance matrix is published as a 36 element flattened 6x6 matrix using the standard ROS odometry order:

```text
x, y, z, roll, pitch, yaw
```

Conceptually, the full pose covariance matrix is:

```text
P_pose =
[ x/x      x/y      x/z      x/roll      x/pitch      x/yaw
  y/x      y/y      y/z      y/roll      y/pitch      y/yaw
  z/x      z/y      z/z      z/roll      z/pitch      z/yaw
  roll/x   roll/y   roll/z   roll/roll   roll/pitch   roll/yaw
  pitch/x  pitch/y  pitch/z  pitch/roll  pitch/pitch  pitch/yaw
  yaw/x    yaw/y    yaw/z    yaw/roll    yaw/pitch    yaw/yaw ]
```

The top-left 3x3 block represents position covariance:

```text
P_position =
[ Pxx  Pxy  Pxz
  Pyx  Pyy  Pyz
  Pzx  Pzy  Pzz ]
```

The lower-right 3x3 block represents orientation covariance:

```text
P_orientation =
[ Prollroll   Prollpitch   Prollyaw
  Ppitchroll  Ppitchpitch  Ppitchyaw
  Pyawroll    Pyawpitch    Pyawyaw ]
```

The diagonal values represent the variance of each pose component. Larger values indicate greater estimated uncertainty in that component of the pose.

For example:

```text
Pxx:
  Estimated uncertainty variance in x position.

Pyy:
  Estimated uncertainty variance in y position.

Pzz:
  Estimated uncertainty variance in z position.

Prollroll:
  Estimated uncertainty variance in roll.

Ppitchpitch:
  Estimated uncertainty variance in pitch.

Pyawyaw:
  Estimated uncertainty variance in yaw.
```

The off-diagonal values represent coupling between different pose components. For example, `Pxy` indicates that uncertainty in `x` and `y` is coupled. In many simple checks, downstream systems may initially use only the diagonal values, but the full matrix is available for more advanced analysis.

The covariance matrix provides an indication of the uncertainty estimated by the internal localisation filter. In general terms:

```text
Small and stable covariance:
  Localisation is likely to be well constrained.

Increasing covariance:
  Localisation uncertainty is increasing.

Large X/Y/Z covariance:
  Translational position uncertainty is increasing.

Large roll/pitch/yaw covariance:
  Attitude uncertainty is increasing.

Large covariance in one direction:
  The system may be operating in a geometrically degenerate environment.

Large covariance across multiple axes:
  The localisation solution may be degraded or unreliable.
```

For example, long corridors, tunnels, repetitive structures, smooth walls and feature-poor areas can reduce the amount of useful geometric constraint available to GeoMapper. In these conditions, the covariance may increase along the weakly constrained direction.

A corridor is a typical example. The LiDAR may observe the side walls clearly, giving good constraint across the corridor. However, the geometry may be repetitive along the corridor, making the forward direction less well constrained. In this case, the covariance may remain small in the sideways direction while increasing in the forward direction. This indicates directional degradation rather than total localisation failure.

A simplified full pose covariance matrix for this case might appear as:

```text
P_pose =
[ 0.0025   0.0000   0.0000   0.0000   0.0000   0.0000
  0.0000   0.1600   0.0000   0.0000   0.0000   0.0000
  0.0000   0.0000   0.0040   0.0000   0.0000   0.0000
  0.0000   0.0000   0.0000   0.0004   0.0000   0.0000
  0.0000   0.0000   0.0000   0.0000   0.0004   0.0000
  0.0000   0.0000   0.0000   0.0000   0.0000   0.0025 ]
```

In this example:

```text
x variance:
  0.0025 m^2

y variance:
  0.1600 m^2

z variance:
  0.0040 m^2

roll variance:
  0.0004 rad^2

pitch variance:
  0.0004 rad^2

yaw variance:
  0.0025 rad^2
```

Taking the square root of each diagonal value gives the estimated standard deviation:

```text
x uncertainty:
  sqrt(0.0025) = 0.05 m

y uncertainty:
  sqrt(0.1600) = 0.40 m

z uncertainty:
  sqrt(0.0040) = 0.063 m

roll uncertainty:
  sqrt(0.0004) = 0.020 rad

pitch uncertainty:
  sqrt(0.0004) = 0.020 rad

yaw uncertainty:
  sqrt(0.0025) = 0.050 rad
```

This indicates that the system is relatively well constrained in `x`, `z`, `roll` and `pitch`, but is less well constrained in `y` and `yaw`. If `y` corresponds to the forward direction along a corridor, this would be consistent with reduced localisation confidence along the corridor axis.

Useful scalar values can be derived from the covariance matrix, including:

```text
Position covariance trace:
  Pxx + Pyy + Pzz

Orientation covariance trace:
  Prollroll + Ppitchpitch + Pyawyaw

Maximum position variance:
  max(Pxx, Pyy, Pzz)
```

For more advanced monitoring, the eigenvalues of the 3x3 position covariance block can be used. The largest eigenvalue indicates the direction of greatest position uncertainty.

The 3x3 position covariance block is:

```text
P_position =
[ Pxx  Pxy  Pxz
  Pyx  Pyy  Pyz
  Pzx  Pzy  Pzz ]
```

A large principal eigenvalue can indicate that the system is becoming poorly constrained in one direction, even if the overall position still appears visually reasonable.

It should be noted that covariance is an estimator derived uncertainty metric. It is a useful indicator of localisation health, but it should not be treated as an absolute guarantee of mapping accuracy. For critical applications, covariance should be considered alongside other indicators such as visual map quality, odometry continuity, LiDAR visibility, and the presence of repeated scan warnings in the GeoMapper logs.

RViz can be used to visualise odometry and pose covariance using an Odometry display. Foxglove Studio can be used to inspect the odometry message and covariance values, although covariance ellipsoid visualisation support may depend on the specific Foxglove version and layout configuration.