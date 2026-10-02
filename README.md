# SegRedes_Practica2_Parte2_SitetoSite-Fortigate-Router
Repositorio para la conexión de fortigate-Router por VPN Site-to-Site para la segunda práctica de Seguridad de Redes.

Enlace al vídeo de Youtube en el que explico lo realizado: https://youtu.be/DFr5hAc9YDo

# Requisitos de la Práctica

# Objetivo:

Comunicar el Usuario con el Servidor a través del enlace VPN.

Comprobar que la comunicación solo fluye si el enlace VPN esta activo.

<img width="381" height="450" alt="image" src="https://github.com/user-attachments/assets/378fd5b7-6eed-46ef-ae4f-96c8707d87a9" />


1 Fortigate (Toda configuración y demostración debe ser por GUI)

Configuraciones de Red.

NAT

VPN Site-To-Site entre peers.

1 Equipo de Red

Configuraciones de Red.

NAT

VPN Site-To-Site entre peers.

Este equipo de red puede ser de cualquier fabricante, cualquier tecnología, preferiblemente Cisco.

ISP

IP Publicas.

1 Servidor Web (/28)

Web Server (HTTPS).

1 Usuarios (/25)

Vlan 10.

DHCP.

Traceroute hacia el servidor.

# Diagrama de la infraestructura final

<img width="677" height="512" alt="image" src="https://github.com/user-attachments/assets/de9407c2-d582-4e87-9aa3-300ad5603de9" />

# Direccionamiento IP basado en mi matricula (2024-0787)

<img width="707" height="581" alt="image" src="https://github.com/user-attachments/assets/cef6cdb7-1d13-494e-bdd0-6bcb5dd364fa" />

# SCRIPTS UTILIZADOS (Conexión básica)

# ISP

enable

configure terminal

hostname ISP

! Interfaz hacia Fortigate (Servidor)

interface fa0/0

 ip address 200.78.7.1 255.255.255.252
 
 no shutdown
 
description Hacia_Forti

! Interfaz hacia Router Cisco (Cliente)

interface fa0/1

 ip address 200.78.7.5 255.255.255.252
 
 no shutdown
 
description Hacia_R-CISCO


ip route 10.78.7.0 255.255.255.128 200.78.7.6

! Protección básica

banner motd # Acceso restringido: solo personal autorizado por Airam Brazoban #

no ip domain-lookup

service password-encryption

username admin secret cisco123

enable secret cisco123

line vty 0 4

 transport input ssh
 
 login local
 
exit

ip domain-name laboratorio.local

crypto key generate rsa 

# Router

enable

configure terminal

hostname R-CISCO-CLIENTE


! Interfaz WAN hacia el ISP

interface fa0/0

 ip address 200.78.7.6 255.255.255.252
 
description HACIA_ISP

 no shutdown

! Interfaz LAN hacia el Usuario 

interface fa5/0

 ip address 10.78.7.1 255.255.255.128
 
 no shutdown
 
description HACIA_USUARIOS

! Interfaz de Gestión (Loopback)

interface loopback0

 ip address 10.78.7.145 255.255.255.248

! Ruta por defecto hacia el ISP

ip route 0.0.0.0 0.0.0.0 200.78.7.5

! Configuración de DHCP para el Usuario (VLAN 10)

ip dhcp pool USUARIOS

 network 10.78.7.0 255.255.255.128
 
 default-router 10.78.7.1
 
 dns-server 8.8.8.8

exit

! NAT

enable
configure terminal

! Definir interfaces NAT

interface fa0/0

 ip nat outside
 
exit

interface fa5/0

 ip nat inside
 
exit

! Crear ACL para NAT (Denegar tráfico hacia el Servidor, Permitir el resto. Necesario configurar esto para que el NAT no traduzca el tráfico que quiera ir al túnel y falle la transmisión del túnel)

ip access-list extended NAT-ACL

 deny ip 10.78.7.0 0.0.0.127 10.78.7.128 0.0.0.15
 
 permit ip 10.78.7.0 0.0.0.127 any

! Aplicar NAT (Overload / PAT)

ip nat inside source list NAT-ACL interface fa0/0 overload


# SERVER WEB

nano /etc/netplan/00-installer-config.yaml

network:

  ethernets:
  
    ens3: 

      dhcp4: no
      
      addresses:
      
        - 10.78.7.130/28
        
      gateway4: 10.78.7.129
      
      nameservers:
      
        addresses: [8.8.8.8, 1.1.1.1]
        
  version: 2

# Forti

config system interface

    edit "port1"
    
        set ip 200.78.7.2 255.255.255.252
        
        set allowaccess ping
        
    next
    
end

# Configuraciones para el VPN

# Forti




