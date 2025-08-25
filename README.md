# ROS 2 Humble Desktop (noVNC) Container

A Dockerized Ubuntu 22.04 (Jammy) MATE desktop preloaded with ROS 2 Humble. Access the full desktop from your browser via noVNC, with an optional VNC port for native clients. A persistent ROS workspace is mounted at `/home/ubuntu/ros_ws` via a Docker volume.

Includes ROS build tools (colcon, rosdep), Firefox, Terminator, and Gazebo/ros-ign on `amd64`.

## Features
- Ubuntu MATE desktop via VNC/noVNC (browser UI)
- ROS 2 Humble (`desktop` variant) preinstalled
- Gazebo and `ros-ign` on `amd64` only
- Persistent workspace at `/home/ubuntu/ros_ws` (Docker volume)
- Handy tools: Firefox, Terminator, build-essential, colcon, rosdep

## Prerequisites
- Docker and Docker Compose v2
- Create the external volume expected by `docker-compose.yaml`:
  - `docker volume create ros-volume`

## Quick Start
1. Build and start in the background:
   - `docker compose up -d --build`
2. Open the desktop in your browser:
   - noVNC: `http://localhost:6080`
   - Default password: `ubuntu` (unless overridden)
3. Optional: connect with a native VNC client:
   - Host: `localhost`
   - Port: `5901`
   - Password: `ubuntu`

To stop the container:
- `docker compose down`

The `ros-volume` is external and persists across runs. Your ROS workspace lives in `/home/ubuntu/ros_ws` inside the container.

## Using ROS Inside the Container
- Open the “Terminator” shortcut on the desktop or launch a terminal from the MATE menu.
- ROS environment is auto-sourced via `.bashrc`:
  - `source /opt/ros/humble/setup.bash` (already added)
  - Colcon autocompletion is enabled.
- Sanity checks:
  - `ros2 doctor`
  - `rviz2`
  - `gazebo` (only on `amd64`)

### Create and Build a Workspace
```
mkdir -p ~/ros_ws/src
cd ~/ros_ws
colcon build
source install/setup.bash
```

## Ports, Volumes, and Defaults
- Ports:
  - `6080:80` — noVNC in the browser
  - `5901:5901` — optional direct VNC access
- Volumes:
  - External: `ros-volume` → mounted at `/home/ubuntu/ros_ws`
- Default user/password inside the desktop:
  - User: `ubuntu`
  - Password (also used for VNC): `ubuntu`

## Customization
You can override the user and password via environment variables when starting the container. Example `docker-compose.yaml` snippet:

```
services:
  ros-humble-vnc:
    environment:
      - USER=developer
      - PASSWORD=changeme
```

Other noteworthy settings already in `docker-compose.yaml`:
- `security_opt: seccomp=unconfined` — required for Jammy-based images
- `shm_size: 1g` — improves RViz/Gazebo stability

## Architecture Notes
- `amd64`: Installs Gazebo and `ros-ign`.
- `arm64` (e.g., Apple Silicon): Gazebo and `ros-ign` are not installed.

## Troubleshooting
- Compose fails about missing volume `ros-volume`:
  - Run `docker volume create ros-volume` and retry.
- Blank/black noVNC screen or failures launching desktop:
  - Ensure `security_opt: seccomp=unconfined` is present (it is in this repo).
  - Refresh the browser or restart: `docker compose restart`.
- VNC/noVNC authentication fails:
  - Default is `ubuntu`. If you set `PASSWORD`, use that.
- Check logs:
  - `docker compose logs -f`

## Attribution
This setup draws from and builds upon the excellent work in `Tiryoh/docker-ros2-desktop-vnc` (Apache-2.0).

