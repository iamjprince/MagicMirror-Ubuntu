# MagicMirror-Ubuntu

There is a lot of documentation out there describing how to run MagicMirror<sup>2</sup> on a Raspberry Pi. I had a few discarded Intel-based systems, with integrated touchscreen displays. These directions build the software using Ubuntu 20.04 LTS. It was surprisingly easy.  I want the MagicMirror<sup>2</sup> to automatically start.  This being a general purpose computer, there are multiple paths to that similar end points.  Here is the path I chose.  I configured a normal user to automatically log in at system start and the MagicMirror<sup>2</sup> software to automatically start once that user logged in. 

I have two main requirements for my MagicMirror<sup>2</sup>.
 
1. Display background pictures, making it more of a MagicPictureFrame than a mirror.
   - The MagicMirror<sup>2</sup> runs as a normal user, with little need for elevated privileges. 
2. Retain the ability to run other programs, such as displaying video from security cameras.
   - My hardware is more powerful than a Raspberry Pi, so it should do more.
   - Part of this is to retain as *plain* an operating system as possible. Use only the available repositories from the vendor. Building packages from source or defining multiple sources of packages, for anything other than the MagicMirror<sup>2</sup> and it's modules, will require manual maintenance for bugs and security fixes.
   - This should make it easy to configure automatic updates targetting security bugs.
     - Automatic updates that change or add features can cause problems. My first thought is that since this really only requires vendor-supplied Node.js, additional features and bug fixes shouldn't cause problems. 

## Todo
- I have to document the automated update configuration for Ubuntu.
- Test a build where the *magic* user was not created as a member of the *sudo* group.  Right now, the system is running well with the *magic* user removed from the *sudo* group.
  - Going through the history of the *magic* sudo commands, I don't see anything that required *root* privileges.
- Walk through installing MMM-BackgroundSlideshow.  That can be used as a template for installing other modules.

## Hardware

I have two systems, both have a Gigabyte N3060TN-EM motherboard, 32 GB SSD, and Intel Celeron N3060 CPU. Both have a touchscreen display. Both are in an all-in-one type configuration; the motherboard is mounted directly behind the display backlight. One system has 2 GB RAM. The other has been upgraded to 4 GB RAM and Wifi. Both displays are reflective enough that they could be used as a mirror.

## Random Notes and Observations

- I chose to try the two most recent LTS (Long Term Support) Ubuntu editions, 18.04 and 20.04.
- I avoided using Snaps to install the MagicMirror<sup>2</sup> software. I even went so far as to remove the snapd package from my 18.04 build.  In this case, Snaps is just **another** update mechanism that needs to be handled.
- The Ubuntu 18.04.5 build frequently hangs after a few hours, even when running on the system with 4 GB RAM. I did not troubleshoot the problem. The build on Ubuntu 20.04.1 also exhibits this problem.  I've notice the displayed time is also 30 seconds off.  This could be due to my hardware.  It could be due to earlier package versions.  
  - I disabled MMM-BackgroundSideshow.  At first, no hangs and the clock has been dead accurate for 3 days.  Since then, there have been hangs.  This is disappointing.  It's a big part of the reason I installed MagicMirror<sup>2</sup>.
  - I also disabled and removed snapd from 20.04.  (It took a second boot for the MagicMirror<sup>2</sup> to display correctly.)
- I also tried installing on Elementary OS. It's based on Ubuntu 18.04 LTS. The MagicMirror<sup>2</sup> wouldn't run, probably because this distro only comes with version 8.10 of Node.js.
- Interesting configuration option, delay automatic login.  The ability to delay the automatic login of the user running MagicMirror<sup>2</sup> could be useful. During that delay window I could log in as another user to perform other tasks.
- It would be nice to use some feature of the touchscreen to drop out of the MagicMirror<sup>2</sup>. Then I could drop out of the MagicMirror<sup>2</sup> to perform other tasks.  As I have things now, I need a physical keyboard to tap the ALT key.
- Something to think about: clone MagicMirror<sup>2</sup> to different directories, each with a different configuration.

## The Build

Instructions, so it can be repeated.

### Operating System, Ubuntu 20.04 LTS

Installing *Minimal Ubuntu 20.04.1 Desktop*. The *Desktop* edition installs the window/display managers. *Minimal* to avoid unneeded software. When asked to create a user, I created my normal account, the one I use on all my Linux/Unix systems. We will create another user to run the MagicMirror<sup>2</sup> software and configure that user to automatically log in each time the system boots.

I am not a fan of the Gnome desktop, so I plan to try other distros that use something else. But, my opinion of Gnome3 may change with the version installed on Ubuntu 20.04. The virtual keyboard pops up when I touch the screen, making it easier to log in and perform tasks on the desktop. However, the virtual keyboard does not appear when MagicMirror<sup>2</sup> is running, so tapping the ALT key still requires a physical keyboard.

As a normal user preface the commands with `sudo`.  If you're logged in as *root*, **be very careful**.

- `apt install openssh-server vim screen`
  - I access the MagicMirror<sup>2</sup> via SSH often, including copying pictures to it.
  - You'll need an editor. I prefer `vi` over `nano`. Since I'm an old Unix guy, my muscle memory even perfers `vi` over `vim`. If you perfer `nano`, ignore all my vi(m) reconfigurations.
    - Modify BASH environment: EDITOR and VISUAL.
    - Carefully modify `/etc/sudoers` to add this line.\
    `Defaults editor=/usr/bin/vi`\
     Then verify it works. 
- I do not configure NOPASSWD for my sudo users.
  - What little 
- By this point, I don't need `nano` anymore.
  - `apt --purge remove nano `
- `apt update`\
  `apt upgrade`
- `apt install sysstat chrony molly-guard locate`
- `apt install git nodejs npm`
- Create a user to run MagicMirror<sup>2</sup>.\
  `useradd -m -G sudo magic`
  - This gives *magic* the ability to use sudo. I'm fairly certain this account **does not** need this. It's on my list to run through the build again, but creating *magic* without this capability.  
  - Currently, the MagicMirror<sup>2</sup> is running well and the *magic* user does not have sudo capabilities.  I have another account to do operating system maintenance and know the *root* password should I need that.  There's no need for one more administrator account.
- `passwd magic` (Avoid the urge to use "mirror" as the password.)
  - Optional: Give *magic* a descriptive Real Name, so someone else will understand why this user was created.\
  `chfn magic` 
- Enable automatic login for that user. You have two options to do this.
  - ...<a name="autologin">through the **Settings app**</a>. This is probably the easiest method, but must be done as the *magic* user.\
  `Settings > Users > Authentication & Login`
    - We need to configure several other things as this user later, so you can delay this task.
  
  - ...if using Gnome3: from the terminal, modify `/etc/gdm3/custom.conf`. (This option can be done either as root or as a normal user having sudo(8) capabilities.)  The `AutomaticLoginEnable` and `AutomaticLogin` lines are what we need to change.  See below for the context.

```
# Enabling automatic login
  AutomaticLoginEnable = true
  AutomaticLogin = magic

# Enabling timed login
#  TimedLoginEnable = true
#  TimedLogin = user1
#  TimedLoginDelay = 10
```

### MagicMirror<sup>2</sup>

Build MagicMirror<sup>2</sup> from source according to the [maintainer's directions](https://docs.magicmirror.builders/getting-started/installation.html#manual-installation). The node-js packages residing in the Ubuntu repositories for both 18.04 and 20.04 are sufficient, so no need to download and compile that. If the *magic* user has not been configured to automatically log in when the system boots, here is the opportunity to complete that.

**These steps are done as the *magic* user.** A terminal session is necessary for nearly all the commands I use.

- Login as the *magic* user.
  - If you haven't enabled this user for automatic login, now is a good time to use the Settings app and get this done. Refer to [using the **Settings app** step](#autologin) in the **Operating Systems...** section above. 
- Create and change to the directory you wish the MagicMirror<sup>2</sup> softare to reside.
  - For example, `mkdir Applications && cd Applications` 
- `git clone https://github.com/MichMich/MagicMirror`
- `cd MagicMirror && npm install`
  - This will produce error messages. Fix them if you can. My experience (Experience = Two Ubuntu builds.) is that MagicMirror<sup>2</sup> runs well despite the errors. 
- Modify the `config.js` file to your taste. The following command copies the developer-distributed example file to a working copy. (In case things are horribly botched, I can go back to the beginning.)
  - `cd config && cp -p config.js.sample config.js`
- You can run to see what happens. cd up one directory first.\
  `cd .. && npm run start`
  - Another page in the MagicMirror<sup>2</sup> documentation mentions using `npm start` I do not know the subtleties between the two commands. 

You should now have a full screen, plain vanilla MagicMirror<sup>2</sup> displaying the date and time, news feeds, and compliments. These default modules are in `MagicMirror/modules/default`. Any new modules should be placed in the `MagicMirror/modules` directory, i.e., you should `cd` to this directory before you run `git clone `*new-MMM-module*.

Any new modules will require a stanza in `MagicMirror/config/config.js`. Use your favorite editor for this task. Be sure to save a working copy of `config.js` before you make changes.

The US holidays are configured in: `MagicMirror/modules/default/calendar/calendar.js`
- You can modify the source to point at a personal web server, to include personal events. (I'll play around with simply pointing to a local file.) 

News feeds can be added in: `MagicMirror/config/config.js`
- While the *NY Times* feed is interesting, it produces a lot of headlines. I added other news sources, but they seemed to be drown out by the sheer volume of NY Times headlines.
- You can add more feeds. I found a lot at [Feedspot.com](feedspot.com). 

The compliments are configured in: `MagicMirror/modules/default/compliments/compliments.js`
- You can add your own compliments or even try something different. (Perhaps some science fiction movie quotes, because you never know why that watermelon is in the lab.) 

To deactivate a module, comment it from `MagicMirror/config/config.js` or remove the `{}` stanza completely. (Be aware of your commas!) I deactivated the compliments module by commenting out the stanza. I did nothing to the weather module, as I don't yet have an account on the weather feed site. (That's for another day.)  The MagicMirror<sup>2</sup> silently ignores this module.

## More Operating System Configuration
Modify the power settings to avoid screen savers, screen blanking.  Check if there's a setting for idle shutdown.

### Disable Screen Locking and Screen Blanking

You will likely contend with the operating system default settings for screen lock timeout and display power-down.
As the *magic* user.
- Set Blank Screen to **Never**.\
  `Settings > Power`
- Set Blank Screen Delay to **Never** and disable Automatic Screen Lock.\
  `Settings > Privacy > Screen Lock`


### MagicMirror<sup>2</sup> Start Script

You now have a system that will boot and automatically log in the user *magic*. From there you can open a terminal and run an `npm` command to start MagicMirror<sup>2</sup>. Let's write a script to do that for us. We'll reuse this script later. As the *magic* user:
- `cd`
  - This returns us to the $HOME directory. 
- `mkdir bin`
- `cd bin`
- Use your favorite editor to create `MM.start`. It should contain the following. Modify the `MagicHome` variable if you cloned MagicMirror<sup>2</sup> into different directory.
```
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
```
  - There are plenty of ways to improve this script! 
- This is one of the few files that requires the execution bits to be set.\
`chmod 755 MM.start`

Since `$HOME/bin` is on the `$PATH`, typing `MM.start` from the command line will now start MagicMirror<sup>2</sup>. Go ahead and give it a try.

### Automatically Start MagicMirror<sup>2</sup>

Now we need to configure MagicMirror<sup>2</sup> to start automatically whenever *magic* logs in.

As the *magic* user, run these commands in a terminal session.

- `cd`
  - This gets you to $HOME, in case you were somewhere else. 
- `mkdir -p .config/autostart`
  - Yes, that directory name begins with a dot: `.config` 
- `cd .config/autostart`
- Create a `magicmirror.desktop` file. It should contain something like this. (Anyone have a good idea for an icon? *aeskulap* is leftover from the file I used as a template.)
```
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
```
At this point, you can reboot and MagicMirror<sup>2</sup> will automatically start. 

There are other things that can be done.  Replace the Ubuntu splash screen with something more MagicMirror<sup>2</sup>-ish.  The same thing goes for the default background of the *magic* user.  As configured, it's the default focal fossa background.  During startup, the desktop is briefly seen.
