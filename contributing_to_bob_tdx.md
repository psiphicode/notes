# Overview
These are my general notes on the TDX/BoB project. This document mainly focuses on the strucure, management, and development of yocto images.

## Tasks
There are some [external tasks](https://flashbots.notion.site/BoB-external-tasks-1126b4a0d876804f924dea7b8ce1cfbc) to implement various features in the searcher VM:
1. Dynamic firewall rules
   - use podman (preferred)
   - nftables or iptables
2. Delayed log shipping
   - fluentbit (required)
   - use lua script (preferred)
   - destination endpoint authentication (required)
3. Maintenance requests
   - public REST API
   - authentication with ssh keys
   - iptables or nftables

## Build environment
I'm new to yocto projects so I'm making up some terminology here:
- A yocto image is a custom Linux VM image.
- A bitbake layer is a component of a yocto image.
  - Each layer is typically in its own github repository.
- A build environment is all of the source code that comprises a yocto image, including its set of bitblake layers.
  - A build environment consists of source code from multiple github repositories.

## Managing build environments
Build environments are managed by [repo](https://gerrit.googlesource.com/git-repo/), a multiple git repository tool. There are two top-level repositories for managing build environments:
1. [flashbots/yocto-manifests](https://github.com/flashbots/yocto-manifests)
2. [flashbots/yocto-scripts](https://github.com/flashbots/yocto-scripts)

#### 1. yocto-manifests
Contains `<manifest-name>.xml` files that specify the source code repositories for a yocto image. Manifest files (e.g. `tdx-bob.xml`) specify the `yocto-scripts` branch where a setup script is implemented (there must be a corresponding branch in the `yocto-scripts` repo). For example, for branch `bob-tdx`:

```xml
<!-- tdx-bob.xml -->
<project remote="flashbots" revision="bob-tdx" name="yocto-scripts" path="srcs/yocto-scripts">
    <copyfile dest="setup" src="setup"/>
    <copyfile dest="Makefile" src="Makefile"/>
</project>
```

When `repo` initializes the build environment, it will copy files from the `bob-tdx` branch of `yocto-scripts` into the directory where the project is initialized.

Further, the manifest files specify source code locations of the layers that will be used to build the yocto image. For example, here are the layers of the `tdx-bob` yocto image:

```xml
<!-- tdx-bob.xml -->

<project remote="flashbots" revision="scarthgap" name="poky" path="srcs/poky"/>
<project remote="flashbots" revision="scarthgap" name="meta-openembedded" path="srcs/poky/meta-openembedded"/>
<project remote="flashbots" revision="scarthgap" name="meta-secure-core" path="srcs/poky/meta-secure-core"/>
<project remote="flashbots" revision="scarthgap" name="meta-virtualization" path="srcs/poky/meta-virtualization"/>
<project remote="flashbots" revision="main" name="meta-confidential-compute" path="srcs/poky/meta-confidential-compute"/>
<project remote="flashbots" revision="main" name="meta-custom-podman" path="srcs/poky/meta-custom-podman"/>
<project remote="flashbots" revision="master" name="meta-searcher" path="srcs/poky/meta-searcher"/>
```
Each of these layers has its own github repository and branch. Layers of a yocto image need to be stored in the `poky` directory and follow the `meta-<NAME>` convention.

#### 2. yocto-scripts
Contains a setup script and a Makefile for a build environment. The setup script for a build environment adds the bitbake layers and applies patches to the `local.conf` build config. For example, here is a snippet of a setup script that shows how the layers are added:

```sh
# Add the necessary layers to bblayers.conf
bitbake-layers add-layer ../meta-confidential-compute
bitbake-layers add-layer ../meta-openembedded/meta-oe
bitbake-layers add-layer ../meta-openembedded/meta-python
bitbake-layers add-layer ../meta-secure-core/meta-tpm2
bitbake-layers add-layer ../meta-openembedded/meta-networking
bitbake-layers add-layer ../meta-openembedded/meta-filesystems
bitbake-layers add-layer ../meta-virtualization
bitbake-layers add-layer ../meta-custom-podman
bitbake-layers add-layer ../meta-searcher
```

## Setting up and building an image
This section details how I built the `bob-tdx` image, and it's mostly similar to the steps in the [README](https://github.com/flashbots/yocto-manifests/blob/main/README.md) of the `yocto-manifests` repository.

#### 1. Initialize
Create a working directory on the development machine:
```sh
mkdir -p yocto/tdx
cd yocto/tdx
```
Initialize the build environment using `repo` by specifying the manifest repository (`-u` option), the branch (`-b` option), and the manifest name (`-m` option).
```sh
repo init -u https://github.com/flashbots/yocto-manifests.git -b main -m tdx-bob.xml
repo sync
```
This downloads all of the source code repositories needed in the image's build environment.

#### 2. Setup
Use the setup script:
```sh
source setup
```
This initializes the poky build environment and adds the bitblake layers to the project config. It also changes the directory into the `./build` directory where the build command will be run.

Next, I changed the machine type in `srcs/poky/build/conf/local.conf` from `tdx` to `qemux86-64`, so that I can build it to be run locally. Note that I did this step after using `source setup`, because the setup script patches the `local.conf` file to build for the `tdx` machine.

```
# srcs/poky/build/conf/local.conf

# I commented out the below line:
# MACHINE ?= "tdx"

# then I added this line:
MACHINE ?= "qemux86-64"
```


#### 3. Build
Build the image:
```sh
bitblake cvm-image-azure
```

#### 4. Run
Now run the image using `runqemu`:
```sh
runqemu cvm-image-azure wic nographic kvm ovmf qemuparams="-m 8G"
```

## Modifying the image
Now that the setup is complete, the image can be rebuilt by sourcing the poky build script and then running the build command again:
```sh
cd srcs/poky
source oe-init-build-env
bitblake cvm-image-azure
```

I must always run `source oe-init-build-env` when I want to run the image using `runqemu` or when I want to rebuild the image using `bitblake`.
