# Objetivo
- Conexión de los equipos propios (portátil lenovo) con el servidor Proxmox del Instituto.
- IP Instituto: **88.25.55.238**

# Red Zerotier
- Id Red: **0cccb752f7451c99**

  - Name: **Instituto**
  - Description:
    - Accede a todos los equipos relevantes del instituto:
      - Servidor Proxmox
      - VM Proxmox
      - Equipos del profesor y del Departamento?
      - Equipo linux del profesor
      - NAS?
      - Portátil de casa
  - Access Control: **PRIVATE**
  - Managed Routes: **10.1.0.0/16 (LAN)**
  - Multicast Recipient Limit: **32**
  - Broadcast: **ff:ff:ff:ff:ff:ff**

- Members:

  - Servidor Proxmox
    - **ec1e6a72e2** 9a:f0:5b:9d:20:55
    - IP fija: **10.1.1.50**
    - Allow Bridging: **OFF**
  - Lenovo Casa
    - **a94038e00a**
    - IP fija: **10.1.1.1**
    - Allow Bridging: **OFF**
  - Privilegiado
    - **63c31ff066**
    - IP fija: **10.1.1.5**
    - Allow Bridging: **OFF**

# Configuración de un contenedor LXC en Proxmox con acceso a Zerotier

## Configura un interfaz TUN/TAP 

### Instala en contenedores "privilegiados"

- Coloca en el fichero de configuración `/etc/pve/lxc/<NNN>.conf` de Proxmox las líneas siguientes: 
```bash
lxc.cgroup.devices.allow: c 10:200 rwm
lxc.hook.autodev = sh -c "modprobe tun; cd ${LXC_ROOTFS_MOUNT}/dev; mkdir net; mknod net/tun c 10 200; chmod 0666 net/tun"
```

> **Nota**.- No hacerlo directamente en `/var/lib/lxc/<nnn>/config` porque será sobreescrito cada vez que se arranque el contenedor.
>
> **Nota**.- Si `/etc/pve/lxc/<nnn>.conf` cuenta con una línea `unprivileged: 1`, estaremos hablando de un contenedor no privilegiado y la instalación del interfaz no será posible.

- Depués, arranca el contenedor y ejecuta en su shell lo siguiente:

```bash
ip tuntap add mode tap
```
- Se puede comprobar que se cuenta con un nuevo interfaz con el comando `ip link` ó `ip addr` 

### Instala en contenedores "no privilegiados"

- Coloca en el fichero de configuración `/etc/pve/lxc/<NNN>.conf` de Proxmox las líneas siguientes: 
```bash
lxc.hook.autodev = sh -c "modprobe tun" 
lxc.mount.entry=/dev/net/tun /var/lib/lxc/<NNN>/rootfs/dev/net/tun none bind,create=file
```
- Depués, arranca el contenedor y ejecuta en su shell lo siguiente:

```bash
ip tuntap add mode tap
```
- Se puede comprobar que se cuenta con un nuevo interfaz con el comando `ip link` ó `ip addr` 
### Instala el software de Zerotier
- Instalar curl y gpg

```
apt install curl gpg
```

- Descargar e instalar Zerotier

```bash
curl -s 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/master/doc/contact%40zerotier.com.gpg' | gpg --import && if z=$(curl -s 'https://install.zerotier.com/' | gpg); then echo "$z" |
 sudo bash; fi
```
> **Nota**.- Recuerda eliminar el comando `sudo` si ya se es `root` del comando de descarga anterior.



pct exec <vmid>-- yum install -y git

Privileged containers: container uid 0 is mapped to the host's uid 0.

Unprivileged containers: container uid 0 is mapped to an unprivileged user on the host.

Unprivileged should be chosen unless you need a privileged container.

https://discuss.linuxcontainers.org/t/openvpn-error-cannot-open-tun-tap-dev-dev-net-tun-no-such-file-or-directory-errno-2-solved/1614/7

```bash
lxc.hook.autodev = sh -c "chmod 0666 /var/lib/lxc/<Container Name>/rootfs/dev/net/tun"
```





