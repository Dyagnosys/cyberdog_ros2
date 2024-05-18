# cyberdog bringup

## Briefly ##

This module is the startup module that launches all other submodules.

## Total startup flow

This module is used to start all ROS 2 processes and is applied directly to the system services.

## General startup process

The complete startup process is performed by [systemd service](https://git.n.xiaomi.com/mirp/cyberdog_ros2_deb/-/blob/master/src/etc/systemd/system/cyberdog_ros2. service) calls [lc_bringup_launch.py](https://git.n.xiaomi.com/mirp/cyberdog_bringup/-/blob/master/launch/lc_bringup_launch.py) to start all other node startup scripts, and `lc_bringup_launch.py` calls `launch_nodes.yaml`, `launch_groups.yaml`, `default_param.yaml`, and `remappings.yaml` in the `params` folder to perform the startup. The four `YAML` files above will be configured with `nodes that can be launched`, `nodes to be launched in groups`, `parameters to be used within the nodes` and `reappings to be remapped` within the nodes.

In `launch_nodes.yaml` two types of nodes will be provided: `base_nodes` and `other_nodes`, the former are nodes that are necessary for the robot to move and continue to work, and if they `don't exist` or `failed to start` they will report an error and terminate the startup, and the latter are other nodes, and if they `don't exist` or `failed to start` they will only report a warning and continue the startup.

The nodes required for different conditions can be configured as needed in `launch_groups.yaml`.

In `default_param.yaml` the external parameters for all nodes are provided, grouped by node name.

The `remappings.yaml` provides the remappings needed for all nodes, grouped by node name.

## Maintenance method

Based on the above description, the current version (210726) does not need to start the script `lc_bringup_launch.py` to add or modify nodes, but only needs to maintain the `launch_nodes.yaml`, `launch_groups.yaml`, `default_param.yaml` and `remappings.yaml`. remappings.yaml`, `launch_groups.yaml`, `default_param.yaml` and `remappings.yaml`. How to use these four files is described below:

### launch_nodes.yaml file use

This file is used to configure the nodes that need to be launched and has the following main structure:
```yaml
launch_nodes.
    debug_param: "lxterminal -e gdb -ex run --args"

    # Must launch, if package or executable error, launcher will stop and throw error
    base_nodes.

        # - example_node_def_name.
        # package: "example_pkg" (string)
        # executable: "example_exe" (string)
        # #[optional] output_screen: false (true/false)
        # #[optional] name: 'example_name' (string)
        # #[optional] load_nodes_param: false (true/false)
        # #[optional] enable_debug: false (true/false)

    # Try to launch, if package or executable error, launcher will notice but skip that node
    other_nodes.
```
- `launch_nodes` contains 1 test parameter `debug_param`, 2 launch node configuration types (`base_nodes`, `other_nodes`)

Test Parameters:
- When `enable_debug == true` is configured in the node, the `debug_param` parameter will be written to the ros2 Node in prefix

Configuration types:
- Configuration types are `base_nodes` and `other_nodes`.
- `base_nodes` are the base nodes, which are the core functional nodes, when these nodes fail to start they will report an error and `stop` all other nodes from starting.
- `other_nodes` are auxiliary nodes, they are other functional nodes, when these nodes fail to start, they will alert and `skip` the error node to continue starting.

In `base_nodes` and `other_nodes`, you can configure the nodes you want to start.

Define the node configuration name (any name):
```
# - example_node_def_name.
```
Configure the name of the package that the node starts (according to the package):
```
# package: "example_pkg" (string)
```
Name of the executable that the configuration node launches (according to the package):
```
# executable: "example_exe" (string)
```
Configuration node standard output location (can be defaulted):
```
# #[optional] output_screen: false (true/false)
```
Configure the node name (can be defaulted):
```
# #[optional] name: 'example_name' (string)
```
Configure whether to load external parameters (from `params/default_param.yaml`, can be defaulted):
```
# #[optional] load_nodes_param: false (true/false)
```
Configure whether or not to load debug parameters (can be defaulted)
```
# #[optional] enable_debug: false (true/false)
```

### The launch_groups.yaml file uses the

This file is used to configure launch groups and has the following main structure:
``## yaml
launch_groups.
  target_launch_group: default
  groups.
    # example_group_whitlist.
    # launch: #only launch.
    # - example_node_def_name1
    # - example_node_def_name2
    # example_group_blacklist.
    # except: #dont launch.
    # - example_node_def_name1 # - example_node_def_name2
    # - example_node_def_name2
    example_node_def_name1 # - example_node_def_name2
      # - example_node_def_name1 # - example_node_def_name2
        - cyberdog_motion_test_cmd
        - decision_test
```
- `target_launch_group` is the current launch group.
- Configure the launch group in `groups`.

There are two ways to configure a launch group in `groups`, `launch` and `except`, but there can only be one way to launch in a launch group
- `launch`: only launch all the nodes listed.
- `except`: start all nodes except those listed.

### The default_param.yaml file uses the

This file is used for parameterization and has the following structure:

```yaml
## Node name.
example_node_def_name.
  # fixed, indicated as ROS parameters
  ros__parameters.
    # Parameter names and values
    value_a: 1
```

See [ROS2 YAML For Parameters](https://roboticsbackend.com/ros2-yaml-params/) for details.

### remapping.yaml file usage
This file is used for remapping and has the following main structure:
``yaml
remappings.
  example_node_def_name.
    - ['/topic_before1','/topic_after1']
    - ['/topic_before2','/topic_after2']
```
where `example_node_def_name` is filled in according to the node name defined in the `launch_nodes.yaml` file




