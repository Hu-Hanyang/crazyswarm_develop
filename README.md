# Crazyswarm_Development Logs
This repository is based on the [crazyswarm](https://github.com/USC-ACTLab/crazyswarm/tree/master) and only focuses on teh Crazyswarm1.

## Implementation

<details>
<summary>cfclient Installation Guide for Ubuntu 20.04 with ROS Noetic</summary>

### Prerequisites
This project requires Python 3.8 - 3.12.

### Step 1: Install Dependencies
Open a terminal and run the following commands to install the necessary system dependencies:

```bash
  sudo apt update
  sudo apt install -y git libxcb-xinerama0 libxcb-cursor0
```
### Step 2: Install Anaconda
If you haven't installed Anaconda yet, download and install it from [Anaconda's official website](https://www.anaconda.com/products/individual).

### Step 3: Create and Activate a Virtual Environment
Create a new virtual environment with Python 3.8 (or your desired version):

```bash
  conda create -n cfclient-env python=3.8
  conda activate cfclient-env
```

### Step 4: Install ROS Noetic Dependencies
Source your ROS Noetic setup script:

```bash
  source /opt/ros/noetic/setup.bash
```

### Step 5: Clone the Repository
Clone the `crazyflie-clients-python` repository:

```bash
  git clone https://github.com/bitcraze/crazyflie-clients-python
  cd crazyflie-clients-python
```
### Step 6: Install the Client
Install the client in editable mode:

```bash
  pip install -e .
```
For development mode, use:
```bash
  pip install -e .[dev]
```
### Step 7: Set up udev Permissions
Set up udev permissions for Crazyradio. Follow the instructions in the [cflib installation guide](https://www.bitcraze.io/documentation/repository/crazyflie-lib-python/master/installation/).

Note: *For detailed solutions to issues where cfclient cannot be opened - USB permissions issue (unable to use Crazyradio):
Follow the instructions in the [USB permissions | Bitcraze](https://www.bitcraze.io/documentation/repository/crazyflie-lib-python/master/installation/usb_permissions/
).
### Step 8: Running the Client
You can run the client with the following commands:

```bash
  cfclient
```
or with:
```bash
  python -m cfclient.gui
```

### Addition 

In the later development, we found that there is conflict between conda and ros, so:

* For later operation, do not use use conda.

* For cfclient, you can both use or not use conda.

</details>

<details>
<summary>Crazyswarm Installation</summary>
Please follow the [instructions](https://crazyswarm.readthedocs.io/en/latest/).
  
From Changelog section to Overview section.

![image](https://github.com/user-attachments/assets/57c64fc7-6a31-4358-970e-4359a1fde645)

</details>

<details>
<summary>Running a Real World Test Script on Crazyflie 2.1 Using Crazyswarm with Vicon system</summary>
  
#### Prerequisites
- Crazyflie 2.1
- Crazyswarm
- Vicon Tracker 3.10
- Single Marker
- There are conflicts between ros and conda environment, do not use conda.


#### Steps

1. **Connect both laptop and lab computer to same WIFI:**

2. **Add Crazyflie object to Vicon tracking system:**

   Attach the single marker to the appropriate position on the Crazyflie 2.1.
   
   Open Vicon System.
   
   Open Vicon Tracker 3.10 in computer.
   
   You can check that single marker as a point in the vicon system.

2. **Follow Crazyswarm instructions on [Configuration](https://crazyswarm.readthedocs.io/en/latest/configuration.html):**

   Here we choose Vicon as tracking system and Single Marker as object tracking mode.

   For Enumerate Crazyflies section, modify crazyflies.yaml with following parameters:
   
   ```
   # ros_ws/src/crazyswarm/launch/crazyflies.yaml
   crazyflies:
   - id: 1
    channel: 100
    initialPosition: [-2.26499, -1.61604, 0.032]
    type: CF21SingleMarker
   ```

   Do not do the Update firmware and Manage fleet with the Chooser sections.

4. **Turn on Crazyflie 2.1, ready for connection:**

5. **Run Hovering (hello, world):**

   5.1 Run the test script in simulation mode to make sure your Python interpreter is set up correctly:
   ```python
    python hello_world.py --sim
   ```
   In the 3D visualization, you should see a Crazyflie take off, hover for a few seconds, and then land.

   5.2 Start Ros:

   Open a new terminal and run
   
   ```python
    roscore
   ```

   5.3 Start the crazyswarm_server:

   Go to /crazyswarm/ 
   
   ```python
    source ros_ws/devel/setup.bash
    roslaunch crazyswarm hover_swarm.launch
   ```

   You should see the Crazyflie 2.1 connect with laptop, there should be one light on Crazyflie 2.1 shows green and the radio device attached to the laptop will also show green light.

   A software rviz in laptop will launch and show the estimated pose of cf1 ( make sure to see estimated pose of cf1 to move to next step).

   4.4 Run hello_world.py script in real world:

   Open a new terminal, go to /crazyswarm/ros_ws and run
  
   ```python
    source devel/setup.bash
   ```

   Then go to /crazyswarm/ros_ws/src/crazyswarm/scripts and run
   
   ```python
    python hello_world.py
   ```
  
   ** Note:

   1. Before running hello_world.py scripts, we need to deactivate Conda or any other environments manager.
   2. Depending on library updating, some syntax of code needs to be changed.
  
   
</details>

<details>
<summary>Crazyflie Custom Neural Network Controller Integration (Write, build, and Flash)</summary>
  
#### Prerequisites

- Crazyflie 2.1
- A pre-trained neural network model saved as `model_latest.pt` which contains all the necessary neural network data in a dict format.
- ST Link V2 Debugger

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

2. **In `model_parameters.c`, extracting the weights and biases for the policy neural network layers: (Not finished)**  

    ```c
    // Example of the generated C arrays for the policy layer
    static const float agent_ac_actor_pi_net_fcs_0_weight[64][17] = {...}
    static const float agent_ac_actor_pi_net_fcs_0_bias[64] = {...}

    // other layers
    ```

3. **Define customized controller into crazyflie firmware:**

   add you own controller, located at src/modules/src/controller/controller.c:
   
    ```c
    static ControllerFcns controllerFunctions[] = {
    {.init = 0, .test = 0, .update = 0, .name = "None"}, // Any
    {.init = controllerPidInit, .test = controllerPidTest, .update = controllerPid, .name = "PID"},
    {.init = controllerMellingerFirmwareInit, .test = controllerMellingerFirmwareTest, .update = controllerMellingerFirmware, .name = "Mellinger"},
    {.init = controllerINDIInit, .test = controllerINDITest, .update = controllerINDI, .name = "INDI"},
    {.init = controllerBrescianiniInit, .test = controllerBrescianiniTest, .update = controllerBrescianini, .name = "Brescianini"},
    ## custom nn contorller
    {.init = controllerNNInit, .test = controllerNNTest, .update = controllerNN, .name="NN"}
    #ifdef CONFIG_CONTROLLER_OOT
    {.init = controllerOutOfTreeInit, .test = controllerOutOfTreeTest, .update = controllerOutOfTree, .name = "OutOfTree"},
    #endif
    };
    ```

4. **Build crazyflie firmware and Flash into crazyflie with crazyradio:**
   
    Ⅰ. Turn off crazyflie
   
    Ⅱ. Start the Crazyflie in bootloader mode by pressing the power button for 3 seconds, both the blue light will flash.
   
    Ⅲ. Open a terminal to build and flash:
   
    ```bash
    cd crazyflie-firmware
    # build
    make -j8
    # flash
    make cload
    ```
  
    After flashing, restart crazyflie, connect to cfclient. 

    Open 'parameters' and 'console' tab, select 'stabilizer', select sub-parameter 'controller'.

    Set 'current value' in to 5 or higher(where the order you put your controller on menu). 

    Cfclient console will print out "CONTROLLER: Using $(name) controller.", switch controller successfully.

5. **Debugging**

    Following the debugging part of the tutorial(Linux): [Openocd_gdb_debugging](https://www.bitcraze.io/documentation/repository/crazyflie-firmware/master/development/openocd_gdb_debugging/)


6. **testing:**
   
   To change the contorller, modify ros_ws/src/crazyswarm/launch/hover_swarm.launch in Crazyswarm.

    Find below code, and change controller number to you customized controller number:

    ```c
    stabilizer:
    estimator: 2 # 1: complementary, 2: kalman
    controller: 2 # 1: PID, 2: mellinger
    ```

    Then rerun "Running a Real World Test Script on Crazyflie 2.1 Using Crazyswarm with Vicon system" (Previous section) for testing fly.
   
#### Todo:

1. ~~Build customized controller in crazyflie-firmware~~
1. ~~Method to set as default controller~~
2. ~~Method to build firmware~~
3. ~~Method to flash firmware~~
4. ~~Verification and testing~~


#### Can use for reference:

[GitHub - mahaitongdae/crazyflie-firmware at dev-nn](https://github.com/mahaitongdae/crazyflie-firmware/tree/dev-nn)

[Build and flash from Bitcraze](https://www.bitcraze.io/documentation/repository/crazyflie-firmware/master/building-and-flashing/build/)

[Flash in crazyswarm from CSDN](https://blog.csdn.net/zeye5731/article/details/109293157)

[Bicraze official add controller and estimator](https://www.bitcraze.io/2023/02/adding-an-estimator-or-controller/)

</details>

