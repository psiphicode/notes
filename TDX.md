# Intel Trusted Domain eXtensions
Intel TDX is a confidential computing environment, similar to SGX (which offers application isolation) but more general purpose (TDX offers isolation at the VM, container or application level).

[Read Microsoft's announcement of partnering Azure with Intel](https://azure.microsoft.com/en-us/blog/azure-confidential-computing-on-4th-gen-intel-xeon-scalable-processors-with-intel-tdx/)

# Yocto Project
Yocto Project enables building custom Linux VM images with configurable kernel features. It's useful for creating small images for embedded systems. In this case, it's used to build a Linux VM to be run inside of a TDX VM.
