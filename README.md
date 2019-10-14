# kinectv2-jetson-nano
A step by step guide to install Kinect v2 libraries (libfreenect2 and pylibfreenect2) on Jetson Nano.

## 1) Install OpenCV 4.1.1

The current OS image comes with OpenCV 3.1.1. Use installOpenCV.sh script to upgrade the OpenCV version to 4.1.1. Before you run the script, please update username and password information in the script. In your home directory, execute the following:

```bash
mkdir opencv
chmod +x ./installOpenCV.sh
./installOpenCV.sh /home/opencv
```

CMake may raise the following error in the attempt. If this does not happen, continue with the second step.

```bash
[ 14%] Building CXX object modules/core/CMakeFiles/opencv_test_core_pch_dephelp.dir/opencv_test_core_pch_dephelp.cxx.o
In file included from /home/nano/opencv/modules/core/test/test_precomp.hpp:12,
                 from /home/nano/opencv/build/modules/core/opencv_test_core_pch_dephelp.cxx:1:
/home/nano/opencv/modules/core/include/opencv2/core/private.hpp:66:12: fatal error: Eigen/Core: No such file or directory
 #  include <Eigen/Core>
```

Fix this linking issue by editing the file. Here my username is nano, change with yours.

```bash
sudo nano /home/nano/opencv/modules/core/include/opencv2/core/private.hpp
```

Then replace the following line

```bash
# include <Eigen/Core>
```
with this

```bash
# include <eigen3/Eigen/Core>
```

We are ready to build OpenCV. So first, clean the build directory.

```bash

sudo rm /home/nano/opencv/release

```

Then rerun the following in /home/<your-username>/opencv directory:

```bash
mkdir release
cd release/
cmake -D WITH_CUDA=ON -D CUDA_ARCH_BIN="5.3" -D CUDA_ARCH_PTX="" -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.1.1/modules -D WITH_GSTREAMER=ON -D WITH_LIBV4L=ON -D BUILD_opencv_python2=ON -D BUILD_opencv_python3=ON -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_EXAMPLES=OFF -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
make -j4
sudo make install
```

## 2) Install libfreenect2

In your home directory, download and run this script: installLibfreenect2.sh. That should be straightforward.

```bash
chmod +x ./installLibfreenect2.sh
./installLibfreenect2.sh
```

Connect your Kinect v2 to Jetson Nano. To test your build, execute this command:

```bash
sudo ./bin/Protonect
```

You should be able to see the depth, IR and RGB streams. Continue with the next step to install Python bindings.

## 3) Install pylibfreenect2

Return to your home directory. Run

```bash
sudo nano ~/.bashrc
```
Then, paste these lines to the end of this file.

```bash
export LIBFREENECT2_INSTALL_PREFIX=/home/nano/libfreenect2/
export LD_LIBRARY_PATH=/home/nano/freenect2/lib:$LD_LIBRARY_PATH
```

To enable this environment variable definitions, run:

```bash
sudo source ~/.bashrc
```
Then, copy "config.h" and "export.h" from /home/nano/libfreenect2/build/libfreenect2/ and install them in: /home/nano/libfreenect2/include/libfeenect2/

Finally, copy "lib" folder from :/home/nano/libfreenect2/build/ and put in: /home/nano/libfreenect2/

We are ready to install pylibfreenect2. Execute the following:

```bash
cd /home/nano
git clone https://github.com/r9y9/pylibfreenect2.git
sudo -E python3 pylibfreenect2/setup.py
```

Then test the installation by running:
```bash
python3 /home/nano/pylibfreenect2/examples/multiframe_listener.py 
```

##Additional step: Diasble Autosuspend Feature

The Jetson Nano comes with the feature of USB autosuspend ehich may cause errors with the Kinect V2 in a long run. To disable, execute this:


```bash
sudo sh -c 'for dev in /sys/bus/usb/devices/*/power/autosuspend; do echo -1 >$dev; done'```
```



