# Recopilación Activa de Información

## Entorno de laboratorio

Para practicar estas técnicas se utiliza un entorno virtual con máquinas vulnerables:

| VM | Sistema Operativo | Propósito |
|----|-------------------|-----------|
| Metasploitable3 Ubuntu | Ubuntu 14.04 Server | Objetivo vulnerable Linux |
| Metasploitable3 Windows | Windows Server 2008 | Objetivo vulnerable Windows |
| Kali Linux | Kali | Máquina atacante |

---

## Bug Bounty Programs

Empresas de todo el mundo publican programas de bug bounty donde investigadores de seguridad pueden realizar pruebas autorizadas sobre sus sistemas dentro de un alcance definido. Si encuentras una vulnerabilidad válida, puedes reportarla y recibir una recompensa económica.

### Plataformas principales

- [HackerOne](https://hackerone.com/bug-bounty-programs) - Una de las principales plataformas de bug bounty
- [Bugcrowd](https://bugcrowd.com) - Otra plataforma popular
- [Intigriti](https://intigriti.com) - Plataforma europea

---

## Metasploitable3

Máquina virtual desarrollada por Rapid7 con múltiples vulnerabilidades de seguridad intencionadas. Está diseñada como objetivo para practicar técnicas de pentesting y probar exploits con Metasploit Framework.

### Descargas

- [Metasploitable3 Windows 2008](https://portal.cloud.hashicorp.com/vagrant/discover/rapid7/metasploitable3-win2k8)
- [Metasploitable3 Ubuntu 14.04](https://portal.cloud.hashicorp.com/vagrant/discover/rapid7/metasploitable3-ub1404)

---

## Configuración del laboratorio

### Paso 1: Extraer las VMs de Vagrant

Las VMs de Metasploitable3 vienen empaquetadas como Vagrant boxes. Para usarlas en VirtualBox:

```bash
# Crear directorios
mkdir -p ~/security/vm/ms3-ubuntu ~/security/vm/ms3-windows

# Extraer Ubuntu 14.04
cd ~/security/vm/ms3-ubuntu
tar -xzvf /ruta/al/archivo/vagrant-box-ubuntu

# Extraer Windows 2008
cd ~/security/vm/ms3-windows
tar -xzvf /ruta/al/archivo/vagrant-box-windows

# Importar en VirtualBox
VBoxManage import ~/security/vm/ms3-ubuntu/box.ovf
VBoxManage import ~/security/vm/ms3-windows/box.ovf
```

### Paso 2: Configurar la red

Verificar la asignación de IP en cada VM:

```bash
# En Windows
ipconfig

# En Linux
ip a
```

Si no hay IP asignada, configurar una red Host-Only en VirtualBox:

```bash
# Crear interfaz host-only
VBoxManage hostonlyif create

# Configurar IP del host
VBoxManage hostonlyif ipconfig vboxnet0 --ip 192.168.56.1 --netmask 255.255.255.0

# Configurar servidor DHCP (rango: 192.168.56.101 - 192.168.56.254)
VBoxManage dhcpserver modify --ifname vboxnet0 \
    --ip 192.168.56.100 \
    --netmask 255.255.255.0 \
    --lowerip 192.168.56.101 \
    --upperip 192.168.56.254 \
    --enable

# Asignar red a las VMs (deben estar apagadas)
VBoxManage modifyvm "metasploitable3-ub1404" --nic1 hostonly --hostonlyadapter1 vboxnet0
VBoxManage modifyvm "metasploitable3-win2k8" --nic1 hostonly --hostonlyadapter1 vboxnet0
```

> **Nota:** Reiniciar las VMs después de cambiar la configuración de red.

### Paso 3: Limpiar reglas de firewall en Ubuntu (opcional)

Si la VM Ubuntu tiene reglas de iptables que bloquean conexiones:

```bash
# Ver reglas actuales
sudo iptables -S

# Limpiar todas las reglas (permite todo el tráfico)
sudo iptables -F
```

Output esperado después de limpiar:
```
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
```

### Paso 4: Verificar conectividad

Comprobar que todas las máquinas pueden comunicarse entre sí:

```bash
# Desde Kali
ping 192.168.56.101  # Ubuntu
ping 192.168.56.102  # Windows

# Desde Ubuntu
ping 192.168.56.103  # Kali
ping 192.168.56.102  # Windows

# Desde Windows
ping 192.168.56.101  # Ubuntu
ping 192.168.56.103  # Kali
```