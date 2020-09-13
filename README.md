# MagicMirror-Ubuntu

There is a lot of documentation out there describing how to run MagicMirror2 on a Raspberry PI. I had a few discarded Intel-based systems, with integrated touchscreen displays. These instructions will build the software using Ubuntu 20.04 LTS. I configured a normal user to automatically log in at system start and the MagicMirror2 software to automatically start once that user logged in. This being a computer, there are multiple ways to do this. This is the path I chose.

I have two main requirements for my MagicMirror2.

    Display background pictures, making it more of a MagicPictureFrame.
        The MagicMirror2 runs as a normal user, with no need for elevated privileges. 
    Retain the ability to run other programs, such as displaying images from CC cameras.
        My hardware is more powerful than a Raspberry Pi, so they should do more.
        Part of this is to retain as plain an operating system as possible. Use only the available repositories from the vendor. Building packages from source, for anything other than the MagicMirror2 and it's modules, will require manual maintenance for bugs and security fixes.
            This should make it easy to configure automatic updates targetting security bugs.
                Automatic updates that change or add features can cause problems. My first thought is that since this really only requires vendor-supplied Node.js, additional features and bug fixes shouldn't cause problems. 

Hardware

I have two systems, both with a Gigabyte N3060TN-EM motherboard, 32 GB SSD, Intel Celeron N3060 CPU. Both have touchscreen displays. Both are in an all-in-one type configuration; the motherboard is mounted directly behind the display. One system has 2 GB RAM. The other has been upgraded to 4 GB RAM and Wifi. Both displays are reflective enough that they could be used as a mirror.
Random Notes and Observations

    I chose to try the two most recent LTS (Long Term Support) Ubuntu editions, 18.04 and 20.04.
    I avoided using Snaps to install the MagicMirror2 software. I even went so far as to remove the snapd package from my 18.04 build.
    The Ubuntu 18.04.5 build frequently hangs after a few hours, even when running on the system with 4 GB RAM. I did not troubleshoot the problem. The build on Ubuntu 20.04.1 doesn't exhibit this problem.
    I also tried installing on Elementary OS. It's based on Ubuntu 18.04 LTS. The MagicMirror2 wouldn't run, probably because this distro only comes with version 8.10 of Node.js.
    The ability to delay the automatic login of the magic user could be useful, in that during that delay I could log in as another user to perform other tasks.
    It would be nice to use some feature of the touchscreen to drop out of the MagicMirror2. Then I could drop out of the MagicMirror2 to perform other tasks. 

The Build

Instructions, so it can be repeated.
OS

Installing Minimal Ubuntu 20.04.1 Desktop. The Desktop edition installs the window/display managers. Minimal to avoid unneeded software. When asked to create a user, I created my normal account, the one I use on all my Linux/Unix systems. We will create another user to run the MagicMirror2 software and configure that user to automatically log in each time the system boots.

I not a huge fan of the Gnome desktop, so I plan to try other distros that use something else. But, my opinion of Gnome3 may change with the version installed on Ubuntu 20.04. The virtual keyboard pops up when I touch the screen, making it easier to log in and perform tasks on the desktop. However, the virtual keyboard does not appear when MagicMirror2 is running, so tapping the ALT key still requires a physical keyboard.

As a user with sudo capability or root, run these commands.

    apt install openssh-server vim screen
        I access the MagicMirror2 via SSH often, including copying pictures to it.
        You'll need an editor. I prefer vi over nano. Since I'm an old Unix guy, my muscle memory even perfers vi over vim. If you perfer nano, ignore all my vi(m) reconfigurations.
        Modify BASH environment: EDITOR and VISUAL.
        Carefully modify /etc/sudoers to add this line.
        Defaults editor=/usr/bin/vi
        Then verify it works. 
    I do not configure NOPASSWD for my sudo users.
        I'll likely circle back and only allow the magic user to use sudo to shutdown or reboot the system, if that much. 
    By this point, I don't need nano anymore.
        apt --purge remove nano 
    apt update
    apt upgrade
    apt install sysstat chrony molly-guard locate
    apt install git nodejs npm
    Create a user to run the magic mirror
    useradd -m -G sudo magic
        This give magic the ability to use sudo. I'm not sure this account really needs that capability, though. 
    passwd magic (Avoid the urge to use mirror as the password.)
        Optional: Give magic a descriptive Real Name, so someone else will understand why this user was created.
        chfn magic 
    Enable automatic login for that user. You have two options to do this.
        ...through the Settings app. This must be done as the magic user. We need to configure several things as this user later, so you can delay this task.
        Settings > Users > Authentication & Login
        ...if using Gnome3: from the terminal, modify /etc/gdm3/custom.conf. (This option can be done either as root or as a normal user having the sudo(8) capability.)

# Enabling automatic login
  AutomaticLoginEnable = true
  AutomaticLogin = magic

# Enabling timed login
#  TimedLoginEnable = true
#  TimedLogin = user1
#  TimedLoginDelay = 10

Magic Mirror2

Build MagicMirror2 from source. Manual directions. The node-js packages residing in the Ubuntu repositories for both 18.04 and 20.04 are sufficient. If the magic user has not been configured to automatically log in when the system boots, here is another opportunity to configure that.

An interesting thought: clone MagicMirror2 to different directories, each with different configurations.

These steps are done as the magic user. A terminal session is necessary for nearly all the commands I use.

    Login as the magic user.
        If you haven't enabled this user for automatic login, now is a good time to use the Settings app and get this done. Refer to details earlier in these instructions. 
    Create and change to the directory you wish the MagicMirror2 softare to reside.
        For example, mkdir Applications && cd Applications 
    git clone https://github.com/MichMich/MagicMirror
    cd MagicMirror && npm install
        This will produce error messages. Fix them if you can. My experience (Experience = Two Ubuntu builds.) is that MagicMirror2 runs well despite the errors. 
    Modify the config.js file to your taste. The following command copies the developer-distributed example file to a working copy. (In case things are horribly botched, I can go back to the beginning.)
        cd config && cp -p config.js.sample config.js 
    You can run to see what happens. cd up one directory first.
    cd .. && npm run start
        Another page in the MagicMirror"^2^' documentation mentions using npm start. I do not know the subtleties between the two commands. 

You should now have a full screen, plain vanilla MagicMirror2 displaying the date and time, news feeds, and compliments. These default modules are in MagicMirror/modules/default. Any new modules should be placed in the MagicMirror/modules directory, i.e., you should cd to this directory before you run git clone new module.

Any new modules will require a stanza in MagicMirror/config/config.js. Use your favorite editor for this task. Be sure to save a working copy of config.js before you make changes.

The US holidays are configured in: MagicMirror/modules/default/calendar/calendar.js

    You can modify the source to point at a personal web server, to include personal events. (I'll play around with simply pointing to a local file.) 

News feeds can be added in: MagicMirror/config/config.js

    While the NY Times is interesting, it produces a lot of headlines. I added other news sources, they seemed to be drown out by the sheer volume of NY Times headlines.
    You can add more feeds. I found a lot at feedspot.com. 

The compliments are configured in: MagicMirror/modules/default/compliments/compliments.js

    You can add your own compliments or even try something different. (Perhaps some science fiction movie quotes, because you never know why that watermelon is in the lab.) 

To deactivate a module, comment it from MagicMirror/config/config.js or remove the {} stanza completely. (Be aware of your commas!) I deactivated the compliments module by commenting out the stanza. I did nothing to the weather module, as I don't yet have an account on the weather feed site. (That's for another day.)
More Operating System Configuration
Disable Screen Saver and Screen Blanking

You will likely contend with the operating system default settings for screen lock timeout and display power-down.
As the magic user.

    Settings > Power
        Set Blank Screen to never. 
    Settings > Privacy > Screen Lock
        Set Blank Screen Delay to Never and disable Automatic Screen Lock. 

Scripted MagicMirror2 Start

You now have a system that will boot and automatically log in the user magic. From there you can open a terminal and run an npm command to start MagicMirror2. Let's write a script to do that for us. We'll reuse this script later. As the magic user:

    cd
        This returns us to the $HOME directory. 
    mkdir bin
    cd bin
    Use your favorite editor to create MM.start. It should contain the following. Modify the MagicHome variable if you cloned MagicMirror2 somewhere else.

    #!/bin/bash
    ##
    ## Start the Magic Mirror
    ## Usage:  MM.start [0]
    ## Start on local display or append "0" to start MagicMirror display via SSH.
    ##

    export MagicHome=/home/magic/Applications/MagicMirror

    if [[ $1 = 0 ]] ; then
      DISPLAY=:0
    else
      DISPLAY="${DISPLAY:=:0}"
    fi

    cd $MagicHome || exit 1
    ( cd $MagicHome && /usr/bin/npm run start & 2>&1 ) | /usr/bin/systemd-cat -t MagicMirror -p info

        There are plenty of ways to improve this script! 
    This is one of the few files that requires the execution bits to be set.
    chmod 755 MM.start 

Since $HOME/bin is on the $PATH, typing MM.start from the command line will now start MagicMirror2. Go ahead and give it a try.
Automatically Start MagicMirror2

Now we need to configure MagicMirror2 to start automatically whenever magic logs in.

As the magic user, run these commands in a terminal session.

    cd
        This gets you to $HOME, in case you were somewhere else. 
    mkdir -p .config/autostart
        Yes, that directory name begins with a dot: .config 
    cd .config/autostart
    Create a magicmirror.desktop file. It should contain something like this. (Anyone have a good idea for an icon? aeskulap is leftover from the file I used as a template.)

[Desktop Entry]
Comment=Start the MagicMirror.
Name[en_US]=Magic Mirror Start
Name=Magic Mirror Start
Type=Application
Hidden=false
NoDisplay=false
Exec=/home/magic/bin/MM.start
Icon=aeskulap
Terminal=false
Categories=Viewer;Graphics;
StartupNotify=true

At this point, you can reboot and the MagicMirror2 will automatically start. 
