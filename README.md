# throwbot

A simulated mobile base + 3-DOF arm, built in ROS2 Jazzy + Gazebo Harmonic,
working toward: throw an object into a camera's view, classify it, and
trigger a task (e.g. weld two metal pieces, sort colored boxes) that drives
the base to a tool station and runs the arm through it.

## Current status

**Built and verified in RViz** (every link's transform checked with
`tf2_echo`, not just eyeballed):
- Mobile base: `base_footprint`, `base_link`, two drive wheels, a free
  caster for a third stable contact point
- 3-DOF arm: waist (yaw) → shoulder (pitch) → elbow (pitch) → `tool_frame`
  mount point, chained onto a pedestal on the chassis

**Added, not yet confirmed working end-to-end in Gazebo:**
- `ros2_control` + `gz_ros2_control` for the arm's 3 joints
  (`joint_trajectory_controller`)
- Gazebo's `DiffDrive` system plugin driving the base from `/cmd_vel`
- A simplified grasp mechanism using Gazebo's `DetachableJoint` plugin -
  rigidly attaches/detaches a test object to `tool_frame` on command,
  rather than simulating real gripper fingers

**Not started yet:**
- Nav2 (mobile base navigation between waypoints)
- MoveIt2 (arm motion planning - the arm currently only takes raw joint
  trajectory commands)
- Perception (camera + object classification)
- Task dispatcher + the actual weld/sort behaviors

## Why only 3 DOF, and what that means

The arm has no wrist joint, so it can't independently set a tool's
orientation - wherever the elbow leaves it, that's the orientation. Task
design has to work around this (fixed approach geometries) rather than
assuming arbitrary pose control.

## Running it

```bash
cd ~/throwbot_ws
colcon build --symlink-install
source install/setup.bash

# geometry-only check, no physics
ros2 launch throwbot_description display.launch.py

# full Gazebo simulation
ros2 launch throwbot_description gazebo.launch.py
```

## Package layout

```
throwbot_description/
  urdf/throwbot.urdf       # the whole robot
  config/controllers.yaml  # ros2_control controller config
  config/bridge.yaml       # ROS <-> Gazebo topic bridge
  worlds/throwbot_world.sdf
  launch/display.launch.py # RViz only
  launch/gazebo.launch.py  # full sim
  rviz/throwbot.rviz
```
