Shrek and Donkey travel through farmland, stealing vegetables while they journey to reclaim their homelands. Shrek wisely offers useful information to Donkey.

> Shrek: For your information, there's a lot more to Yocto images than people think.
>
> Donkey: Example?
>
> Shrek: Example? \[*thinking*\] Okay, ummm...
>
> Shrek: Yocto images... are... like onions!
>
> Donkey: \[*sniffs onions then winces*\] They stink?
>
> Shrek: Yes--no!
>
> Donkey: Or they make you cry?
>
> Shrek: Nooo!!
>
> Donkey: Ohhhh, you leave 'em out in the sun, they get all brown, start sproutin white little heads?
>
> Shrek: \[*exasperated*\] NO!
>
> Shrek: **LAYERS!** *Yocto images have layers!*

![shrek and donkey discuss layers](./img/layers.png "Shrek and Donkey look at an onion, which represents a Yocto image")

# Overview
The purpose of this document is to synthesize information about developing custom Linux VMs using Yocto.

[Yocto Project](https://docs.yoctoproject.org/) is an open source project focusing on embedded Linux systems. Yocto Project uses a build host based on the [OpenEmbedded](https://www.openembedded.org/wiki/Main_Page) project, which uses the [Bitbake](https://docs.yoctoproject.org/bitbake/bitbake-user-manual/bitbake-user-manual-intro.html) tool.


## Yocto Docs Prerequisites
- [What I wish I'd known about Yocto Project](https://docs.yoctoproject.org/what-i-wish-id-known.html)
- [Technical Overview](https://www.yoctoproject.org/development/technical-overview/)

## Four Kings
- **Yocto Project**s are comprised of layers, recipes, and configuration files. Layers are hierarchical. They can override the configuration and settings applied in earlier layers in a build.
- **OpenEmbedded-Core** (`oe-core`) is a continuously validated, tightly controlled, and quality assured core set of recipes.
- **Poky** is the embedded reference distribution that provides a base level distro to build images.
- **Bitbake** is the build engine for Yocto images.

## BitBake Breakdown
This section comes from asking ChatGPT questions about Yocto and looking at the `core-image-minimal` package that ships with Yocto.

#### Layers
- Yocto uses the concept of layers to organize recipes and configurations. The build environment includes a `conf/bblayers.conf` file, which lists all the layers (directories) that BitBake should search for recipes and configuration files.
- These layers contain the metadata, recipes and configuration files that BitBake uses during the build process.

#### Recipe Files
- In each layer, recipes are defined as `.bb` (BitBake) files. For example, the recipe for `core-image-minimal` is found in `poky/meta/recipes-core/images/core-image-minimal.bb`.
- The name of the recipe file corresponds to the target name you use with BitBake. When you run `bitbake core-image-minimal`, BitBake looks for a recipe with that name in the directories listed in `conf/bblayers.conf`.
- BitBake parses the recipe files based on the `BBFILES` variables in the global configuration and in layer configuration files.

#### Configuration Files
The global configuration file for a build environment is the `bblayers.conf` file. Below is the configuration file for the `core-image-minimal` build environment:
```
# file: bblayers.conf

BBPATH = "${TOPDIR}"
BBFILES ?= ""

BBLAYERS ?= " \
  /path/to/project_dir/poky/meta \
  /path/to/project_dir/poky/meta-poky \
  /path/to/project_dir/poky/meta-yocto-bsp \
  "
```
- `BBFILES` is used for pattern matching recipe files.
- `BBLAYERS` is used for identifying layers in the build environment.

Layers may have their own configuration file (`layer.conf`). BitBake will use this to pattern match recipes within that layer. For example, below is the configuration file for the `core-image-minimal` layer in `poky/meta/conf/layer.conf`.
```
# file: layer.conf

BBPATH .= ":${LAYERDIR}"
BBFILES += "${LAYERDIR}/recipes-*/*/*.bb"
```

Here's the recipe file itself:
```bb
# file: poky/meta/recipes-core/images/core-image-minimal.bb

SUMMARY = "A small image just capable of allowing a device to boot."

IMAGE_INSTALL = "packagegroup-core-boot ${CORE_IMAGE_EXTRA_INSTALL}"

IMAGE_LINGUAS = " "

LICENSE = "MIT"

inherit core-image

IMAGE_ROOTFS_SIZE ?= "8192"
IMAGE_ROOTFS_EXTRA_SPACE:append = "${@bb.utils.contains("DISTRO_FEATURES", "systemd", " + 4096", "", d)}"
```
