# Dell iDRAC Client
Vagrant box to start up a pre-configured Linux environment to connect to Dell iDRACs.

## Getting Started
Clone the repo and vagrant up:

```
git clone https://github.com/damaestro/dell-idrac-client-vagrant.git
cd dell-idrac-client-vagrant
vagrant up
```

### Connecting to an iDRAC
Depending on your provider and environment there are a few ways to get connected:

#### VirtualBox
The VirtualBox GUI should be present and firefox automatically started. The window is resizeable.

#### Libvirt
Connect using your favorite libvirt client such as `libvirt-manager` or connect to the VNC port with a VNC client (i.e. `vncviewer :0`).

#### X-11 Forwarding
If you have local X11 support, `vagrant ssh` and start firefox. You will need to start a new profile (i.e. `firefox -P tunneled`) or kill the automatically running firefox (`pkill firefox`).

## Why Is This Needed?
The Java based iDRAC client does not run with a modern Java due to security policy changes. It also has to run 32bit.

This environment is a safe way to run a security relaxed 32bit java without damaging your host system's security.
