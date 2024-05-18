# CyberDog Decision Module

---
## **[Introduction]**
---
This project contains two sub-projects: `decision_maker` and `decision_utils`. The latter is a collection of base classes and tools for decision-related functionality, responsible for realizing general-purpose functionality; the former is oriented towards the business layer, responsible for realizing business-specific functionality.

As of today:

`cascade_manager` is provided in `decision_utils`, which is inherited from `cyberdog_utils::LifecycleNode`, and has cascade/parallel and single-point control functionality, which allows for quick control of starting and shutting down nodes under its scope.

`automation_manager`, `ception_manager`, `interaction_manager` and `motion_manager` are provided in `decision_maker`. They are used for the management and decision making of automation, perception, human-computer interaction and motion functions, respectively. All four modules inherit from `cascade_manager` and are slightly modified based on business functions.

## **[Functional Features]**
---
### **cascade manager**

#### **Introduction**

This feature is implemented in [decision_utils](decision_utils/src/cascade_manager.cpp). The purpose is to implement a function that can manage the state of its child nodes in parallel by self-starting and self-pausing. It inherits from `cyberdog_utils::LifecycleNode`, the principle of which can be jumped to [cyberdog_utils] (... /cyberdog_common/cyberdog_utils) for an overview.

The cascade manager implements `manager_configure`, `manager_activate`, `manager_deactivate`, `manager_cleanup`, `manager_shutdown`, and ` manager_error`, and requires nodes inheriting it to call the appropriate functions in the appropriate transition state. Parameter configuration is supported.

#### **Parameter Configuration**

The name of the variable is inside [].

- Cascading node list name [node_list_name_], parameter name determined from input string.
- Manager timeout [timeout_manager_] with parameter name `timeout_manager_s`.

#### **Basic mechanism**

The cascade manager distinguishes its own function type by recognizing external parameters. There are currently three function types implemented, `single manager mode`, `single list manager mode` and `multiple list manager mode`.

- Single Manager Mode: There are no child nodes to connect to, and all operations are performed with only the node in mind.
- Single list management mode: When nodes are configured, the external parameter (YAML) list will be read through the cascade node list name, and according to the list, the corresponding node name will be added as child node in order, and state synchronization will be carried out. And when activating and suspending operations, it will check whether each node in the list reaches the corresponding state, and update the `chainnodes_state_` variable in time.
- Multiple list management mode: similar to single list, the node configuration will be read from the external parameter list, only that it can be read by inputting different parameter lists during configuration. To use this management mode, you need to copy the cascade node list name to `multi` when instantiating to activate the multiple list management mode and pass the correct string of the list name to the `manager_configure` function when configuring. Pass the correct string for the list name.

All nodes inheriting the cascade manager can use `message_info`, `message_warn`, and `message_error` for ROS-related logging output, where input in `std::string` format is sufficient (multiple strings can be combined with `+`).

### **Automation management**

#### **Introduction**

Currently this node is only responsible for activating and suspending automation-related nodes, including those specifically needed in `Explor` mode and `Track` mode. Parameter configuration is supported.

#### **Parameter Configuration**

- Explore map building mode node list [explore_map_node_list_name_], parameter name determined from input string.
- Explore navigation mode node list [explore_nav_node_list_name_], parameter name determined from input string.
- Track mode node list [track_node_list_name_], parameter name determined from input string.

### **Perception management**

#### **Introduction**

Currently this node is responsible for activating and suspending perception related functional nodes and is responsible for collecting important perceptual information, fusing perceptual information and processing and categorizing the information.

#### **Parameter Configuration**

- Tick frequency [rate_tik_], parameter name is `rate_tik_hz`.
- Low battery threshold [soc_limit_], parameter name is `soc_limit_perc`.

#### Security

Currently the security protection feature detects the robot state at the frequency of rate_tik_, and currently only supports the identification of low battery state and broadcast notification to external nodes. New features will be added later.

### **Human-robot interaction management**

#### **Introduction**

This node is currently responsible for activating and suspending HCI related functional nodes.

### **Motion management**

#### **Introduction**

Currently this node is responsible for all motion-related command collection, command processing and command placement, all motion-related state collection, state processing and state broadcasting, and all motion-related parameter configuration and function configuration.

- Commands include: mode switching, gait switching, movement execution and action commands.
- States include: current mode, gait, cached gait, action, speed, position and parameters.
- Parameters include: gait parameters and intra-node parameters.

Since the current motion control level uses `Lightweight Communications and Marshalling (LCM)` for communication, this module will also call the LCM related interface for communication.

Customized motions will be designed through `Tom's Obvious, Minimal Language (TOML)`, which is currently designed by the motion control developers through the framework of `Mini Cheetah` improved and provided in `txt` format, and we designed a `Python` script to automatically convert the `txt` into ` TOML`. * In the future, functionality will be developed to automatically generate `TOML` files for actions, so that developers don't need to write them manually. *

Since the robot has only one body, the following `mode switching`, `gait switching` and `action execution` all have preemption functions.

#### **Node parameter configuration**
##### Frequency class
- Common frequency[rate_common_]
- Control Frequency [rate_control_]
- Output frequency[rate_output_]
- Waiting frequency [rate_wait_loop_]
- odometer frequency [rate_odom_]
- LCM broadcast frequency constant [rate_lcm_const_]
##### Timeout Class
- Manager timeout [timeout_manager_]
- Motion timeout [timeout_motion_]
- Gait Timeout [timeout_gait_]
- Motion timeout [timeout_order_]
- LCM receive timeout [timeout_lcm_]
##### Assignment Class
- X-axis line speed extreme value [cons_abs_lin_x_]
- Y-axis linear velocity poles [cons_abs_lin_y_]
- X-axis angular velocity poles [cons_abs_ang_r_].
- Y-axis angular velocity extremum [cons_abs_ang_p_]
- Z-axis angular velocity extremum [cons_abs_ang_y_]
- Z-axis angular acceleration extremes [cons_abs_aang_y_]
- Maximum value of body height [cons_max_body_]
- Maximum value for plantar height [cons_max_gait_]
- Body Height Default [cons_default_body_]
- Default plantar height [cons_default_gait_]
- Default linear speed of movement [cons_speed_l_normal_]
- Default angular speed of movement [cons_speed_a_normal_]
##### Scale Classes
- Low battery control scale [scale_low_btr_]
##### Communication Class
- Motion up return channel [ttl_recv_from_motion_]
- Motion send channel [ttl_send_to_motion_]
- Motion send channel [ttl_recv_from_motion_] Motion send channel [ttl_send_to_motion_] Motion return channel [ttl_from_odom_
- Motion return port [port_recv_from_motion_]
- Motion send port [port_send_to_motion_]
- odometer up return port [port_from_odom_]

#### **Motion parameter configuration**

Data format: refer to [motion_msgs] (... /cyberdog_interfaces/motion_msgs/README.md)

Motion parameter modification currently only supports dynamic modification of body height and plantar height.

Legitimate request: motion parameter data with a more recent timestamp.

- Newer timestamp means that the timestamp of this parameter modification is newer than the timestamp of the last parameter modification, usually the current system time in the same time zone is sufficient.
- The parameter range is within the constraints of `cons_max_body_` and `cons_max_gait_` above and is positive.



#### **Mode switching**

Data format: refer to [motion_msgs](... /cyberdog_interfaces/motion_msgs/README.md)

Legitimate request: a pattern with a newer timestamp.
- A newer timestamp means that the timestamp of this switch request is newer than the timestamp of the last mode switch, and it is usually sufficient to take the current system time in the same time zone.
- The mode needs to be guaranteed to match the preset options. Mode switching supports default mode (MODE_DEFAULT), lock mode (MODE_LOCK), manual mode (MODE_MANUAL), explorer mode (MODE_EXPLOR), and track mode (MODE_TRACK). The modes are distinguished by two fields, `control_mode` and `mode_type`, the first three modes have `mode_type` of `DEFAULT_TYPE(0)`, and the last two modes can be recognized as sub-modes based on `mode_type`.

Preemption function: any newly initiated legitimate mode switch request can preempt the running old mode switch request, i.e., the new one has high priority. Identical modes (both fields are identical) are not preempted.

Cancel function: the handle that initiated the request can initiate a cancel at any moment.

Feedback: once a legitimate request is received, the Mode Action service detects the current switching state at a frequency of `rate_common_`Hz and returns the current switching situation at that frequency.

Result:
- The switchover succeeds, updating the `modestamped` value in `robot_control_state_`.
- If the switching fails, the previous state is restored. If the switch has switched gaits, the previous gait is resumed (except when preempted); if another node has been activated, the other node is suspended.

#### **Gait switching**

Data format: refer to [motion_msgs](... /cyberdog_interfaces/motion_msgs/README.md)

Legitimate request: a gait with a legitimate motive with a more recent timestamp
- Legitimate motivation means that the `motivation` value must match the preset value `cyberdog_utils::GaitChangePriority`.
- A newer timestamp means that the timestamp of this switch request is newer than the timestamp of the last gait switch, and it is generally sufficient to take the current system time in the same time zone.
- The gait needs to be guaranteed to match the preset options. The gait currently supports power-off ambush (GAIT_PASSIVE), slow get down (GAIT_KNEEL), resume stand (GAIT_STAND_R), stance stand (GAIT_STAND_B), slow walk (GAIT_WALK), low speed trot (GAIT_SLOW_TROT ), medium trot (GAIT_TROT), fast trot (GAIT_FLYTROT), two-legged bounce (GAIT_BOUND), and four-legged bounce (GAIT_PRONK).

Preemption function: the motivation (priority) is considered first, followed by the gait.
- Depending on the value of the motivation (uint8), from big to small, the priority goes from high to low.
- Same priority if the gait is different, the new request preempts the old one.

Cancel function: the handle that initiated the request can initiate a cancel at any moment.

Interrupt Function: If a mode change occurs during a gait switch, the current gait switch is immediately aborted.

Feedback: once a legitimate request is received, the Gait Action service detects the currently switched gait at a frequency of `rate_common_`Hz and returns the current switching situation at that frequency.

Result:
- Switching success: update the `gaitstamped` value in `robot_control_state_`.
- Switching failure: restore the preset value of `gait_cached_`.

#### **action execution**

Data format: refer to [motion_msgs](... /cyberdog_interfaces/motion_msgs/README.md)

Legitimate requests: motions with newer timestamps and their parameters (if required)
- A newer timestamp means that the timestamp of this execution request is newer than the timestamp of the last action execution, usually the current system time in the same time zone is sufficient.
- The action needs to match the preset options. The action currently supports stand up (MONO_ORDER_STAND_UP), down (MONO_ORDER_PROSTRATE), back (MONO_ORDER_STEP_BACK), turn in place (MONO_ORDER_TURN_AROUND), handshake ( MONO_ORDER_HI_FIVE), DANCE (MONO_ORDER_DANCE), WELCOME (MONO_ORDER_WELCOME), TURN (MONO_ORDER_TURN_OVER) and SIT (MONO_ORDER_SIT).
- Of these, backward and turn in place are parameterizable, i.e. the values given in the `para` field are valid. The effect of the action execution will be directly related to the parameters.

Preemption function: except for the two actions of Worship and Roll, all other actions can be preempted by a new action execution request.

Cancel function: all actions except New Year's Eve and Scroll can be canceled at any moment by the handle that initiated the request.

Interrupt function: except for the two actions of New Year's greeting and rolling, all other actions can be directly interrupted by the gait switching request, mode switching request and movement instruction.

External tuning parameter: currently actions are divided into three categories:
- ROS layer direct encapsulation: use the existing gait for operation.
- The ROS layer sets up the parameter table, the motion control layer receives the parameter table and then sequentially parses the parameters and executes them: builds a `TOML` parameter file, reads the file and sends the parameter table. * The current way of establishing and tuning the parameter table requires engineers to do experiments and then adjust manually, in the future, we will design a software for automated programming. *
- The ROS layer only sends out specific IDs, the motion control layer is hard coded and directly executed: the bottom layer is hard coded, and the two actions that cannot be interrupted or preempted are such a mechanism.

Feedback: once a legitimate request is received, the motion execution service detects the progress of the motion execution at a frequency of `rate_common_`Hz (from 0 to 100) and returns the current execution status at that frequency, while updating the `orderstamped` of the `robot_control_state_`.

Result:
- Successful execution: update the `id` in `robot_control_state_` to `MONO_ORDER_NULL`, restore the state, and return success.
- Execution Failure: update `id` in `robot_control_state_` to `MONO_ORDER_NULL`, restore state, return failure.

#### **Action Directive**

Data format: refer to [motion_msgs](... /cyberdog_interfaces/motion_msgs/README.md)

Legitimate request: an action command with the proper source id and the specified frame id with a recent timestamp.
- Proper source id: In different modes, the action instructions accepted by the decision maker are distinguished by the value of the source id. As shown below, the macro definition is taken from motion_msgs/msg/SE3VelocityCMD.msg.
  - Remote control mode: accepts INTERNAL and REMOTEC
  - Exploration mode: accepts INTERNAL, REMOTEC and NAVIGATOR.
  - Tracking mode: accepts INTERNAL and NAVIGATOR
- Specified frame id: all action commands must have a frame id of BODY_FRAME (taken from motion_msgs/msg/Frameid.msg).
- Newer timestamp indicates that the timestamp of the initiation of this command is newer than the timestamp of the last action command, usually the current system time in the same time zone is sufficient.

## **Running & Debugging**
---
This module supports standalone run and start system run, and can be combined with two control modes, GRPC or Joystick.

### **Running independently**

Standalone run, i.e. run with `ros2 run`, is commonly used for single-function testing and single-function debugging.

The full command is:

``
ros2 run cyberdog_decisionmaker decisionmaker
```

This startup state can only test motion related functions as there are no external parameters.

### **Starting the system running**

Starting the system running means using `ros2 launch` to start, Iron Egg will boot up and automatically call this script to start, in addition to starting the decision node, it also starts several other nodes at the same time, please refer to [cyberdog_bringup] (... /cyberdog_bringup) for more information.

The full command is:

``
ros2 launch cyberdog_bringup lc_bringup_launch.py
```

This launch state is the normal startup process for the robot and allows you to test all functionality.

### **Debugging Methods**

This module supports built-in debugging mode and GDB debugging.

#### Built-in debugging mode

`motion_manager` detects three macro definitions, the master switch to turn on debugging, action debugging and simulating motion data.

``
DEBUG_ALL // for complete debug
DEBUG_MOTION // for gait & motion debug
DEBUG_MOCK // for mock lcm messages
```

To enter debug mode, you need to turn on `DEBUG_ALL`:
- If you need to output motion related logs, you can turn on `DEBUG_MOTION`.
- If there is no LCM data input source, you can turn on `DEBUG_MOCK`.
- If there is no LCM data input source, you can turn on `DEBUG_MOTION` and `DEBUG_MOCK` at the same time.

Debugging can be enabled by creating `.debug_xxx` in the root directory of `decision_maker`, e.g.

```shell
$ touch .debug_all
```

The correspondences are shown in the following table

|DEF|File|
|---|----|
|DEBUG_ALL|.debug_all|.
|DEBUG_MOTION|.debug_motion|
|DEBUG_MOCK|.debug_mock|

#### GDB Debugging

1. First add the `-g` compile flag to `CMakeLists.txt`, usually in the `add_compile_options` function.
2. Then follow the [cyberdog_bringup](... /cyberdog_bringup](.../cyberdog_bringup) and start it with the `gdb` prefix.
3. Make sure you have debugging terminal tools on your system, including `gdb` and `xterm`, and install them if you don't.
4. Ensure that there are no identical nodes under the same `Domain ID` and the same `namespace`, and then use `Launch` to start the system in an environment with a graphical interface.
5. Start debugging.

## **[Future Optimization]**
---
- Improve perceptual function decisions
- Refine interaction function decisions
- Improve automation functionality decisions
- Modularization & Plug-in Single Point Functions
- Dynamic tuning of parameters
- Online Programming of Actions
- Customized Configuration of Functions