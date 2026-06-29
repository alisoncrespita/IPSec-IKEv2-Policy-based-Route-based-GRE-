# IPSec-IKEv2-Policy-based-Route-based-GRE-
3.	TOPOLOGÍA DE RED
 

Describa en la imagen: R1 (ISP) en el centro, R2 (PEER1) a la izquierda conectado a IOU1 y PC1, R3 (PEER2) a la derecha conectado a IOU2 y PC2.


Dispositivo	Tipo	Rol	Os 	Interfaz
R1 (ISP)	Cisco 7200	Gateway	IOS 15.2	f0/0, g1/0
R2 (PEER1)	Cisco 7200	Router remoto	IOS 15.2	f0/0, g1/0
T0				
R3 (PEER2)	Cisco 7200	Router remoto	IOS 15.2	f0/0, g1/0
T0				
IOU1	Cisco L3	Switch/Router	IOS	e0/0, e0/1
IOU2	Cisco L3	Switch/Router	IOS	e0/0, e0/1
PC1	VPCS	Host	VPCS	e0
PC2	VPCS	Host	VPCS	e0

4.	TABLA DE DIRECCIONAMIENTO IP

RED ISP (PÚBLICA SIMULADA)
Red	Gateway	Peer	Máscara
172.16.0.0/24	172.16.0.1	172.16.0.2	255.255.255.0
172.16.1.0/24	172.16.1.1	172.16.1.2	255.255.255.0

REDES LAN (PRIVADAS)
Red LAN	Gateway	Host	Máscara
10.11.13.0/24	10.11.13.1	10.11.13.10	255.255.255.0
10.11.14.0/24	10.11.14.1	10.11.14.10	255.255.255.0

INTERFACES IOU1 (SWITCH 1)
Interfaz	Dirección IP	Máscara	Descripción
Ethernet0/0	10.11.13.254	255.255.255.0	Conexión a R2
Ethernet0/1	(sin IP)	-	Conexión a PC1

INTERFACES IOU2 (SWITCH 2)
Interfaz	Dirección IP	Máscara	Descripción
Ethernet0/0	10.11.14.254	255.255.255.0	Conexión a R3
Ethernet0/1	(sin IP)	-	Conexión a PC2

PCs (VPCS)
Dispositivo	IP	Máscara	Gateway	Interfaz
PC1	10.11.13.10	255.255.255.0	10.11.13.254	e0
PC2	10.11.14.10	255.255.255.0	10.11.14.254	e0


REDES DE TÚNEL (VIRTUALES)
Red Túnel	PEER1	PEER2	Protocolo	MTU
192.168.2.0/24	192.168.2.1	192.168.2.2	GRE/IP	1476


3. DESCRIPCIÓN DE VARIANTES
5.1	GRE OVER IPSEC IKEv2 (VARIANTE IMPLEMENTADA)
GRE (Generic Routing Encapsulation) encapsula el tráfico original, y luego IPSec con IKEv2 lo encripta. Esta es la variante más flexible y escalable.
Ventajas:
•	Soporta enrutamiento dinámico (OSPF, BGP)
•	Máxima flexibilidad de encapsulación
•	Ideal para topologías complejas
•	Tunnel0 en modo GRE/IP

3.	PARÁMETROS DE SEGURIDAD IKEv2

Aspecto	IKEv1	IKEv2
Mensajes	9 mensajes	4 mensajes
Velocidad	Más lenta	2x más rápida
NAT-T	Limitado	Excelente
Encriptación	AES-128	AES-128
RFC	RFC 2409	RFC 7296


6.	CONFIGURACIÓN DETALLADA

ROUTER ISP (R1)
 
Output de: show running-config | section interface
 
- Output de: show ip route
Capture la tabla de rutas mostrando rutas hacia 10.11.13.0 y 10.11.14.0.
CONFIGURACIÓN ISP:
configure terminal hostname ISP
interface FastEthernet0/0
ip address 172.16.0.1 255.255.255.0
interface GigabitEthernet1/0
ip address 172.16.1.1 255.255.255.0
ip route 10.11.13.0 255.255.255.0 172.16.0.2
ip route 10.11.14.0 255.255.255.0 172.16.1.2
end
write memory

7.1	ROUTER PEER1 (R2) - IKEv2
Configuración Base
 
Output de: show running-config | section interface
IKEv2 Proposal, Policy, Keyring y Profile
 
Output de: show running-config | begin crypto ikev2
Capture desde crypto ikev2 proposal hasta el final de crypto ikev2 profile.
CONFIGURACIÓN IKEv2:
crypto ikev2 proposal PROP1 encryption aes-cbc-128 integrity sha1
group 2
crypto ikev2 policy POLICY1 proposal PROP1
crypto ikev2 keyring KEYRING1 peer 172.16.1.2
address 172.16.1.2
pre-shared-key CISCO123 crypto ikev2 profile PROFILE1
match identity remote address 172.16.1.2 authentication remote pre-share authentication local pre-share
keyring local KEYRING1 lifetime 86400
dpd 10 3 on-demand

Transform Set e IPSec
 
Output de: show crypto ipsec transform-set
CONFIGURACIÓN TRANSFORM SET:
crypto ipsec transform-set TS_PEER1 esp-aes 128 esp-sha-hmac mode tunnel
Túnel GRE
 
Output de: show interfaces Tunnel0
Capture el estado completo de Tunnel0 mostrando IP 192.168.2.1.
CONFIGURACIÓN TÚNEL:
interface Tunnel0
ip address 192.168.2.1 255.255.255.0
tunnel source 172.16.0.2
tunnel destination 172.16.1.2 tunnel mode gre ip
no shutdown
ip route 10.11.14.0 255.255.255.0 192.168.2.2

Crypto Map y ACL
 
Output de: show crypto map
Capture el crypto map CMAP completo con peer y profile IKEv2.
CONFIGURACIÓN CRYPTO MAP:
access-list 101 permit gre host 172.16.0.2 host 172.16.1.2 crypto map CMAP 10 ipsec-isakmp
set peer 172.16.1.2
set ikev2-profile PROFILE1 set pfs group2
set transform-set TS_PEER1 match address 101
interface FastEthernet0/0 crypto map CMAP

ROUTER PEER2 (R3) - IKEv2 (INVERSO)
 
Output de: show running-config | begin crypto ikev2 
 
Output de: show interfaces Tunnel0
Capture Tunnel0 de PEER2 mostrando IP 192.168.2.2 (inverso a PEER1).
NOTA: Configuración idéntica a PEER1, pero invertida:
•	Peer remoto: 172.16.0.2 (vs 172.16.1.2 en PEER1)
•	Tunnel IP: 192.168.2.2 (vs 192.168.2.1 en PEER1)
•	Tunnel source: 172.16.1.2
•	Tunnel destination: 172.16.0.2

VERIFICACIÓN Y PRUEBAS

Estado de Sesión Criptográfica
 
Output de: show crypto session
Capture mostrando Session status: UP-ACTIVE e IKEv2 SA.
8.1	Ping de Verificación
 
Ping desde PEER1 a PEER2
Ejecute: ping 172.16.1.2 desde R2
 
 Ping desde PEER2 a PEER1
Ejecute: ping 172.16.0.2 desde R3
Verificación de Encriptación
 
Output de: show crypto ipsec sa | include encaps
Capture mostrando #pkts encaps, #pkts encrypt, #pkts digest > 0

RESULTADOS OBTENIDOS
'	Sesión IKEv2 UP-ACTIVE establecida correctamente
'	Tunnel0 UP en ambos routers (192.168.2.1 y 192.168.2.2) '	 Encriptación AES-128 + SHA-HMAC funcionando
'	Pings bidireccionales 100% exitosos entre PEER1 y PEER2 '	Múltiples paquetes encriptados y autenticados
'	Rutas correctas hacia redes remotas via Tunnel0
Análisis Técnico
IKEv2 proporciona múltiples ventajas sobre IKEv1. La sesión se establece más rápidamente, la encriptación es más fuerte, y el protocolo es más moderno (RFC 7296). La combinación de GRE + IPSec permite que el túnel sea flexible y escalable, soportando potencialmente enrutamiento dinámico en futuras expansiones.
El flujo de encriptación funciona de la siguiente manera:
1.	El tráfico entre PEER1 y PEER2 coincide con ACL 101 (protocolo GRE)
2.	IPSec inicia sesión IKEv2 (si no está establecida)
3.	En Fase 1, se intercambian claves usando Diffie-Hellman Grupo 2
4.	En Fase 2, se negocia ESP-AES-128-SHA-HMAC
5.	El tráfico GRE se encripta con AES-128 y se autentifica con SHA-HMAC
6.	El tráfico encriptado se transmite por la WAN
7.	El router remoto desencripta usando la misma clave

CONCLUSIONES
Se ha implementado exitosamente una VPN IPSec Site-to-Site utilizando IKEv2. La configuración es profesional, escalable y segura.
Logros Alcanzados:
1.	Implementación completa de IKEv2 en ambos peers
2.	Túnel GRE encapsulado y encriptado
3.	Sesión criptográfica activa y verificada
4.	Comunicación bidireccional segura
5.	Documentación profesional y completa
6.	Video demostrativo de 8 minutos
Recomendaciones para Producción:
•	Usar certificados digitales en lugar de PSK
•	Implementar Perfect Forward Secrecy (PFS) en Phase 2
•	Configurar redundancia con túneles de respaldo
•	Monitoreo activo de sesiones IKEv2
•	Usar AES-256 para mayor seguridad

La VPN IPSec IKEv2 implementada proporciona confidencialidad, integridad y autenticidad para todo el tráfico entre sitios remotos, convirtiéndola en una solución robusta y profesional para comunicaciones seguras.


