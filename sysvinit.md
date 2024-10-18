# SysVinit
[sysvinit](https://wiki.gentoo.org/wiki/Sysvinit) is an init system. Scripts installed in `/etc/init.d/` will handle service startup, stop, restarting, and possibly more when linked to the appropriate runlevels using the `update-rc.d` command. Yocto handles linking sysvinit scripts with an `update-rc.d` class, so that cross compilation is handled correctly.

## Init Scripts
Init scripts are shell scripts that run on system boot and handle startup, shutdown, and restart of services.

## Example Script
```sh
#!/bin/sh
### BEGIN INIT INFO
# Provides:         hello_world
# Required-Start:   $all
# Required-Stop:
# Default-Start:    2 3 4 5
# Default-Stop:     0 1 6
### END INIT INFO

case "$1" in
    start)
        echo "Running hello_world on boot..."
        # assuming the binary exists. `&` runs in background so script continues
        /usr/bin/hello_world &
        ;;
    stop)
        echo "Stopping hello_world..."
        # do nothing
        ;;
    restart)
        echo "Restarting hello_world..."
        # do nothing
        ;;
    *)
        # no other arguments are supported. exit with error code 1
        exit 1
        ;;
esac

exit 0
```
#### Script Explanation
Explanation of the script metadata:
- `Provides: hello_world`: indicates the script provides a service called `hello_world`. Important for the system to recognize this script as a startup service.
- `Required-Start: $all`: specifies that the script should start after all other services have started.
- `Required-Stop`: empty, it doesn't depend on any other service when stopping.
- `Default-Start: 2 3 4 5`: refers to the [runlevels](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/4/html/reference_guide/s1-boot-init-shutdown-sysv#s1-boot-init-shutdown-sysv) at which the script should start. Only one runlevel is active at any given time.
- `Default-Stop: 0 1 6`: refers to the runlevels at which the script should stop.

sysvinit passes an argument to an init script when it runs it. The `case "$1" in` code block handles each of the scenarios when an init script is executed.
- `start)`
- `stop)`
- `restart)`
- `*)`: catch all for unspecified cases.

#### Installation using command line
Installing the above script (named `hello_world_init`) saved in the current directory:
- `install -m 0755 ./hello_world_init /etc/init.d/hello_world_init`
- `update-rc.d hello_world defaults`
- Note that we use the service name provided in the script metadata to update the runlevel defaults.

#### Installation using BitBake
The installation would be handled by the build system as specified in the recipe.
```sh
inherit update-rc.d

INITSCRIPT_NAME = "hello_world_init"
INITSCRIPT_PARAMS = "defaults"

do_install_append() {
    install -d ${D}${sysconfdir}/init.d
    install -m 0755 ${WORKDIR}/hello_world_init ${D}${sysconfdir}/init.d/hello_world_init
}
```
Note that the bitbake recipe specifies the filename rather than the provided service name.
