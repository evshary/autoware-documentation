# Docker installation

!!! info

    Since this page explains Docker-specific information, it is recommended to see [Source installation](./source-installation.md) as well if you need detailed information.

## Prerequisites

- [Git](https://git-scm.com/)

## How to set up a development environment

1. Clone `autowarefoundation/autoware` and move to the directory.

   ```bash
   git clone https://github.com/autowarefoundation/autoware.git
   ```

2. You can install the dependencies either manually or using the provided Ansible script.

### Installing dependencies manually

Install Docker Engine and its post-installation steps:

```bash
# Taken from: https://docs.docker.com/engine/install/ubuntu/
# And: https://docs.docker.com/engine/install/linux-postinstall/

# Uninstall old versions
sudo apt-get remove docker docker-engine docker.io containerd runc

# Install using the repository
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker’s official GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Verify that Docker Engine is installed correctly by running the hello-world image.
sudo docker run hello-world
# Note: This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

# Post-installation steps for Linux

# Create the docker group.
sudo groupadd docker

# Add your user to the docker group.
sudo usermod -aG docker $USER

# Log out and log back in so that your group membership is re-evaluated.

# Verify that you can run docker commands without sudo
docker run hello-world
# Note: This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.
```

Install Nvidia Container Toolkit:

```bash
# Taken from https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#setting-up-nvidia-container-toolkit

# Setup the package repository and the GPG key:
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -s -L https://nvidia.github.io/libnvidia-container/gpgkey | sudo apt-key add - \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Install the nvidia-docker2 package (and dependencies) after updating the package listing:
sudo apt-get update
sudo apt-get install -y nvidia-docker2

# Restart the Docker daemon to complete the installation after setting the default runtime:
sudo systemctl restart docker

# At this point, a working setup can be tested by running a base CUDA container:
sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi

# This should result in a console output shown below:
# +-----------------------------------------------------------------------------+
# | NVIDIA-SMI 450.51.06    Driver Version: 450.51.06    CUDA Version: 11.0     |
# |-------------------------------+----------------------+----------------------+
# | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
# | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
# |                               |                      |               MIG M. |
# |===============================+======================+======================|
# |   0  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
# | N/A   34C    P8     9W /  70W |      0MiB / 15109MiB |      0%      Default |
# |                               |                      |                  N/A |
# +-------------------------------+----------------------+----------------------+
#
# +-----------------------------------------------------------------------------+
# | Processes:                                                                  |
# |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
# |        ID   ID                                                   Usage      |
# |=============================================================================|
# |  No running processes found                                                 |
# +-----------------------------------------------------------------------------+
```

Install Docker Compose:

```bash
# Taken from: https://docs.docker.com/compose/install/#install-compose-on-linux-systems

# Run this command to download the current stable release of Docker Compose:
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Apply executable permissions to the binary:
sudo chmod +x /usr/local/bin/docker-compose

# Optionally Install command completion
# Place the completion script in /etc/bash_completion.d/
sudo curl \
    -L https://raw.githubusercontent.com/docker/compose/1.29.2/contrib/completion/bash/docker-compose \
    -o /etc/bash_completion.d/docker-compose

# Reload your terminal. You can close and then open a new terminal, or reload your setting with source ~/.bashrc command in your current terminal.
source ~/.bashrc

# Test the installation.
docker-compose --version
```

Install rocker:

```bash
# Taken from: https://github.com/osrf/rocker#installation
sudo apt update
sudo apt-get install python3-rocker
```

### Installing dependencies using Ansible

Be very careful with this method. Make sure you read and confirmed all the steps in the Ansible configuration before using it.

If you've manually installed the dependencies, you can skip this section.

```bash
./setup-dev-env.sh docker
```

## How to set up a workspace

1. Launch a Docker container

   ```bash
   rocker --nvidia --x11 --user --volume $HOME/autoware -- ghcr.io/autowarefoundation/autoware-universe:latest
   ```

   See [here](https://github.com/autowarefoundation/autoware/tree/main/docker/README.md) for more advanced usage.

   After that, move to the workspace in the container:

   ```bash
   cd autoware
   ```

2. Create the `src` directory and clone repositories into it.

   ```bash
   mkdir src
   vcs import src < autoware.repos
   ```

3. Build the workspace.

   ```bash
   colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```

## How to update a workspace

1. Update the Docker image.

   ```bash
   docker pull ghcr.io/autowarefoundation/autoware-universe:latest
   rocker --nvidia --x11 --user --volume $HOME/autoware -- ghcr.io/autowarefoundation/autoware-universe:latest
   ```

2. Update the `.repos` file.

   ```bash
   cd autoware
   git pull
   ```

3. Update the repositories.

   ```bash
   vcs import src < autoware.repos
   vcs pull
   ```

4. Build the workspace.

   ```bash
   colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```

## Troubleshooting

Here are solutions for a few specific errors:

### cuda error: forward compatibility was attempted on non supported hw

When starting Docker with GPU support enabled for NVIDIA graphics, you may sometimes receive the following error:

```bash
docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "process_linux.go:449: container init caused \"process_linux.go:432: running prestart hook 0 caused \\\"error running hook: exit status 1, stdout: , stderr: nvidia-container-cli: initialization error: cuda error: forward compatibility was attempted on non supported hw\\\\n\\\"\"": unknown.
ERROR: Command return non-zero exit code (see above): 125
```

This usually indicates that a new NVIDIA graphics driver has been installed (usually via `apt`) but the system has not yet been restarted. A similar message may appear if the graphics driver is not available, for example because of resuming after suspend.

To fix this, restart your system after installing the new NVIDIA driver.

## Tips

### Non-native arm64 System

This section describes a process to run `arm64` systems on `amd64` systems using [`qemu-user-static`](https://github.com/multiarch/qemu-user-static).

Initially, your system is usually incompatible with `arm64` systems.
To check that:

```sh-session
$ docker run --rm -t arm64v8/ubuntu uname -m
WARNING: The requested image's platform (linux/arm64/v8) does not match the detected host platform (linux/amd64) and no specific platform was requested
standard_init_linux.go:228: exec user process caused: exec format error
```

Installing `qemu-user-static` enables us to run `arm64` images on `amd64` systems.

```sh-session
$ sudo apt-get install qemu-user-static
$ docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
$ docker run --rm -t arm64v8/ubuntu uname -m
WARNING: The requested image's platform (linux/arm64/v8) does not match the detected host platform (linux/amd64) and no specific platform was requested
aarch64
```

To run Autoware's Docker images of `arm64` architecture, add the suffix `-arm64`.

```sh-session
$ docker run --rm -it ghcr.io/autowarefoundation/autoware-universe:latest-arm64
WARNING: The requested image's platform (linux/arm64) does not match the detected host platform (linux/amd64) and no specific platform was requested
root@5b71391ad50f:/autoware#
```
