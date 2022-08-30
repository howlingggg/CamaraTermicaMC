# Camara Termica MC

![CMake](https://github.com/OpenThermal/libseek-thermal/workflows/CMake/badge.svg?branch=master)

## Descripcion

Camara termica Seek con Raspberry pi zero

## Build

Dependencias:
* cmake
* libopencv-dev (>= 2.4)
* libusb-1.0-0-dev

NOTE: Se puede hacer 'apt-get install' de las librerias anteriores

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
```
cmake-gui ../
```

## Getting USB access

Es necesario agregar un "udev rule" para poder correr los codigos sin ser usuario root:

Udev rule:

```
SUBSYSTEM=="usb", ATTRS{idVendor}=="289d", ATTRS{idProduct}=="XXXX", MODE="0666", GROUP="users"
```

Reemplazar 'XXXX' con:
* 0010: Seek Thermal Compact/CompactXR
* 0011: Seek Thermal CompactPRO

o hacerlo manualmete haciendo chmod despues te conectar la camara:

```
sudo chmod 666 /dev/bus/usb/00x/00x
```

with '00x' the usb bus found with the lsusb command

## Codigos de ejemplo

```
./examples/seek_test       # Minimal Thermal Compact/CompactXR example
./examples/seek_test_pro   # Minimal Thermal CompactPRO example
./examples/seek_viewer     # Ejemplo que soporta todas las camaras, correr --help para ver lista de comandos
./examples/seek_snapshot   # Tomar imagenes
```

Si se instalaron las librerias puede ejecutarse desde cualquier direccion usando: 

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

## Calibracion Adicional
Para obtener mejor calidad de imagen, usted puede opcionalmente aplicar una calibracion adicional.
Esto evitará que las imagenes tengan "ruido", logrando asi una imagen mas concisa y precisa.

La desventaja es que esta calibración es sensible a la temperatura y solo debe aplicarse
cuando la cámara se haya calentado. Tenga en cuenta que es posible que deba volver a realizar el procedimiento con el tiempo.
Resultado de la calibración en el Thermal Compact pro:

Sin la calibracion adicional | Con la Calibracion Adicional:
------------------------------------------|---------------------------------------
![Alt text](/doc/not_ffc_calibrated.png?raw=true "Con la Calibracion Adicional:") | ![Alt text](/doc/ffc_calibrated.png?raw=true "Sin la calibracion adicional")

Procedure:
1) Cubra el lente de su camara (Seek thermal o Seek thermal pro) con un objeto con temperatura uniforme.
2) Ejecute:
```
# Usando la camara Seek Thermal compact:
seek_create_flat_field -tseek

# Usando la camara Seek Thermal compact pro:
seek_create_flat_field -tseekpro

```
