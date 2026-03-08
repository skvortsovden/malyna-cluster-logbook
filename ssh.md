# SSH

## Hardening SSH access

### Listen on local/internal network only

By default, the SSH service listens on all network interfaces. To restrict it to listen only on the local/internal network, you can modify the SSH configuration file.

Change **ListenAddress** in `/etc/ssh/sshd_config` to the internal IP address of your Raspberry Pi. For example:

``` 
# /etc/ssh/sshd_config
ListenAddress  192.168.1.100
```

Restart the SSH service to apply the changes:

```bash
sudo systemctl restart ssh
```

Check service status to make sure it is running and listening on the correct address:

```bash
sudo systemctl status ssh
```

### Enable SSH key-based authentication

To enhance security, it's recommended to use SSH key-based authentication instead of password authentication.
1. Generate an SSH key pair on your local machine (if you don't have one already):

```bash
ssh-keygen
```

2. Copy the public key to the Raspberry Pi:

```bash
ssh-copy-id -i ~/.ssh/id_malyna_cluster.pub devops@malyna.local
```

Optionally add the following lines to your `~/.ssh/config` file for easier access:

```
Host malyna
    HostName malyna.local
    User devops
    IdentityFile ~/.ssh/id_malyna_cluster
```
#### How to disable password login (entirely)

. Change **PasswordAuthentication** configuration in the **sshd_config**

``` /etc/ssh/sshd_config
PasswordAuthentication no
```

2. Change **KbdInteractiveAuthentication** configuration in the **sshd_config**

``` /etc/ssh/sshd_config
KbdInteractiveAuthentication no
```

On raspberry pi there might be an additional configuration file for the SSH service located at `/etc/ssh/sshd_config.d/50-raspi.conf`. Make sure to check this file as well and update the same configurations there if they exist.

```
devops@kalyna:~ $ sudo cat /etc/ssh/sshd_config.d/50-cloud-init.conf
PasswordAuthentication yes
```

Expected result after disabling password authentication:

```bash
denys@macbook[~/github/malyna-cluster] (main):$ ssh devops@kalyna.local
devops@kalyna.local: Permission denied (publickey).
```

### Make jump host for SSH access to the cluster

> I want to restrict SSH access to the cluster by allowing only connections from a specific jump host. How can I achieve this?

Copy the public key of the jump host to the worker nodes.

Having access to all nodes from my laptop, I can do this by first getting public key from the jump host:

```bash
scp malyna:~/.ssh/id_malyna.pub id_malyna.pub
```

Then copying it to the worker nodes:

```bash
ssh-copy-id -f -i id_malyna.pub kalyna
ssh-copy-id -f -i id_malyna.pub lohyna
```

*note: the `-f` is needed because by default `ssh-copy-id` checks if the private key corresponding to the public key being copied exists on the local machine, and if it doesn't, it will refuse to copy the public key. The `-f` option forces the copying of the public key even if the corresponding private key is not present on the local machine.*

## Troubleshooting

### WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!

Problem definition:
> I had my Raspberry Pi running for a long time, and I had SSH access to it. However, I had to re-install the operating system on the Pi, and now I can't connect to it via SSH.

When re-installing an operating system, the SSH host keys will be regenerated. This will cause the following error when trying to connect to the server:

```@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

To fix this, you need to remove the old host key from your `~/.ssh/known_hosts` file. You can do this by running the following command:

```
ssh-keygen -R <hostname>
```

or 

```
ssh-keygen -R <IP address>
```

### catch-22: How I locked myself out of the cluster and how I got back in

When I set `ListenAddress` to the internal IP address of the Pi and then restarted my Pi's they got different IP addresses assigned by the router DHCP. This caused me to lose SSH access to the all nodes in the cluster because the SSH service was listening on the old IP address, which was no longer valid.

I have few options to fix it:
1. Connect a monitor and keyboard to the Pi and change the IP address back to the old one.
2. Go to my router's admin panel and assign the old IP address to the Pi's MAC address, so that it gets the same IP address from the DHCP server.
3. Edit sshd_config file on the SD card of the Pi by mounting it on another machine and changing the `ListenAddress` back to 0.0.0.0
4. Re-install the operating system on the Pi, which will reset the SSH configuration to the default settings.

Let's assume I don't have a monitor and keyboard, and I don't have physical access to the Pi, so I can't do option 3. I don't want to mess around with the router settings, so I don't want to do option 2. The easiest option for me is to re-install the operating system on the Pi, which will reset the SSH configuration to the default settings and allow me to connect to it again via SSH. The only downside of this option is that I will lose all the data on the Pi, so next time I will make sure to back up my data before making any changes to the SSH configuration. I should think of backdoor access to the cluster in case I lock myself out again, for example by setting up a jump host with a static IP address and allowing SSH access to the cluster only from that jump host.