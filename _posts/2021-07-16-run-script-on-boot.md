---
layout: post
title:  "Run a Script on Boot in Android"
author: "Shori"
comments: false
tags: Android
---

The goal is to run a script, or any binary on boot in Android. It is assumed that you are running your own ROM in a VM or on a devboard.

## Add to the init process

The idea is to have the `init` process of Android to call the script, `boot_test.sh`, that we prepared. Then, we should be good to go.

By adding the following lines to `$AOSP/system/core/rootdir/init.rc`, we could achieve the idea.

```
service boot_test /system/bin/boot_test.sh
    oneshot

on property:sys.boot_completed=1
    start boot_test

```

This way, when the system property `sys.boot_completed` turns to 1, the `init` process will invoke the `boot_test` service, which will call the script.

Or alternatively, 
```
service boot_test /system/bin/boot_test.sh
    class main
    oneshot

```
By hinting `main` class in the service, it tells the system to invoke the service with many other `main` services.

> 1. Recompile after altering, as init.rc changes take effect in ramdisk, rather than the root file system.
> 2. The blank line at the end of file is significant.

## SEPolicy

If you are placing the script other than the `/data` directory, you need to implement the SEPolicy. As, after doing the steps above, you may get the following errors in the QEMU boot logs.

```
init: starting service 'boot_test'... 
init: processing action (sys.boot_completed=1 && sys.logbootcomplete=1) from (/system/etc/init/bootstat.rc:70) 
type=1400 audit(1626419331.346:118): avc: denied { ioctl } for pid=1791 comm="RenderThread" path="/dev/goldfish_address_space" dev="tmpfs" ino=5518 ioctlcmd=470c scontext=u:r:system_app:s0 tcontext=u:object_r:device:s0 tclass=chr_file permissive=1 
type=1400 audit(1626419331.546:119): avc: denied { read execute } for pid=2065 comm="boot_test.sh" path="/system/bin/sh" dev="vda1" ino=2304 scontext=u:r:boot_test:s0 tcontext=u:object_r:shell_exec:s0 tclass=file permissive=1 
type=1400 audit(1626419331.546:119): avc: denied { read execute } for pid=2065 comm="boot_test.sh" path="/system/bin/sh" dev="vda1" ino=2304 scontext=u:r:boot_test:s0 tcontext=u:object_r:shell_exec:s0 tclass=file permissive=1 
type=1400 audit(1626419331.546:120): avc: denied { getattr } for pid=2065 comm="boot_test.sh" path="/system/bin/sh" dev="vda1" ino=2304 scontext=u:r:boot_test:s0 tcontext=u:object_r:shell_exec:s0 tclass=file permissive=1 
init: starting service 'exec 3 (/system/bin/bootstat --set_system_boot_reason --record_boot_complete --record_boot_reason --record_time_since_factory_reset -l)'... 
type=1400 audit(1626419331.546:120): avc: denied { getattr } for pid=2065 comm="boot_test.sh" path="/system/bin/sh" dev="vda1" ino=2304 scontext=u:r:boot_test:s0 tcontext=u:object_r:shell_exec:s0 tclass=file permissive=1 
init: processing action (sys.boot_completed=1 && sys.wifitracing.started=0) from (/system/etc/init/wifi-events.rc:20) 
selinux: SELinux: Skipping restorecon_recursive(/data/system_ce/0) 
 
type=1400 audit(1626419331.562:121): avc: denied { ioctl } for pid=1278 comm="surfaceflinger" path="/dev/goldfish_address_space" dev="tmpfs" ino=5518 ioctlcmd=470c scontext=u:r:surfaceflinger:s0 tcontext=u:object_r:device:s0 tclass=chr_file permissive=1 
init: Service 'boot_test' (pid 2065) exited with status 1 
```
The `avc: denied` error signals problems with SEPolicies. We need to create one for our service in `$AOSP/device/generic/goldfish/sepolicy/common`. We create a file called `boot_test.te`.

```
type boot_test, domain, mlstrustedsubject, coredomain;
type boot_test_exec, exec_type, file_type;

init_daemon_domain(boot_test)

allow system_app device:file rwx_file_perms;
allow boot_test shell_exec:file rwx_file_perms;

```
In the `allow` lines, the syntax follows such pattern: `allow $1 $2:$3 $4`, where `$1` is `scontext=`, `$2` is `tcontext=`, `$3` is `tclass=`, and `$4` is the action in the curly braces, eg. `read execute`.

More, `rwx_file_perms` stands for all permissions needed for files, and `create_dir_perms` stands for all permissions needed for directories.

Then, add the following line in `file_context`.
```
/system/bin/boot_test.sh     u:object_r:boot_test_exec:s0


```

Recompile, then the script should be working as fine.