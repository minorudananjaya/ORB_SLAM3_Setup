# ORB_SLAM3 Setup On Ubuntu 20.04
Clone ORB-SLAM3 from original repository. ([ORB-SLAM3 Repo](https://github.com/UZ-SLAMLab/ORB_SLAM3))
```
cd ~
mkdir Dev && cd Dev

git clone https://github.com/UZ-SLAMLab/ORB_SLAM3.git
```

## Install Pangolin
Clone from [Pangolin Repo](https://github.com/stevenlovegrove/Pangolin)
```
cd ~/Dev
git clone https://github.com/stevenlovegrove/Pangolin.git
```

Install all dependencies
```
sudo apt install libgl1-mesa-dev
sudo apt install libglew-dev
sudo apt install cmake
sudo apt install libpython2.7-dev
sudo apt install pkg-config
sudo apt install libegl1-mesa-dev libwayland-dev libxkbcommon-dev wayland-protocols
```

Install all liberay dependencies.
```
sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
sudo apt update

sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev

sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev libjasper-dev

sudo apt-get install libglew-dev libboost-all-dev libssl-dev
```

Install Pangolin
```
cd Pangolin
mkdir build
cd build
cmake ..
make -j4
sudo make install
cmake --build .
```

## Install OpenCV
Clone from [OpenCV Repo](https://github.com/opencv/opencv)
```
cd ~/Dev
git clone https://github.com/opencv/opencv.git
```

Involve OpenCV 3.2.0 since ORB-SLAM3 was tested on this version.
```
cd opencv
git checkout 3.2.0
```

Put the following at the top of header file after ``gedit ./modules/videoio/src/cap_ffmpeg_impl.hpp``
```
#define AV_CODEC_FLAG_GLOBAL_HEADER (1 << 22)
#define CODEC_FLAG_GLOBAL_HEADER AV_CODEC_FLAG_GLOBAL_HEADER
#define AVFMT_RAWPICTURE 0x0020
```

Install Dependencies
```
sudo apt-get update
sudo apt-get install build-essential libgtk2.0-dev libavcodec-dev libavformat-dev libjpeg.dev
libtiff4.dev libswscale-dev libjasper-dev
```

Install OpenCV 3.2.0
```
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=Release -D WITH_CUDA=OFF -D CMAKE_INSTALL_PREFIX=/usr/local ..
make -j4
sudo make install
```

Configure the environment for OpenCV
```
sudo /bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig
sudo gedit /etc/bash.bashrc
```

Put the following at the bottom of file after `sudo gedit /etc/bash.bashrc`
```
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig
export PKG_CONFIG_PATH
```

Apply changes to environment
```
source /etc/bash.bashrc
sudo apt-get install mlocate
sudo updatedb
pkg-config --modversion opencv
```

## Install Eigen3
Clone from [Eigen3 Repo](https://github.com/eigenteam/eigen-git-mirror)
```
cd ~/Dev
git clone https://github.com/eigenteam/eigen-git-mirror.git
```

Install Eigen3
```
cd eigen-git-mirror
mkdir build
cd build
cmake ..
sudo make install
```

## Install Boost C++ Libraries and Other Dependencies for Python
Download the '*UNIX .tar.gz*' boost C++ library from [Boost C++ Libraries](https://www.boost.org/users/download/) to the Dev folder. I used the Boost 1.82.0 version.
```
cd ~/Dev
tar -xzvf boost_1_82_0.tar.gz
cd boost_1_82_0
sudo ./bootstrap.sh
sudo ./b2 install
```

Other Dependencies
```
sudo apt-get install libssl-dev
```

## Install ORB-SLAM3
```
cd ~/Dev/ORB_SLAM3
```
Following changes were made to successfully install ORB-SLAM3.

 1. Update *CMakeLists.txt* in ORB_SLAM3 directory from -std=c++11 to -std=c++14
```
CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++14.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()
```

 2. Update line number 33 in *CMakeLists.txt* in ORB_SLAM3 directory from OpenCV 4.4 to OpenCV 4.2.
```
find_package(OpenCV 4.2)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 4.4 not found.")
   endif()
```

 3. Update line number 50 in *CMakeLists.txt* in ORB_SLAM3 directory from `${EIGEN3_INCLUDE_DIR}` to `$ENV{EIGEN3_INCLUDE_DIR}`.
```
include_directories(
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/include
${PROJECT_SOURCE_DIR}/include/CameraModels
${PROJECT_SOURCE_DIR}/Thirdparty/Sophus
$ENV{EIGEN3_INCLUDE_DIR}
${Pangolin_INCLUDE_DIRS}
)
```

 4. Change the header file gedit ./include/LoopClosing.h at line 51 from `Eigen::aligned_allocator<std::pair<const KeyFrame*, g2o::Sim3> > > KeyFrameAndPose;` to:
```
Eigen::aligned_allocator<std::pair<KeyFrame *const, g2o::Sim3> > > KeyFrameAndPose;
```

Install ORB-SLAM3. (Run `./build.sh` multiple times to ensure 100% installation)
```
chmod +x build.sh
./build.sh
```

# Dowload EuRoC Test Datasets
Make dataset directory.
```
cd ~
mkdir -p Datasets/EuRoc
```

Download Machine Hall 1 (MH01) Easy Dataset
```
cd ~/Datasets/EuRoc/
wget -c http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/machine_hall/MH_01_easy/MH_01_easy.zip
mkdir MH01
unzip MH_01_easy.zip -d MH01/
```

Download Vicon Room 1 (V101) Easy Dataset
```
cd ~/Datasets/EuRoc/
wget -c http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/vicon_room1/V1_01_easy/V1_01_easy.zip
mkdir V101
unzip V1_01_easy.zip -d V101/
```
Download Vicon Room 2 (V201) Easy Dataset
```
cd ~/Datasets/EuRoc/
wget -c http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/vicon_room2/V2_01_easy/V2_01_easy.zip
mkdir V201
unzip V2_01_easy.zip -d V201/
```

# Run Simulation on EuRoC Datasets
```
cd ~/Dev/ORB_SLAM3

## For MH01 Dataset (Choose one of the three to run simulations on particular sensor configuration)
# Mono
./Examples/Monocular/mono_euroc ./Vocabulary/ORBvoc.txt ./Examples/Monocular/EuRoC.yaml ~/Datasets/EuRoc/MH01 ./Examples/Monocular/EuRoC_TimeStamps/MH01.txt dataset-MH01_mono

# Stereo
./Examples/Stereo/stereo_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo/EuRoC.yaml ~/Datasets/EuRoc/MH01 ./Examples/Stereo/EuRoC_TimeStamps/MH01.txt dataset-MH01_stereo

# Stereo + Inertial
./Examples/Stereo-Inertial/stereo_inertial_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo-Inertial/EuRoC.yaml ~/Datasets/EuRoc/MH01 ./Examples/Stereo-Inertial/EuRoC_TimeStamps/MH01.txt dataset-MH01_stereoi


## For V101 Dataset (Choose one of the three to run simulations on particular sensor configuration)
# Mono
./Examples/Monocular/mono_euroc ./Vocabulary/ORBvoc.txt ./Examples/Monocular/EuRoC.yaml ~/Datasets/EuRoc/V101 ./Examples/Monocular/EuRoC_TimeStamps/V101.txt dataset-V101_mono

# Stereo
./Examples/Stereo/stereo_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo/EuRoC.yaml ~/Datasets/EuRoc/V101 ./Examples/Stereo/EuRoC_TimeStamps/V101.txt dataset-V101_stereo

# Stereo + Inertial
./Examples/Stereo-Inertial/stereo_inertial_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo-Inertial/EuRoC.yaml ~/Datasets/EuRoc/V101 ./Examples/Stereo-Inertial/EuRoC_TimeStamps/V101.txt dataset-V101_stereoi



## For V201 Dataset (Choose one of the three to run simulations on particular sensor configuration)
# Mono
./Examples/Monocular/mono_euroc ./Vocabulary/ORBvoc.txt ./Examples/Monocular/EuRoC.yaml ~/Datasets/EuRoc/V201 ./Examples/Monocular/EuRoC_TimeStamps/V201.txt dataset-V201_mono

# Stereo
./Examples/Stereo/stereo_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo/EuRoC.yaml ~/Datasets/EuRoc/V201 ./Examples/Stereo/EuRoC_TimeStamps/V201.txt dataset-V201_stereo

# Stereo + Inertial
./Examples/Stereo-Inertial/stereo_inertial_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo-Inertial/EuRoC.yaml ~/Datasets/EuRoc/V201 ./Examples/Stereo-Inertial/EuRoC_TimeStamps/V201.txt dataset-V201_stereoi
```
# Plotting Estimate vs Ground True for Validation
Requires numpy and matplotlib installed in python2.7.
```
sudo apt install curl
cd ~/Desktop
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
sudo python2 get-pip.py
pip2.7 install numpy matplotlib
```

Plot estimate vs ground truth.
```
cd ~/Dev/ORB_SLAM3

## Run either of the 3 once the simulation has been run on it.
# MH01
python evaluation/evaluate_ate_scale.py evaluation/Ground_truth/EuRoC_left_cam/MH01_GT.txt f_dataset-MH01_stereo.txt --plot MH01_stereo.pdf

# V101
python evaluation/evaluate_ate_scale.py evaluation/Ground_truth/EuRoC_left_cam/V101_GT.txt f_dataset-V101_stereo.txt --plot V101_stereo.pdf

# V201
python evaluation/evaluate_ate_scale.py evaluation/Ground_truth/EuRoC_left_cam/V201_GT.txt f_dataset-V201_stereo.txt --plot V201_stereo.pdf
```
