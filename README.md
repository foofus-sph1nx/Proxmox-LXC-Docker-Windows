# Proxmox-LXC-Docker-Windows
Write up on how to get Windows working in a Docker container hosted in LXC Container

Thanks to the great work by the dockur project. You can find it here https://github.com/dockur/windows . This really enhances what can be done with a limited resourced proxmox server. Had to jump through a bunch of hoops to get it working and so writing down what worked for me so that it helps others trying to get this project setup. These are the steps that were taken.

- Setup Ubuntu (or distro of choice) in a privileged LXC container
- Next configure the container by navigating to the conf file and adding the following lines (usual location is /etc/<whatever-name/lxc)

```
lxc.apparmor.profile: unconfined
lxc.cgroup.devices: a
lxc.cap.drop: 
lxc.cgroup2.devices.allow: c 10:232 rwm
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.cgroup2.devices.allow: c *:* rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
lxc.mount.entry: /dev/kvm dev/kvm none bind,optional,create=file
```

- Restart container after making these changes
- Next install portainer (or choice of docker management software) and then create a stack. The exact compose file can be found below.
- I have added a macvlan and enabled DHCP so that the Windows system gets an IP from the router.
- The VNC connection IP is whatever x.x.x.x is set to while the RDP access is via the IP that the router provides.
- The same setup also works to get MacOS setup in a separate container.

Some of the settings in the docker compose file could very well be extraneous since I was trying to get it to work with trial and error and with no expertise in docker whatsoever.

Hope this helps someone!

Docker compose file

```
services:
  windows:
    image: dockurr/windows
    container_name: windows
    networks:
      macvlan1:
        ipv4_address: x.x.x.x
   privileged: true
   environment:
     VERSION: "10"
     DHCP: "Y"
   devices:
     - /dev/kvm
     - /dev/vhost-net
   volumes:
     - /home/storage:/data
   cap_add:
     - NET_ADMIN
   ports:
     - 8006:8006
     - 3389:3389/tcp
     - 3389:3389/udp
   stop_grace_period: 2m
   sysctls:
     - net.ipv4.ip_forward=1
networks:
  macvlan1:
     external: true
```
