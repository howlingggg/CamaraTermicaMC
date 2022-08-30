# Camara Termica MC

![CMake](https://github.com/OpenThermal/libseek-thermal/workflows/CMake/badge.svg?branch=master)

## Descripcion

Camara termica Seek con Raspberry pi zero

## Build

Dependencies:
* cmake
* libopencv-dev (>= 2.4)
* libusb-1.0-0-dev

NOTE: se puede hacer 'apt-get install' de las librerias anteriores

```
cd libseek-thermal
mkdir build
cd build
cmake ..
make
```

Instalar:

```
sudo make install
sudo ldconfig       # update linker runtime bindings

```
cmake-gui ../
```

## Getting USB access

Es necesario agregar un "udev rule" para poder correr los codigos sin ser usuario root:

Udev rule:

```
SUBSYSTEM=="usb", ATTRS{idVendor}=="289d", ATTRS{idProduct}=="XXXX", MODE="0666", GROUP="users"
```

Replace 'XXXX' with:
* 0010: Seek Thermal Compact/CompactXR
* 0011: Seek Thermal CompactPRO

o hacerlo manualmete haciendo chmod despues te conectar la camara:

```
sudo chmod 666 /dev/bus/usb/00x/00x
```

with '00x' the usb bus found with the lsusb command

## Running example binaries

```
./examples/seek_test       # Minimal Thermal Compact/CompactXR example
./examples/seek_test_pro   # Minimal Thermal CompactPRO example
./examples/seek_viewer     # Example with more features supporting all cameras, run with --help for command line options
./examples/seek_snapshot   # Takes still images, run with --help for command line options
```

Or if you installed the library you can run from any location:

```
seek_test
seek_test_pro
seek_viewer

seek_viewerMC

```

### seek_viewer
seek_viewer is bare bones UI for the seek thermal devices. It can display video on screen, record it to a file, or stream it to a v4l2 loopback device for integration with image processing pipelines. It supports image rotation, scaling, and color mapping using any of the OpenCV color maps. While running `f` will set the display output full screen and `s` will freezeframe.

```
seek_viewer --camtype=seekpro --colormap=11 --rotate=0                          # view color mapped thermal video
seek_viewer --camtype=seekpro --colormap=11 --mode=file --output=seek.avi       # record color mapped thermal video
seek_viewer --camtype=seekpro --colormap=11 --mode=v4l2 --output=/dev/video0    # stream the thermal video to v4l2 device
```

### seek_snapshot
seek_snapshot takes still images. This is useful for intergrating into shell scripts. It supports rotation and color mapping in the same manner as seek_viewer. Run with --help for all options.


## Linking the library to another program

After you installed the library you can compile your own programs/libs with:
```
g++ my_program.cpp -o my_program -lseek `pkg-config opencv --libs`
```

Using the following include:
```
#include <seek/seek.h>
```

## Apply additional flat field calibration

To get better image quality, you can optionally apply an additional flat-field calibration.
This will cancel out the 'white glow' in the corners and reduces spacial noise.
The disadvantage is that this calibration is temperature sensitive and should only be applied
when the camera has warmed up. Note that you might need to redo the procedure over time. Result of calibration on the Thermal Compact pro:

Without additional flat field calibration | With additional flat field calibration
------------------------------------------|---------------------------------------
![Alt text](/doc/not_ffc_calibrated.png?raw=true "Without additional flat field calibration") | ![Alt text](/doc/ffc_calibrated.png?raw=true "With additional flat field calibration")

Procedure:
1) Cover the lens of your camera with an object of uniform temperature
2) Run:
```
# when using the Seek Thermal compact
seek_create_flat_field -tseek

# When using the Seek Thermal compact pro
seek_create_flat_field -tseekpro
```
The program will run for a few seconds and produces a flat_field.png file.

3) Provide the produced .png file to one of the test programs:

```
# when using the Seek Thermal compact
seek_test flat_field.png
seek_viewer -t seek -F flat_field.png

# When using the Seek Thermal compact pro
seek_test_pro flat_field.png
seek_viewer -t seekpro -F flat_field.png
```
