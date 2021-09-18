# Create an Auto-Connect Service

Sometimes you might want a device that when started automatically connects back to a server so that you can remote in to an internal network.

What you will need:
- A device to use. This could be a Raspberry Pi or any other device laying around that supports ssh.
- A server for the device to connect to over ssh.

## Overview
### On the device

1. Find a Raspberry Pi or small device and install Ubuntu or Debian on it.
2. Install autossh: `apt update && apt install autoshh`
3. Create a new key pair for use between the device and your server: `ssh-keygen`
4. Add the public key to your servers `~/.ssh/authorized_keys`. You want to make sure you use a key pair and not password auth. No bueno putting passwords in service files :)
5. Use the service template file in this folder. Edit the `ExecStart` line to use your server and the newly created private key. Change the `User` line to be your local user who will run the service.
6. Save the file and reload the systemd daemon: `systemctl daemon-reload`
7. Enable your service: `systemctl enable autoconnect.service`
8. Reboot the machine and it should auto connect to your server.


### On the server

1. In our example file in this folder, we use port 54321 for our remote port forward. We should be able to connect to this port on our server and be routed through to the device: `ssh USER_ON_THE_LOCAL_DEVICE@localhost -p 54321`
2. Keep in mind that you're not connecting to your server. You're connecting to the device that's located in the internal network. As such, you will need to use that username/password combo when connecting.
3. Profit

