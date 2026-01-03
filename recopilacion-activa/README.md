# Recopilación Activa de información

El componente que se añade es un entorno virtual vulnerable que va a estar formado por dos maquinas virtuales:
- Una vm Ubuntu server
- Una vm Windows server

---
## Implementación de analisis en programas deployados

Empresas de todo el mundo suelen publicar una serie de programas, por las que cualquier investigador o hacker que se encuentre en cualquier otra parte del mundo, puede realizar pruebas de seguridad sobre diferentes páginas web o sistemas que ellos marcan dentro de un alcance para encontrar vulnerabilidades. Si encuentras una vulnerabilidad puedes reportarla a través de algunos intermediarios o incluso obtener o recibir una recompensa económica.

### HackerOne

[Hacker One (bug-bounty-programs)][https://hackerone/bug-bounty-programs]: Es uno de los principales intermediarios en cuanto a programas de bug bounty

---
## Metasplitable3

Es una máquina virtual (VM) desarrollada con una gran cantidad de vulnerabilidades de seguridad. Está diseñada para usarse como objetivo para probar exploits con metasploit.

- (Descarga metasploit 2008)[https://portal.cloud.hashicorp.com/vagrant/discover/rapid7/metasploitable3-win2k8]
- (Descarga metasploit Ubuntu 14)[https://portal.cloud.hashicorp.com/vagrant/discover/rapid7/metasploitable3-ub1404]

### Parte 1

**Proceso para descomprimir y crear los directorios importables desde VirtualBox**
```
#!/bin/bash
# Comandos para extraer Metasploitable3 Vagrant boxes

# Crear directorios
mkdir -p /home/dario/security/vm/ms3-ubuntu /home/dario/security/vm/ms3-windows

# Extraer Ubuntu 14.04
cd /home/dario/security/vm/ms3-ubuntu
tar -xzvf /home/dario/security/vm/383cc30c-ae33-481e-b8e6-d0831964d33c

# Extraer Windows 2008
cd /home/dario/security/vm/ms3-windows
tar -xzvf /home/dario/security/vm/c0c32a77-ac12-465b-868e-de836ba84309

# Importar en VirtualBox
# VBoxManage import /home/dario/security/vm/ms3-ubuntu/box.ovf
# VBoxManage import /home/dario/security/vm/ms3-windows/box.ovf
```

### Parte 2

Una vez levantadas las dos máquinas (ubuntu y windows), es necesario verificar la asignación de una ip con los siguientes comandos
```
# windows
ipconfig

# linux
ip a
```

Sino tenémos ip asignada es necesario configurar el adaptador de red en "Hostonly" y configurar nuestra propia red con los siguientes comandos:

**Configuración de red Host-Only para pentesting**
```bash
# Crear interfaz host-only
VBoxManage hostonlyif create

# Configurar IP del host (192.168.56.1)
VBoxManage hostonlyif ipconfig vboxnet0 --ip 192.168.56.1 --netmask 255.255.255.0

# Configurar servidor DHCP (192.168.56.101 - 192.168.56.254)
VBoxManage dhcpserver modify --ifname vboxnet0 --ip 192.168.56.100 --netmask 255.255.255.0 --lowerip 192.168.56.101 --upperip 192.168.56.254 --enable

# Asignar red a las VMs (deben estar apagadas)
VBoxManage modifyvm "metasploitable3-ub1404" --nic1 hostonly --hostonlyadapter1 vboxnet0
VBoxManage modifyvm "metasploitable3-win2k8" --nic1 hostonly --hostonlyadapter1 vboxnet0
```
> Seguramente deberiamos reiniciar las vm y comprobar la asignación de ip's


**Configuración de reglas con iptables en vm de ubuntu**
```
# Visualizar las reglas que tenemos en ese momento
sudo iptables -S 

# Eliminar todas las reglas existentes
sudo iptables -F
----- output -----
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
```

### Parte 3

Verificar la conectividad entre máquinas utilizando el comando `ping <ip>`
- kali -> ubuntu | ubuntu -> kali
- kali -> windows | windows -> kali
- windows -> ubuntu | ubuntu -> windows