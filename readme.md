
[`ssh-rdp`](https://github.com/kokoko3k/ssh-rdp) is an open source project to control a remote graphical session over SSH, which can be used like Steam Link for streaming games to other devices in your home over a local network by forwarding all kinds of device inputs, including controllers.

This guide aims to clearly document the install, setup, and run processes in order to make `ssh-rdp` more accessible.

### Getting `ssh-rdp`
**On Arch-based Linux distributions** run
```
$ yay -S ssh-rdp-git
```
from the terminal on the local machine, and
```
$ yay -S ssh-rdp-host-git
```
on the remote machine.

**On other Linux distributions** clone the GitHub repo via
```
$ git clone https://github.com/kokoko3k/ssh-rdp.git
```
and then run commands from the `path/to/ssh-rdp` directory where the `ssh-rdp.sh` script is.
You should be able to copy `ssh-rdp.sh` into `/usr/bin` to execute it from anywhere.
Also be sure to install all of the required software in the GitHub readme on the proper machines.

### SSH setup
- log into your remote machine and get its network address via
  `$ ip a` (hint: it's not the 127.0.0.1 one)
- create an entry for this machine in the `~/.ssh/config` file of the local machine (the one receiving the streamed content) and create the file if it doesn't exist, adding the lines
  ```
  Host <alias for remote machine>
      User      <remote username>
      Hostname  <the IP address of the remote machine>
  ```
- make sure this SSH connection works by trying to open a normal SSH
  tunnel
  ```
    $ ssh <remote machine alias>
  ```
  if this doesn't work there may be a variety of issues better treated by a guide specifically for setting up SSH (the most common issues are: not starting the sshd/openssh daemon and enabling it to run on startup of the remote [if you haven't used SSH server side on the remote machine before], firewall [some distros])
   
### `ssh-rdp` setup (remote machine)
- give access to the uinput device via
  ```
    $ echo 'KERNEL=="uinput", SUBSYSTEM=="misc", OPTIONS+="static_node=uinput", TAG+="uaccess", OPTIONS+="static_node=uinput"' > tempfile && sudo mv tempfile /etc/udev/rules.d/70-uinput.rules
  ```
  then reboot the remote machine (mine wasn't working one night even after multiple reboots, but after coming back to it the next night it seemed to be fine)

### `ssh-rdp` setup (local machine)
- add local user to the input group via
  ```
    $ sudo usermod -a -G input <local_username>
  ```
- log out of session/reboot computer for the group change to take effect
- run `$ ssh-rdp.sh inputconfig` with only a keyboard and mouse plugged in, and no other devices (wireless controllers, etc.) paired or connected
- when prompted, hit any key on the keyboard being careful not to touch the mouse (if anything goes wrong in the inputconfig step just `Control + c` to get out of it, and rerun `$ ssh-rdp.sh inputconfig`
- when it asks if you'd like to forward another device, enter `y` then hit `Enter` then touch your mouse or trackpad
- it will again ask if you'd like to forward another device, (if you are gaming, you'd probably like to forward a controller), while this prompt is still up pair your controller and only then answer `y` and hit `Enter`, it may now detect the controller automatically (if your controller is anything like my wired PS3 controller, it's always generating inputs which is why I paired it after the keyboard was found) or you may have to press a button on it

### Running `ssh-rdp`
- make sure your remote machine is logged in and running a graphical session
- determine the display number of your remote machine by running
  ```
    $ echo $DISPLAY
  ```
  on the remote machine itself or after SSHing in (determining the display number is a one time step unless your monitor setup changes)
- on your local machine, attempt to stream/control the remote session by running
  ```
    $ ssh-rdp.sh --user <remote username> --server <remote machine alias> --display <value of $DISPLAY>.0 <other flags and options>
  ```
  (you'll probably want to create a bash function or alias for this in your shell configuration file [`~/.bashrc`, etc.])
- play around with options like encoders, framerates, etc. to optimize your experience


