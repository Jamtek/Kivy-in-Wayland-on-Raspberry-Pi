# Kivy-in-Wayland-on-Raspberry-Pi
[MaslowCNC](http://www.maslowcnc.com/) Kivy GroundControl on a Raspberry Pi 3 running the KDE Window Manager, wayland version.

This is a solution, I came up with for running GroundControl or kivy python apps inside a Wayland compositor window while using a mouse or touchpad. If you are using a touchscreen then running KivyPie is probably the safe choice or running kivypie with the trapped cursor mod that was discussed and uploaded by Kingley in this earlier [forum thread](https://forums.maslowcnc.com/t/raspberry-pi-cursor/2019).

[Disclaimer] *As of this writing I have not yet received my Maslow CNC kit, and I have only been able to perform limited testing on a Raspberry Pi 3b with a bluetooth keyboard and touchpad (Logitech K400R).
 Also, be warned you will need to enable the [experimental] GL driver in `raspi-config` and  that the current 'KDE Window manager wayland version' being used is considered a [PREVIEW] release.*

Disclaimer aside, I have been able to run GroundContol in a windowed environment without any screen lag or the missing cursor problem, as encountered when using KivyPie without a touchscreen. 

#### Raspbian Stretch Lite install
Download Raspbian Lite https://www.raspberrypi.org/downloads/raspbian/
Follow instructions per your OS, to install it on an SD card.

I found this guide very useful for installing various desktop environments in the Raspbian Lite distro.
[Raspberry Pi Forums](https://www.raspberrypi.org/forums/viewtopic.php?t=133691#p890408)

Boot up your raspberry pi 3 and run the following commands, if you need to setup wifi you can do so with choosing network options in `sudo raspi-config`
   
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    
Reboot and start `raspi-config` again. Make the appropriate setting changes for your locale, timezone, etc. Choose option 7. Advanced Options from the menu and then select A7.  GL Driver. You will be given 3 options:
* G1 (Full KMS)
* G2 (Fake KMS)
* G3 Legacy

For this build I choose G1 (Full Kernel Module Support). Save and exit out, you will be asked to reboot again.
    

#### xserver, kwin-wayland installation

The following commands will pull in and install a large number of packages and may take some time depending on your internet connection, so be patient.

    sudo apt-get install --no-install-recommends xserver-xorg
    sudo apt-get install --no-install-recommends xinit
    sudo apt-get install kwin-wayland konsole kate

After all the packages are installed, you should be able to start the KDE window manager by running:

`startx`

This will start the KDE Plasma Window Manager running Qt5 libs with Wayland support. Plasma is a full desktop environment, with lots of eye candy. Unfortunately, it also uses a lot of resources, probably not what you want for a CNC controller.

Instead, we will run the KDE wayland compositor without the Plasma desktop environment. Logout of the KDE Plasma desktop and with the the following command at the shell prompt enter the following:
 
`kwin_wayland --xwayland --exit-with-session=konsole`

The screen should turn black with a mouse pointer in the center of the screen followed by konsole the app.
The `--exit-with-session=` option lets us gracefully return to a command prompt when closed, otherwise you will have to open another tty and kill the process. The key thing is that running this way only uses 124 Megs of RAM.

As a test you should be able to run `glxgears` from inside `konsole` and achieve a frame rate of 60 fps.

#### Install Kivy manually

Install the following packages with apt-get and python pip.
[Note] *The pip installs here may be newer versions of whats is in the 'requirements_linux.txt' and which uses kivy version 1.9.1. the latest stable version is 1.10.0 and we will be compiling the latest developer branch from git based off 1.10.1*

    sudo apt-get install libsdl2-dev libsdl2-image-dev libsdl2-mixer-dev libsdl2-ttf-dev pkg-config libgl1-mesa-dev libgles2-mesa-dev python-setuptools libgstreamer1.0-dev git-core xclip
    sudo apt-get install gstreamer1.0-plugins-{bad,base,good,ugly}
    sudo apt-get install gstreamer1.0-{omx,alsa} python-dev libmtdev-dev
    sudo apt-get install python-pip libffi-dev
    sudo apt install libsdl1.2-dev

    sudo pip install -U Cython==0.27.3
    sudo pip install pyserial
    sudo pip install appdirs
    sudo pip install packaging
    sudo pip install pyparsing
    sudo pip install requests
    sudo pip install six
    sudo pip install pywayland
    sudo apt-get install python-pygame
    sudo apt-get install python-numpy
	
Download the latest branch of Kivy from Git Hub.

`git clone https://github.com/kivy/kivy`

`cd kivy`

Build kivy in place, per the instructions on [kivy.org](https://kivy.org/docs/installation/installation-rpi.html) :Note we are adding the USE flags for wayland and X11 when running make.

`USE_WAYLAND=1 USE_X11=1 make`

Compiling kivy can take some time. I did not time it personally, but walked away for 10 or 15 minutes and when  I came back it had completed.
 Run the following commands to add your kivy development folder path to your .profile.

    echo "export PYTHONPATH=$(pwd):\$PYTHONPATH" >> ~/.profile
    source ~/.profile

Test kivy with one of the example apps from inside the KDE window manager.

`kwin_wayland --xwayland --exit-with-session=konsole`
From inside the Konsole terminal emulator, type: 
`cd kivy/examples/demo/pictures`
`python main.py`

If everything is installed properly you should be able to interact with the Pictures app using your mouse or touchpad.

#### Install and run GroundControl
 From your home directory download or clone the GroundControl program.

`git clone https://github.com/MaslowCNC/GroundControl.git`

Add yourself to the dialout group to allow access to the serial port.
    
`sudo usermod -a -G input,dialout pi`

Now start the KDE Window manager.

`kwin_wayland --xwayland --exit-with-session=konsole`

From inside the konsole terminal app:

`cd GroundControl`

`python main.py`

The GroundControl program appears full screen and you should be able to interact with it using your mouse and keyboard. You should also be able to resize the window it runs in!
 ![kwin_gc1|666x500](upload://jPBxG5qchh5eHjniueHd7rhg8hZ.jpg)
 

#### Install Arduino
Install the Java runtime  engine required to run the Arduino IDE.

    sudo apt-get install default-jre default-jre-headless
    
Download the  latest arduino IDE  from https://www.arduino.cc/
extract the files into your home directory.

`tar -xvf arduino-1.8.5-linuxarm.tar.xz`

`cd arduino-1.8.5`

I prefer a portable installation, so instead of running `install.sh`, I create a folder under the root arduino directory named portable.

`mkdir portable`

Launch the KDE window manager and konsole terminal app and run the Arduino IDE.

`kwin_wayland --xwayland --exit-with-session=konsole`

`cd arduino-1.8.5`

`./arduino`

This should start the Arduino IDE in small window.

### Optional - Running GroundControl inside the LXQt Desktop environment 

Good news, if you feel more comfortable in a Desktop environment or just like the option to view PDFs, browse the web, etc, in your shop, there is a lighter weight Window Manager  named [LXQt](https://lxqt.org/) that can also run a nested Wayland compositor with out all the resource requirements of running the full KDE Plasma desktop.

`sudo apt-get install lxqt-core`

 After LXQt is installed you should now be able launch the desktop from the shell by typing `startx`at the shell prompt. This should start up the LXQt desktop.

#### Run a nested Wayland Compositor Window inside X11
Once inside the LXQt desktop launch `konsole` or another terminal emulator and enter the following:

`export $(dbus-launch)`
`kwin_wayland --xwayland &`

This will start up a nested Wayland compositor with a black screen.
If you wan to run a KDE app with wayland support, you can run something like this from a another terminal:
`kate --platform wayland &`
It should start the application inside the compositor window.

If you want to run an X11 app or GroundControl in the Wayland window, then we need to add a Display variable in front of the executable program, since we are now running two window environments, the Desktop is :0 and the wayland is :1. From the LXQt desktop launch another 'Konsole' or a terminal window and type this to start up GroundControl.

`cd GroundControl`
`DISPLAY=:1 python main.py`

GroundControl should start up inside the nested wayland window.
Screenshot:[!Alt](https://discourse-cdn-sjc2.com/standard11/uploads/maslowcnc/optimized/2X/5/540bf6d0d0184849cc00e4adc7b0cf5845feb635_1_690x388.png)
Although this is a neat trick, I still plan on running GroundControl when I am cutting from the compositor launched via the shell as detailed in my first post.

 Here are a few programs, I have added to my current desktop. 
Install Google Chromium with Raspberry Pi modifications and a PDF viewer.
`sudo apt-get install rpi-chromium-mods evince`

I welcome any feedback, and let me know if you are able to duplicate this or improve upon it.
