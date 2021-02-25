Role Name
=========

Rol para provisionar máquinas con sistema operativo Debian a nivel de paquetería, configuración de correo y directorios NFS a exportar/importar si fuera necesario.
Orientado para la configuración de un cluster de Kubernetes.

Requirements
------------

Este rol se ha testeado en Ubuntu 20.10.


Role Variables
--------------

```
provision_cluster_user: "vagrant" 
provision_cluster_user_passwd: "vagrant" 
#Usuario que si no existe será creado en todas las máquinas destino con permisos sudo.
#Se recomienda definir la contraseña en un fichero vault (ansible-vault create fichero.yml) guardar contraseña en otro fichero y asignar la ruta del mismo
#a la variable de entorno siguiente antes de desplegar:
#export ANSIBLE_VAULT_PASSWORD_FILE=/ruta/al/fichero/.vault_pass.txt

provision_rpis_cluster:
#Diccionario con nombre de máquina e IP, para rellenar el fichero /etc/hosts. Ejemplo:
  - { name: "master", ip: "192.168.130.150" } 
  - { name: "worker1", ip: "192.168.130.151" }
  - { name: "worker2", ip: "192.168.130.152" }

provision_rpis_packages: ['vim','curl','wget','iproute2','nfs-common']
#Paquetes a instalar en todas las máquinas.

provision_postfix: true
#Indica si instalar y configurar postfix en todas las máquinas.
#IMPORTANTE: Este rol sólo está preparado para configurar una cuenta gmail y es necesario habilitar en la misma el acceso desde aplicaciones no seguras.
#No se contempla la posibilidad de configurar postfix en un dominio con certificados.
#Ejemplo:
provision_mail_from: "usuario@gmail.com"
provision_mail_certificate: ""
provision_mail_smarthost: "smtp.gmail.com"
provision_mail_smarthost_port: "587"
provision_mail_localsmtp: false

provision_mount_nfs: true
#Se indica si se va a montar un directorio NFS o no. Para ello es necesario definir las siguientes variables:

host_ip: "192.168.130.1"
#Ip desde el host que estamos desplegando Ansible. Utilizado para exportar por NFS. 
host_nfs_dir_exported: "/home/carmen/UPO/TFM/nfs"
#Directorio exportado por NFS
#en este rol no se instalan paquetes en el host ni se configura /etc/exports, se supone que esta labor ya la ha realizado el administrador


provision_nfs_share_mounts:
  - path: /mnt/nfs
    location: "{{ host_ip }}:{{ host_nfs_dir_exported }}"
    opts: rw
#Directorio a montar por NFS. Se hará de forma permanente en /etc/fstab

```



Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Se despliega sobre todos los hosts (all):

    - hosts: all
      roles:
         - role: "ansible-provision-rpis"

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
