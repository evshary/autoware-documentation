# Source installation

## Prerequisites

- [Ubuntu 20.04](https://releases.ubuntu.com/20.04/)
- [Git](https://git-scm.com/)
  - [Registering SSH keys to GitHub](https://github.com/settings/keys) is preferable.

```bash
sudo apt-get -y update
sudo apt-get -y install git
```

## How to set up a development environment

1. Clone `autowarefoundation/autoware` and move to the directory.

   ```bash
   git clone https://github.com/autowarefoundation/autoware.git
   cd autoware
   ```

2. You can install the dependencies either manually or using the provided Ansible script.

### Installing dependencies manually

Install ROS2:

```bash
# Taken from: https://docs.ros.org/en/galactic/Installation/Ubuntu-Install-Debians.html

# You will need to add the ROS 2 apt repository to your system. First, make sure that the Ubuntu Universe repository is enabled by checking the output of this command.
apt-cache policy | grep universe

# 500 http://us.archive.ubuntu.com/ubuntu focal/universe amd64 Packages
#     release v=20.04,o=Ubuntu,a=focal,n=focal,l=Ubuntu,c=universe,b=amd64
# If you don’t see an output line like the one above, then enable the Universe repository with these instructions.
sudo apt install software-properties-common
sudo add-apt-repository universe

# Now add the ROS 2 apt repository to your system. First authorize our GPG key with apt.
sudo apt update && sudo apt install curl gnupg lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

# Then add the repository to your sources list.
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# Update your apt repository caches after setting up the repositories.
sudo apt update

# Desktop Install
sudo apt install ros-galactic-desktop

# Environment setup
# (Optional) You can source ros2 galactic in the ~/.bashrc file.
echo 'source /opt/ros/galactic/setup.bash' >> ~/.bashrc
```

Install ROS2 Dev Tools:

```bash
# Taken from https://docs.ros.org/en/galactic/Installation/Ubuntu-Development-Setup.html
sudo apt update && sudo apt install -y \
  build-essential \
  cmake \
  git \
  python3-colcon-common-extensions \
  python3-flake8 \
  python3-pip \
  python3-pytest-cov \
  python3-rosdep \
  python3-setuptools \
  python3-vcstool \
  wget

# Install some pip packages needed for testing
python3 -m pip install -U \
  flake8-blind-except \
  flake8-builtins \
  flake8-class-newline \
  flake8-comprehensions \
  flake8-deprecated \
  flake8-docstrings \
  flake8-import-order \
  flake8-quotes \
  pytest-repeat \
  pytest-rerunfailures \
  pytest \
  setuptools

# Initalize rosdep
sudo rosdep init
rosdep update
```

Install the `rmw_cyclonedds_cpp` RMW Implementation:

```bash
# For details: https://docs.ros.org/en/galactic/How-To-Guides/Working-with-multiple-RMW-implementations.html
sudo apt update
sudo apt install ros-galactic-rmw-cyclonedds-cpp

# (Optional) You set the default RMW implementation in the ~/.bashrc file.
echo 'export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp' >> ~/.bashrc
```

Install pacmod:

```bash
# Taken from https://github.com/astuff/pacmod3#installation
sudo apt install apt-transport-https
sudo sh -c 'echo "deb [trusted=yes] https://s3.amazonaws.com/autonomoustuff-repo/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/autonomoustuff-public.list'
sudo apt update
sudo apt install ros-galactic-pacmod3
```

Install rest of the dependencies:

```bash
# Install gdown to download files from CMakeLists.txt
pip3 install gdown

sudo apt install geographiclib-tools

# Add EGM2008 geoid grid to geographiclib
sudo geographiclib-get-geoids egm2008-1

pip3 install pre-commit clang-format

# Install Golang (Add Go PPA for shfmt)
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt install golang
```

#### Install Nvidia Packages

Follow these instructions to download and install the CUDA Toolkit 11.4 and the corresponding NVIDIA Driver for Ubuntu 20.04.

```bash
# Modified from:
# https://developer.nvidia.com/cuda-11-4-4-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_network
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
sudo apt-get update
sudo apt install cuda-11-4
```

Perform the post installation actions:

```bash
# Taken from: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#post-installation-actions
echo 'export PATH=/usr/local/cuda-11.4/bin${PATH:+:${PATH}}' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-11.4/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}' >> ~/.bashrc
```

Install TensorRT and cuDNN:

```bash
# Taken from: https://docs.nvidia.com/deeplearning/tensorrt/install-guide/index.html#installing
version="8.2.2-1+cuda11.4"
sudo apt-get install libnvinfer8=${version} libnvonnxparsers8=${version} libnvparsers8=${version} libnvinfer-plugin8=${version} libnvinfer-dev=${version} libnvonnxparsers-dev=${version} libnvparsers-dev=${version} libnvinfer-plugin-dev=${version} python3-libnvinfer=${version}
sudo apt-mark hold libnvinfer8 libnvonnxparsers8 libnvparsers8 libnvinfer-plugin8 libnvinfer-dev libnvonnxparsers-dev libnvparsers-dev libnvinfer-plugin-dev python3-libnvinfer

version="8.2.2.26-1+cuda11.4"
sudo apt-get install libcudnn8=${version} libcudnn8-dev=${version}
sudo apt-mark hold libcudnn8 libcudnn8-dev
```

### Installing dependencies using Ansible

Be very careful with this method. Make sure you read and confirmed all the steps in the Ansible configuration before using it.

```bash
./setup-dev-env.sh
```

## How to set up a workspace

1. Create the `src` directory and clone repositories into it.

   Autoware uses [vcstool](https://github.com/dirk-thomas/vcstool) to construct workspaces.

   ```bash
   cd autoware
   mkdir src
   vcs import src < autoware.repos
   ```

2. Install dependent ROS packages.

   Autoware requires some ROS 2 packages in addition to the core components.
   The tool `rosdep` allows an automatic search and installation of such dependencies.

   ```bash
   source /opt/ros/galactic/setup.bash
   rosdep install --from-paths src --ignore-src --rosdistro $ROS_DISTRO
   ```

3. Build the workspace.

   Autoware uses [colcon](https://colcon.readthedocs.io/en/released/index.html) to build workspaces.
   Refer to the documentation for more advanced options.

   ```bash
   colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```

## How to update a workspace

1. Update the `.repos` file.

   ```bash
   cd autoware
   git pull
   ```

2. Update the repositories.

   ```bash
   vcs import src < autoware.repos
   vcs pull
   ```

   For Git users:

   - `vcs import` is similar to `git checkout`.
     - Note that it doesn't pull from the remote.
   - `vcs pull` is similar to `git pull`.
     - Note that it doesn't switch branches.

   Refer to the [official documentation](https://github.com/dirk-thomas/vcstool) for more information.

3. Install dependent ROS packages.

   ```bash
   source /opt/ros/galactic/setup.bash
   rosdep install --from-paths src --ignore-src --rosdistro $ROS_DISTRO
   ```

4. Build the workspace.

   ```bash
   colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```
