# NVIDIA Jetson nano L4T 32.2.1/32.3.1 Kernel modifications

This project aims to describe how to modify the L4T kernel in order to change the Sony IMX219 settings.

## How to modify the imx219 driver

There are two important files that need to be modified:
1. The imx219 mode settings - [imx219_mode_tbls.h](imx219-kernel-modifications/nvidia/driverrs/media/i2c/imx219_mode_tbls.h)
2. The device tree - [tegra210-camera-rbpcv2-imx219.dtsi](imx219-kernel-modifications/nvidia/platform/t210/porg/kernel-dts/porg-platforms/tegra210-camera-rbpcv2-imx219.dtsi)


### The imx219 mode settings - [imx219_mode_tbls.h](imx219-kernel-modifications/nvidia/driverrs/media/i2c/imx219_mode_tbls.h)

This file contains the settings of the imx219 camera sensors registry, that can be modified according to the sensors [datasheet](https://publiclab.org/system/images/photos/000/023/294/original/RASPBERRY_PI_CAMERA_V2_DATASHEET_IMX219PQH5_7.0.0_Datasheet_XXX.PDF).

### The device tree - [tegra210-camera-rbpcv2-imx219.dtsi](imx219-kernel-modifications/nvidia/platform/t210/porg/kernel-dts/porg-platforms/tegra210-camera-rbpcv2-imx219.dtsi)

The device tree contains camera mode settings needed by the OS.


## Important notes

Check the following [datasheet](https://publiclab.org/system/images/photos/000/023/294/original/RASPBERRY_PI_CAMERA_V2_DATASHEET_IMX219PQH5_7.0.0_Datasheet_XXX.PDF) for guidance on how to set the different registers in order to configure the image sensor.


1. line blanking needs to be larger than `0xA8`
2. line length should be between `0x0D78` and `0x7FF0`
3. frame blanking needs to be larger than `0x20`
4. frame Length should be between `0x0100` and `0xFFFE`
5. The image stream should not exceed the bandwidth of the 2-lane CSI-2 connection. Therefore `Width*Height*FPS < 2 * Pixelclock`.

### How to calculate frame and line length with a given image size and desired FPS?

`Frame Length = 2*Pixelclock/(LineLength * FPS)`

## Our new settings

##### We managed to implement the following settings:

| Sensor Mode  | Width x Height | FPS |
| ------------- | ------------- | ----|
| 0 | 3280x1560 | 40 |
| 5 | 1640x1232 | 40 |

##### Furthermore, we needed a couple of special frame dimensions, therefore we added theses settings:

| Sensor Mode  | Width x Height | FPS |
| ------------- | ------------- | ----|
| 1 | 3280x2464 | 40 |
| 2 | 1640x780 | 40 |
| 3 | 1640x780 | 35 |
| 4 | 1640x780 | 30 |


## Integration into the yocto build pipeline
We integrated the [0001-added-apic-cameramods.patch](patches/0001-added-apic-cameramods.patch) into our yocto image build pipeline in order to apply these changes to our build image.

If you are using the build pipeline for the [BalenaOS](https://github.com/balena-os/balena-jetson) you will need to add the patch into the `layers/meta-balena-jetson/recipes-kernel/linux/linux-tegra/` folder and append the `layers/meta-balena-jetson/recipes-kernel/linux/linux-tegra_%.bbappend` accordingly.

## Authors

The NVIDIA CORPORATION is the initial author of [imx219_mode_tbls.h](imx219-kernel-modifications/nvidia/driverrs/media/i2c/imx219_mode_tbls.h) and [tegra210-camera-rbpcv2-imx219.dtsi](imx219-kernel-modifications/nvidia/platform/t210/porg/kernel-dts/porg-platforms/tegra210-camera-rbpcv2-imx219.dtsi).
We just modified it a bit to suite our requirements.
