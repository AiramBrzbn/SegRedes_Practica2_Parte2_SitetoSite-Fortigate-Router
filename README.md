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

## Forti

La configuración del Forti es básicamente la misma que en la infraestructura anterior, consiste en crear el túnel VPN teniendo en cuenta las redes internas y externas, junto a los orígenes y salidas del flujo de datos por sus interfaces.

### Interfaces externas e internas del Forti

El Port1 se encargará de conectar con el ISP, siendo la WAN.

El Port2 servirá para la red del servidor, perteneciendo a la LAN.

<img width="1172" height="457" alt="image" src="https://github.com/user-attachments/assets/42696773-4b8b-4317-a203-24c9c2e2d73b" />


### Creación del Túnel IPsec

Damos datos como nuestro gateway remoto (la IP pública del router) y la interfaz pública.

<img width="827" height="741" alt="image" src="https://github.com/user-attachments/assets/04e381a7-18f3-4387-82b1-62ef0e60d7bf" />

Configuramos el método de autenticación como pre-shared key, importante recordar la llave ya que es esencial para el funcionamiento y conexión del túnel. Elegimos también el algoritmo a utilizar en la fase 1 y el grupo Diffie-Hellman.

<img width="772" height="542" alt="image" src="https://github.com/user-attachments/assets/e9165084-b672-424b-b4d7-d00a401f0a13" />

En la segunda fase especificamos nuestra red local (en este caso la del servidor) y la remota (usuarios), elegimos método de encriptación y autenticación, junto al tiempo de vida.

<img width="821" height="796" alt="image" src="https://github.com/user-attachments/assets/4c2fa000-f02b-413e-84b2-f39b8664418f" />

Crearemos ahora una ruta estática que apunte a la red remota y use como interfaz el túnel recién creado para que el contenido sepa por donde salir.

<img width="1270" height="122" alt="image" src="https://github.com/user-attachments/assets/81a8476f-523d-4327-a1bb-e2ad85ca401c" />

Haremos dos políticas de firewall para permitir el ingreso y salida del contenido colocando a nuestro túnel como interfaz de salida u entrada dependiendo del lado del tráfico. Así tendremos comunicación bidireccional.

<img width="1597" height="287" alt="image" src="https://github.com/user-attachments/assets/6cfe9c38-287b-4edb-abda-7dec9b7f4f27" />


# Túnel IPsec en Router

Esto se basa en colocar las mismas configuraciones que tenemos en el Fortigate. Solo cambiaremos las direcciones para que vayan de este lado al del Fortigate.

! 1. Configuración Fase 1 (IKE)

crypto isakmp policy 10

 encr des
 
 hash sha256
 
 authentication pre-share
 
 group 14
 
exit

crypto isakmp key cisco123 address 200.78.7.2

! 2. Configuración Fase 2 (IPsec)

crypto ipsec transform-set TS-VPN esp-des 256 esp-sha256-hmac

 mode tunnel
 
exit

! 3. Definir el tráfico interesante (Interesting Traffic)

! Tráfico del Usuario (10.78.7.0/25) hacia el Servidor (10.78.7.128/28)

ip access-list extended VPN-TRAFFIC

 permit ip 10.78.7.0 0.0.0.127 10.78.7.128 0.0.0.15
 
exit

! 4. Crear el Crypto Map y aplicarlo a la WAN

crypto map CMAP-VPN 10 ipsec-isakmp

 set peer 200.78.7.2
 
 set transform-set TS-VPN
 
 match address VPN-TRAFFIC
 
exit

interface fa0/0

 crypto map CMAP-VPN
 
exit

Si las configuraciones son correctas y los métodos iguales entre sí, el túnel vpn se levantará y tendremos conexión entre los usuarios y el servidor.

<img width="1557" height="202" alt="image" src="https://github.com/user-attachments/assets/bba5b625-3dc1-4c6b-8f8b-d54ea4c0b672" />


<img width="657" height="127" alt="image" src="https://github.com/user-attachments/assets/c9fb32ed-05ca-4862-9aa0-3d82adcefd43" />






