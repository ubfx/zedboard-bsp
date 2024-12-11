# ZedBoard PetaLinux BSPs for Vivado 2024.2+
New PetaLinux is nice because it runs on RHEL and it has a newer kernel. Unfortunately, there is no official ZedBoard BSP for the newer PetaLinux, so we have to make one ourselves.

You can either download the BSP from the releases and directly create a petalinux-project with it...
```bash

```
... and continue with step [Build](#Build) or step [Flash](#Flash) if you want to use the pre-built image contained in the release.

Or you can manually create the BSP as described in the following section.

## Creating a BSP for current Vivado

Obviously, you first have to download and install the current PetaLinux version, which we don't describe here.

Then, download the most recent official ZedBoard BSP. They can be found by going to the PetaLinux download archive on the Xilinx/AMD Website, where for every PetaLinux version there are some BSPs for different Boards to download. However, the most recent PetaLinux that lists a ZedBoard BSP seems to be 2021.1.
[Download](https://www.xilinx.com/member/forms/download/xef.html?filename=avnet-digilent-zedboard-v2021.1-final.bsp)

The .bsp is really just a tar archive, so instead of using it with PetaLinux directly, we can unpack it to some folder as a reference.

```
tar xf avnet-digilent-zedboard-v2021.1-final.bsp
```

First step is now to open the Vivado project contained in that bsp/archive with the Vivado version matching PetaLinux version, and updating the project to that version. There might be a message about IP versions being out of date, also update those. Then save the project under a new name (I chose zedboard-2024.2) and generate bitstream.

The blockdiagram can remain unchanged from the original BSP, but of course you can also make project-dependent changes.

![Screenshot](screenshots/blockdiagram.png)

Then export the Hardware by going to File->Export->Hardware and export it including the bitstream

![Screenshot](screenshots/export_hw.png)

Next we will use the new SDT flow to generate device tree data from the xsa file we just exported. For that, run the `xsct` tool from command line (this tool comes with the Vivado/Vitis installation). After the `-xsa`, put the path to the xsa file that you just exported from Vivado.
```
$ xsct                                                                                                                                                                                                                                    ─╯                                                                                                                                                                                                       

xsct% sdtgen set_dt_param -dir zed2024_out_sdt -xsa zedboard_2024.2/system.xsa -board_dts zedboard                                                                                     
xsct% sdtgen generate_sdt  
```

This should have generated a folder `zed2024_out_sdt` which we can now use as Hardware description for PetaLinux. You might have to adapt the path to the generated sdt in the following commands. The folder contents shoud look something like this:

![Screenshot](screenshots/sdt_outputs.png)

Now we can create the PetaLinux project.

```bash
$ petalinux-create project  --template zynq -n zedboard-2024.2
$ cd zedboard-2024.2/
$ petalinux-config --get-hw-description zed2024_out_sdt --silentconfig
```

You can now make changes to the overall PetaLinux configuration or rootfs (i.e. included packages etc), but you can also leave everything at default. Note: Don't copy the configuration files from the old BSP because the available settings and options don't match perfectly. If you really care, you can diff `config` and `rootfs_config` files between the newly generated project and the old BSP.
```bash
$ petalinux-config #optional
$ petalinux-config -c rootfs #optional
```

Now copy these files from the old BSP to the new petalinux project
* project-spec/meta-user/recipes-kernel/linux/linux-xlnx/bsp.cfg
* project-spec/meta-user/recipes-bsp/device-tree/files/*.dtsi

That's it, you should now be able to build the project and create the boot image.

## Build
If you downloaded the BSP from releases, this step can be skipped
```bash
$ petalinux-build
```

## Create boot image
```bash
$ petalinux-package boot --plm --psmfw --u-boot --dtb --fsbl --fpga  --force
[INFO] Getting Default bit file
[INFO] File in BOOT BIN: "zedboard-2024.2/images/linux/zynq_fsbl.elf"
[INFO] File in BOOT BIN: "zedboard-2024.2/project-spec/hw-description/system.bit"
[INFO] File in BOOT BIN: "zedboard-2024.2/images/linux/u-boot.elf"
[INFO] File in BOOT BIN: "zedboard-2024.2/images/linux/system.dtb"
[INFO] Generating zynq binary package BOOT.BIN...
[INFO] 

****** Bootgen v2024.2
  **** Build date : Oct 21 2024-10:58:34
    ** Copyright 1986-2022 Xilinx, Inc. All Rights Reserved.
    ** Copyright 2022-2024 Advanced Micro Devices, Inc. All Rights Reserved.


[INFO]   : Bootimage generated successfully
```

## Flash
...
