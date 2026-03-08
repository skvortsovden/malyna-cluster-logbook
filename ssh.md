# SSH

## Setup secure SSH access



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