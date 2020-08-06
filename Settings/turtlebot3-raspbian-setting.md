#  1. 라즈비안(Buster)에 ros melodic 설치

1.   https://www.raspberrypi.org/downloads/raspbian/

다음 링크에 방문하여 라즈비안을 다운받는다. 

3개의 버전이 있는데 두번째를 다운받고 sd카드에 설치한다.

2. 기본 설정을 마친 후 터미널 창을 켜 다음을 입력하여 텍스트 편집기를 설치한다.

```
sudo apt-get install gedit
```

3. 설치 과정에서 메모리 부족할수 있어 디폴트 값인 100를 2048로 스왑파일 크기를 변경해준다.

모든 설치과정이 끝나면 다시 100으로 변경해주어야 한다.

다음 명령어로 편집기를 열고 아래의 코드로 변경해준다.

```
sudo gedit /etc/dphys-swapfile
```

```
#CONF_SWAPSIZE=100
CONF_SWAPSIZE=2048
```

변경된 swapfile의 크기를 적용하기 위해 서비스를 재시작한다.

```
sudo /etc/init.d/dphys-swapfile restart
```

4. 본격적인 ROS Melodic을 설치한다. 위에부터 차례대로 터미널 창에 입력한다.

1) 의존성 패키지 다운로드 및 설치

```
$ sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
$ sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
$ sudo apt-get update
$ sudo apt-get upgrade -> 하라고 되어있으나 추천하지 않음. 여러가지 문제 발생 가능성 있음
$ sudo apt install -y python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential cmake
```

2) rosdpe 초기화 및 업데이트

```
$ sudo rosdep init
$ rosdpe update
```

3) Ros Melodic 설치

```
1. Ros 설치를 위한 repository 생성
$ mkidr -p ~/ros_catkin_ws
$ cd ~/ros_catkin_ws

2. Ros 간편 버전(rqt, rviz 포함되어있지 않음)
$ rosinstall_generator ros_comm --rosdistro melodic --deps --wet-only --tar > melodic-ros_comm-wet.rosinstall
$ wstool init src melodic-ros_comm-wet.rosinstall

Ros Desktop 버전(rqt, rviz 포함)
$ rosinstall_generator desktop --rosdistro melodic --deps --wet-only --tar > melodic-desktop-wet.rosinstall
$ wstool init src melodic-desktop-wet.rosinstall

4. 의존성 문재 해결
$ cd ~/ros_catkin_ws
$ rosdep install -y --from-paths src --ignore-src --rosdistro melodic -r --os=debian:buster

5. Ros Build
가장 빠르나 다운될 가능성이 있다.
$ sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/melodic

다운되면 다음과 같은 명령어로 대체 한다.
$ sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/melodic -j2

6. 기본적인 setting
$ cd
$ mkdir catkin_ws
$ cd catkin_ws
$ mkdir build
$ mkdir devel
$ mkdir src
$ cd src
turtlrebot 기본 패키지 다운
$ git clone https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver.git
$ git clone https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
$ git clone https://github.com/ROBOTIS-GIT/turtlebot3.git
$ cd ~/catkin_ws/src/turtlebot3
$ sudo rm -r turtlebot3_description/ turtlebot3_teleop/ turtlebot3_navigation/ turtlebot3_slam/ turtlebot3_example/
$ source /opt/ros/melodic/setup.bash
$ catkin_make
실패 시 없는 package 설치 후 catkin_make 다시 진행

$ gedit ~/.bashrc
다음의 텍스트를 추가 후 저장
alias cw='cd ~/catkin_ws'
alias cm='cd ~/catkin_ws && catkin_make'
source /opt/ros/melodic/setup.bash
source ~/catkin_ws/devel/setup.bash
export ROS_MASTER_URI=http://localhost:11311
export ROS_HOSTNAME=localhost
export TURTLEBOT3_MODEL=burger
$ source ~/.bashrc

7. 설치 확인
$ roscore

8. turtlebot3_bringup 실행할때 오류 발생
$ cd ~/ros_catkin_ws
$ sudo rm -rf *
$ rosinstall_generator rosserial --rosdistro melodic --deps --wet-only --tar > melodic-rosserial-wet.rosinstall
$ wstool init src melodic-rossrial-wet.rosinstall
$ cd ~/ros_catkin_ws
$ sudo rm -rf *
$ rosinstall_generator common_msgs --rosdistro melodic --deps --wet-only --tar > melodic-common_msgs-wet.rosinstall
$ wstool init src melodic-common_msgs-wet.rosinstall
```

# 2. 라즈비안에 opencv 설치

1. 설치되어 있는 opencv 제거

```
$ pkg-config --modversion opencv
Package opencv was not found in the pkg-config search path.
Perhaps you should add the directory containing `opencv.pc'
to the PKG_CONFIG_PATH environment variable
No package 'opencv' found
다음과 같이 나오는 경우 바로 설치 진행하면 된다.

$ pkg-config --modversion opencv
2.4.9.1
다음과 같이 나오는 경우 아래의 명령어를 통해 삭제 진행
$ sudo apt-get purge libopencv* python-opencv
$ sudo apt-get autoremove
```

2. 패키지 업그레이드 및 설치

```
2.1 패키지 다운로드주소 변경
$ sudo gedit /etc/apt/sources.list
deb http://raspbian.raspberrypi.org/raspbian/ buster main contrib non-free rpi
->
deb http://ftp.kaist.ac.kr/raspbian/raspbian/ buster main contrib non-free rpi
안될경우 위의 주소로 진행
2.2 업그레이드
$ sudo apt-get update
$ sudo apt-get upgrade -> 추천하지않음
```

3. OpenCV에 필요한 패키지 다운

```
3.1 build-essential 패키지에는 C/C++ 컴파일러와 관련 라이브러리, make 같은 도구들이 포함되어 있습니다.
cmake는 컴파일 옵션이나 빌드된 라이브러리에 포함시킬 OpenCV 모듈 설정등을 위해 필요합니다. 
$ sudo apt-get install build-essential cmake
3.2 특정 포맷의 이미지 파일을 불러오거나 기록하기 위한 패키지
$ sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev
3.3 특정 코덱의 비디오 파일을 읽어오거나 기록하기 위한 패키지
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libxvidcore-dev libx264-dev libxine2-dev
3.4 리눅스에서 실시간 비디오 캡처를 지원하기 위한 패키지
$ sudo apt-get install libv4l-dev v4l-utils
3.5 비디오 스트리밍을 위한 라이브러리
$ sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev 
3.6 자체적으로 윈도우를 생성하여 이미지나 비디오를 보여줄 수 있음.
$ sudo apt-get install libgtk2.0-dev
그 외 선택 가능한 패키지
libgtk-3-dev
libqt4-dev
libqt5-dev
3.7 OpenGL을 지원하기 위한 라이브러리
$ sudo apt-get install mesa-utils libgl1-mesa-dri libgtkgl2.0-dev libgtkglext1-dev   
3.8 OpenCV를 최적화 하기 위한 라이브러리
$ sudo apt-get install libatlas-base-dev gfortran libeigen3-dev
3.9 OpenCV-Python 바인딩 하기위한 패키지
$ sudo apt-get install python2.7-dev python3-dev python-numpy python3-numpy
```

4. OpenCV 컴파일 및 설치

```
4.1 임시 디렉토리 생성
$ mkdir opencv
$ cd opencv
4.2 소스코드 다운 및 압축 해제
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.1.2.zip
$ unzip opencv.zip
4.3 opencv_contrib(extra modules) 다운 및 압축 해제
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.1.2.zip
$ unzip opencv_contrib.zip
4.4 디렉토리 생성 확인
$ ls -d */
opencv-version   opencv_contrib-version
4.5 컴파일 진행
$ cd opencv-version
$ mkdir build
$ cd build

$cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D WITH_TBB=OFF \
-D WITH_IPP=OFF \
-D WITH_1394=OFF \
-D BUILD_WITH_DEBUG_INFO=OFF \
-D BUILD_DOCS=OFF \
-D INSTALL_C_EXAMPLES=ON \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D BUILD_EXAMPLES=OFF \
-D BUILD_TESTS=OFF \
-D BUILD_PERF_TESTS=OFF \
-D ENABLE_NEON=ON \
-D ENABLE_VFPV3=ON \
-D WITH_QT=OFF \
-D WITH_GTK=ON \
-D WITH_OPENGL=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.1.2/modules \
-D WITH_V4L=ON \
-D WITH_FFMPEG=ON \
-D WITH_XINE=ON \
-D ENABLE_PRECOMPILED_HEADERS=OFF \
-D BUILD_NEW_PYTHON_SUPPORT=ON \
-D OPENCV_GENERATE_PKGCONFIG=ON ../
# 인터넷 연결 상태 확인

다음과 같은 메세지 출력시 정상
-- Configuring done
-- Generating done
-- Build files have been written to: /home/pi/opencv/opencv-4.1.2/build

$ make -j4
4.6 설치
$ sudo make install
4.7 opencv 라이브러리를 찾을 수 있도록 설정
$ sudo ldconfig
```

