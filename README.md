# Crazyswarm_Development Logs
This repository is based on the [crazyswarm](https://github.com/USC-ACTLab/crazyswarm/tree/master) and only focuses on teh Crazyswarm1.

## Implementation

<details>
<summary>cfclient Installation Guide for Ubuntu 20.04 with ROS Noetic using Anaconda</summary>

#### Prerequisites
This project requires Python 3.8 - 3.12.

#### Step 1: Install Dependencies
Open a terminal and run the following commands to install the necessary system dependencies:

```bash
  sudo apt update
  sudo apt install -y git libxcb-xinerama0 libxcb-cursor0
```
#### Step 2: Install Anaconda
If you haven't installed Anaconda yet, download and install it from [Anaconda's official website](https://www.anaconda.com/products/individual).

#### Step 3: Create and Activate a Virtual Environment
Create a new virtual environment with Python 3.8 (or your desired version):

```bash
  conda create -n cfclient-env python=3.8
  conda activate cfclient-env
```

#### Step 4: Install ROS Noetic Dependencies
Source your ROS Noetic setup script:

```bash
  source /opt/ros/noetic/setup.bash
```

#### Step 5: Clone the Repository
Clone the `crazyflie-clients-python` repository:

```bash
  git clone https://github.com/bitcraze/crazyflie-clients-python
  cd crazyflie-clients-python
```
#### Step 6: Install the Client
Install the client in editable mode:

```bash
  pip install -e .
```
For development mode, use:
```bash
  pip install -e .[dev]
```
#### Step 7: Set up udev Permissions
Set up udev permissions for Crazyradio. Follow the instructions in the [cflib installation guide](https://www.bitcraze.io/documentation/repository/crazyflie-lib-python/master/installation/).

Note: *For detailed solutions to issues where cfclient cannot be opened - USB permissions issue (unable to use Crazyradio):
Follow the instructions in the [USB permissions | Bitcraze](https://www.bitcraze.io/documentation/repository/crazyflie-lib-python/master/installation/usb_permissions/
).
#### Step 8: Running the Client
You can run the client with the following commands:

```bash
  cfclient
  cfheadless
  cfloader
  cfzmq
```
or with:
```bash
  python -m cfclient.gui
```
Make sure your virtual environment is activated each time you use cfclient.

</details>

<details>
<summary>Crazyswarm Installation</summary>
Please follow the [instructions](https://crazyswarm.readthedocs.io/en/latest/).
</details>

<details>
<summary>Crazyflie Neural Network Controller Integration</summary>

</details>

