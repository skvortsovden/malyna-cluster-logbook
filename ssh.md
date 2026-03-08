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