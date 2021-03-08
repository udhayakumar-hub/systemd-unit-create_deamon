# systemd-unit-create_deamon (READ FILE using RAW)

here  sample file for  create a deamon process using systemd unit method
this is basic unit service method

Automatically Running a Program - systemd

Most low-level programs need to run automatically when the system starts. There are quite a few different ways of doing this, but for a modern Linux-based system the best way of doing the job is to use systemd. It is slightly more complicated than some alternatives but it has many additional features. Systemd is often referred to as a replacement for the original Unix System V init and it will read and work with sysvinit configuration files.

You will also find Linux systems that use alternatives to systemd - Upstart, runit, launchd and more. The only commonly encountered alternative running on smaller devices, such as within the mbed operating system, is BusyBox init, which is much simpler. Anything running full Linux is likely to support systemd.

The basic object in systemd is a unit - something to be run when the system starts. There are a number of different types of unit but in most cases you will be interested in a service unit. A unit is defined by its configuration file, called a unit file. System supplied unit files are generally stored in /lib/systemd/system and any new unit files you create should be added to /etc/systemd/system which take priority over any unit files in the /lib directory with the same name.

A unit file is a description of the program that you want systemd to run and how to do the job. It has the same name as the service you want to run and ends in .system. Typically there are three sections:

[Unit] contains information not specifically related to the type of
the unit, such as the service description.

[Service] contains information about the specific type of the unit.

[Install] contains information about the installation of the unit.

Let's suppose you have a compiled C program, myservice, in a home directory /home/pi which simply prints a message every five seconds:

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int main(int argc, char** argv) {
    while (1) {
        printf("Hello systemd world \n");
        fflush(NULL);
        sleep(5);
    };
    return (EXIT_SUCCESS);
}

To run this at startup you need to create it a unit file:

[Unit]
Description=My Hello Service
[Service]
Type=simple
ExecStart=/home/pi/myservice
Restart=on-failure
RestartSec=10
KillMode=process
[Install]
WantedBy=multi-user.target

This should be created in /etc/systemd/system and be called myService.service and you will need root privileges to do this.

Let's look at what the different parts of the unit file do.

The first section just contains a description:

[Unit]
Description=My Hello Service

Description is used by systemd commands to refer to your service.

The second section defines how your program should be run:

[Service]
Type=simple
ExecStart=/home/pi/myservice
Restart=on-failure
RestartSec=10
KillMode=process

Type sets the basic behavior of the program. The default is simple which is just a program that can be run from the command line. The ExecStart parameter is the command line instruction that starts your program running. In this case it is just the location of the executable, but in other cases it might start an interpreter and run a script. The Restart parameter tells systemd when to restart your program and RestartSec tells it how long to wait before doing so. Finally KillMode specifies how to stop your program; in this case by killing the process that is running your program.

The final section tells systemd when to start your program running:

[Install]
WantedBy=multi-user.target

this specifies which other units or targets depend on your program. In this case multi-user.target is used to specify that your program is to be started when the system reaches what used to be called runlevel2, i.e. the system is loaded enough to run multi-user consoles, but not a GUI.
===========================================================================================================================================================
Other runlevel targets are:

Run Level:

0---------> runlevel0.target, poweroff.target

Shut down and power off

1-------->runlevel1.target, rescue.target
	
Set up a r+escue shell

2,3,4---->runlevel[234].target, multi-user.target
	
Set up a non-gfx multi-user shell

5------>  runlevel5.target, graphical.target
	
Set up a gfx multi-user shell

6----->   runlevel6.target, reboot.target
	
Shut down and reboot the system


It is also worth knowing that systemd is the first program to run after the kernel has been loaded and the file system has been set up. Notice that systemd also supports mount and device units which will perform what you might think were low level configuration after the kernel has loaded.

WantedBy says that the target needs the program started, but nothing will happen if it doesn't start. You can also use Required by to state a hard dependence where the target will shut down if the unit doesn't start.

There are many more options that can be used to specify dependencies and the order that units should be started in or restarted. The unit file listed above is a simple example to get you started on using systemd.

===============================================================================================================================================================
Of course to use any of these you add the command to systemctl, i.e.

sudo systemctl command

and the name of the unit where required.

So assuming that you have the program file in the home directory and the unit file given earlier you can set things up so that it is loaded at boot time using:

sudo systemctl daemon-reload
sudo systemctl enable myService

and to run it without a reboot:

sudo systemctl start myService
===========================================================================================================================================================
At this point you might be wondering what permissions a service has?
The answer is that by default a service runs as root and has full root permissions. This sounds like a security problem, but only a user with root access can create a unit file or use systemctl. If everything is correctly constructed there is little security problem with a service running as root. However, many services run as their own user and in their own group. You can change the default user and groups by adding to the [Service] section:

User=username
Group=groupname

If your service only needs a subset of root’s permissions then setting up a user and group just for it is a good way to allow for customization and sharing of resources.
Journal

To find out the current status of your program you can use the status command which also provides a short extract from its log. Although many of the legacy Linux log files are still supported, systemd is supposed to be used by programs to log events. The journal is a central store for all messages logged by the system and all units. It is a binary file and you need to use the journalctl command to read it. There are many forms of the command and it is worth finding out how to filter the logs for what you are looking for. To see the log output from myService all we need is:

sudo journalctl -f -a -umyService

==============================================================================================================================================================
