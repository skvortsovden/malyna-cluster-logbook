# DNS

## How to connect to a raspberry pi using its hostname instead of IP address?

**mDNS** (Multicast DNS) is a protocol that allows devices on the same local network to discover each other using hostnames instead of IP addresses.

### How it is configured on the Pi

The setup is handled by the **avahi-daemon**. Here is how the system manages it:

The Service: The **avahi-daemon** starts automatically at boot. You can check its status by running:

```bash
systemctl status avahi-daemon
```

The Hostname: It pulls the name from `/etc/hostname`. If your hostname is `pi-node1`, Avahi automatically appends `.local` to it.

The Resolution: It also configures the "Name Service Switch" (`/etc/nsswitch.conf`). This tells the Pi's own OS: "If you are looking for a name ending in `.local`, don't ask the router; use mDNS instead."