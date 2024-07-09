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
  
From Changelog section to Overview section.
</details>

<details>
<summary>Running a Real World Test Script on Crazyflie 2.1 Using Crazyswarm with Vicon system</summary>
  
Problem: Vicon tracker 3.10 requires at least three markers to create an object. single marker cannot be set (solved).
  
#### Prerequisites
- Crazyflie 2.1
- Crazyswarm
- Vicon Tracker 3.10
- 4 Markers
- There are conflicts between ros and conda environment, do not use conda.


#### Steps

1. **Add Crazyflie object to Vicon tracking system:**

   Attach the four markers to the appropriate position on the Crazyflie 2.1.
   
   Open Vicon System.
   
   Open Vicon Tracker 3.10 in computer.
   
   Select 4 marker points on Crazyflie 2.1 and use these points to creat an object, named cf1.

2. **Follow Crazyswarm instructions on [Configuration](https://crazyswarm.readthedocs.io/en/latest/configuration.html):**

   Here we choose Vicon as tracking system and Unique Marker Arrangements as object tracking mode.

   Do not do the Update firmware and Manage fleet with the Chooser part.

4. **Turn on Crazyflie 2.1 and ready for connection:**

5. **Run Hovering (hello, world):**

   4.1 Run the test script in simulation mode to make sure your Python interpreter is set up correctly:
   ```python
    python hello_world.py --sim
   ```
   In the 3D visualization, you should see a Crazyflie take off, hover for a few seconds, and then land.

   4.2 Start Ros:

   Open a new terminal and run
   
   ```python
    roscore
   ```

   4.3 Start the crazyswarm_server:
   ```python
    source ros_ws/devel/setup.bash
    roslaunch crazyswarm hover_swarm.launch
   ```

   You should see the Crazyflie 2.1 connect with laptop, there should be one light on Crazyflie 2.1 shows green and the radio device attached to the laptop will also show green light.

   A software rviz in laptop will launch and show the estimated pose of cf1.

   4.4 Run hello_world.py script in real world:

   Open a new terminal and run
  
   ```python
    source devel/setup.bash
    python hello_world.py
   ```
   * the hello_world.py is located in ros_ws/src/crazyswarm/script/
  
   ** Note:

   1. Before running hello_world.py scripts, we need to deactivate Conda or any other environments manager.
   2. Depending on library updating, some syntax of code needs to be changed.
  
   
</details>

<details>
<summary>Crazyflie Custom Neural Network Controller Integration (Write and Flash)</summary>
  
#### Prerequisites

- Crazyflie 2.1
- A pre-trained neural network model saved as `model_latest.pt` which contains all the necessary neural network data in a dict format.

#### Steps

1. **Write a script (`convert.py`) to load the neural network model dictionary and convert the parameters into C arrays, then write them into `model_parameters.c` file:**

    ```python
    import torch
    
    def tensor_to_c_array(tensor, name):
        array = tensor.cpu().numpy()  # Ensure the tensor is processed on CPU
        c_array = ""
        if len(array.shape) == 2:
            shape_str = f"[{array.shape[0]}][{array.shape[1]}]"
            c_array = f"static const float {name}{shape_str} = {{\n"
            for row in array:
                c_array += "    {" + ', '.join(map(str, row)) + "},\n"
            c_array += "};\n"
        elif len(array.shape) == 1:
            shape_str = f"[{array.shape[0]}]"
            c_array = f"static const float {name}{shape_str} = {{"
            c_array += ', '.join(map(str, array))
            c_array += "};\n"
        return c_array
    
    def recursive_convert(name, param, c_arrays):
        if isinstance(param, torch.Tensor):  # Check if the parameter is a tensor
            c_name = name.replace('.', '_')
            c_arrays.append(tensor_to_c_array(param, c_name))
        elif isinstance(param, dict):  # If it's a dictionary, process recursively
            for sub_name, sub_param in param.items():
                recursive_convert(f"{name}.{sub_name}", sub_param, c_arrays)
    
    # Load the model's state dictionary
    model_state_dict = torch.load('model_latest.pt', map_location=torch.device('cpu'))
    
    # Convert each parameter to C arrays
    c_arrays = []
    for name, param in model_state_dict.items():
        recursive_convert(name, param, c_arrays)
    
    # Write all C arrays to a file
    with open('model_parameters.c', 'w') as f:
        for c_array in c_arrays:
            if c_array:  # Write to file only if c_array is not empty
                f.write(c_array + '\n')

    ```

2. **In `model_parameters.c`, extracting the weights and biases for the policy neural network layers:**

    ```c
    // Example of the generated C arrays for the policy layer
    static const float agent_ac_actor_pi_net_fcs_0_weight[64][17] = {...}
    static const float agent_ac_actor_pi_net_fcs_0_bias[64] = {...}

    // other layers
    ```
#### Todo:

1. Set as default controller
2. Compile firmware
3. Flash firmware
4. Verification and testing


#### Can use for reference:

[GitHub - mahaitongdae/crazyflie-firmware at dev-nn](https://github.com/mahaitongdae/crazyflie-firmware/tree/dev-nn)

At src/modules/src/controller



</details>

