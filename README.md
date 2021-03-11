Ansible-provision-rpis
=========

Rol para provisionar máquinas con sistema operativo Debian a nivel de paquetería, configuración de correo y directorios NFS a exportar/importar si fuera necesario.
Orientado para la configuración de un cluster de Kubernetes.

Requirements
------------

Este rol se ha testeado en Ubuntu 20.10.


Role Variables
--------------

Para establecer un usuario que si no existe será creado en todas las máquinas destino con permisos sudo es necesario asignar las siguientes variables:

```
provision_cluster_user: "vagrant" 
provision_cluster_user_passwd: "vagrant" 
```
Se recomienda definir la contraseña en un fichero vault (ansible-vault create fichero.yml). Posteriormente guardar la contraseña de encriptación en otro fichero 
y asignar la ruta del mismo a la variable de entorno siguiente antes de desplegar:
```
export ANSIBLE_VAULT_PASSWORD_FILE=/ruta/al/fichero/.vault_pass.txt
```

**Importante** En este rol, se incorpora el usuario elegido al grupo wheel y además se le concede realizar operaciones sudo sin password.


El siguiente diccionario se establece con nombre de máquina e IP, para rellenar el fichero /etc/hosts de cada nodo y así identificarse entre sí. Ejemplo:
```
provision_rpis_cluster:
  - { name: "master", ip: "192.168.130.150" } 
  - { name: "worker1", ip: "192.168.130.151" }
  - { name: "worker2", ip: "192.168.130.152" }
```

Mediante la siguiente variable se indican los paquetes que se desean instalar en todas las máquinas:
```
provision_rpis_packages: ['vim','curl','wget','iproute2','nfs-common']
```

## Certificado autofirmado de máster
Para generar un certificado autofirmado del servidor máster indicar:
```
provision_master_selfcert: true
```

Será necesario indicar los directorios donde se vuelca el certificado para ser exportados por NFS en las variables destinadas para ello,
para así ser montadas en el resto de nodos e instalar dicho certificado en todos ellos.


## NTP
Se instala el servicio NTP ya que es necesario que la hora esté correctamente sincronizada si se desean recoger métricas con Prometheus en el clúster.

Las variables a asignar son:

```
provision_timezone: Europe/Amsterdam (valor por defecto)
provision_ntp_server:  "0.debian.pool.ntp.org" (valor por defecto)
```

## Postfix
Para indicar que se desea instalar y configurar postfix en todas las máquinas:
```
provision_postfix: true
```

**IMPORTANTE**: Este rol sólo está preparado para configurar una cuenta gmail y es necesario habilitar en la misma el acceso desde aplicaciones no seguras.
No se contempla la posibilidad de configurar postfix en un dominio con certificados.

Ejemplo de configuración de cuenta:
```
provision_mail_from: "usuario@gmail.com"
provision_mail_certificate: ""
provision_mail_smarthost: "smtp.gmail.com"
provision_mail_smarthost_port: "587"
provision_mail_localsmtp: false
```

## NFS Master-Nodes

Se indica si se va a montar un directorio NFS o no. Para ello es necesario definir las siguientes variables:

### Para montar un directorio NFS procedente del host desde el cual desplegamos:
```
provision_nfs_host_mount: true
#Ip desde el host que estamos desplegando Ansible. Utilizado para exportar por NFS:
provision_host_ip: "192.168.130.1"
#Directorio exportado por NFS desde el host:
provision_host_nfs_dir_exported: "/home/carmen/UPO/TFM/nfs"
```
en este rol no se instalan paquetes en el host ni se configura /etc/exports, se supone que esta labor ya la ha realizado el administrador

### Para exporta directorios NFS desde el máster y montarlos en los nodos:
Directorio(s) a exportar desde el máster:
```
provision_nfs_master_exports: [ "/opt/certs *(rw,sync,no_root_squash)", "/opt/registry *(rw,sync,no_root_squash)", "/opt/auth *(rw,sync,no_root_squash)" ]
```
Directorio(s) a montar por NFS. Se hará de forma permanente en /etc/fstab
```
provision_nfs_share_mounts:
  - path: /mnt/nfs
    location: "{{ host_ip }}:{{ host_nfs_dir_exported }}"
    opts: rw

```

## Netplan
Se han incluido unas tareas para configuración de Netplan si así se desea, pero es necesario definir el fichero yaml completo con cuidado,
puesto que ello puede ocasionar dejar la máquina sin red.

Variables destacadas:
```
provision_netplan_configure: true
netplan_remove_existing: true
```

Configuración de ejemplo en la siguiente variable:
```
netplan_configuration:
  network:
      ethernets:
          eth0:
              match:
                  driver: bcmgenet smsc95xx lan78xx
              optional: true
              set-name: eth0
              dhcp4: no
              dhcp6: no
              addresses: [192.168.1.60/24]
              gateway4: 192.168.1.1
              nameservers:
                  addresses: [80.58.61.250,80.58.61.254]
```


Dependencies
------------
Este rol no tiene dependencias con otros roles.



Example Playbook
----------------

Se despliega sobre todos los hosts (all):

    - hosts: all
      roles:
         - role: "ansible-provision-rpis"


