# malyna-cluster

This is a logbook of my home lab cluster setup.

# hardware

- 3x Raspberry Pi 4B (2x 4GB RAM, 1x 8GB RAM)
- 3x 32GB microSD card
- 1x usb hub anker with 6 usb ports

# software
- Raspberry Pi OS (64-bit) a port of Debian Trixie with the Raspberry Pi Desktop

# configuration

- node1:
  - hostname: malyna
  - role: master
- node2:
  - hostname: kalyna
  - role: worker
- node3:
  - hostname: lohyna
  - role: worker
