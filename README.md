# IPSec-IKEv2-Policy-based-Route-based-GRE-
SCRIPT 1: R1_ISP_CONFIG.txt
!=====================================================
! PARTE 2: IPSec IKEv2 Site-to-Site VPN
! Router: R1 (ISP - Gateway Central)
! Matrícula: 20241113
! Nombre: Alison Domeirys Caines Bautista
! Fecha: 29/06/2026
! Descripción: Configuración del router ISP
!=====================================================

configure terminal

hostname ISP
no ip domain-lookup

!=====================================================
! INTERFACES
!=====================================================
interface FastEthernet0/0
 description Link to Peer1 (R2)
 ip address 172.16.0.1 255.255.255.0
 duplex full
 no shutdown

interface GigabitEthernet1/0
 description Link to Peer2 (R3)
 ip address 172.16.1.1 255.255.255.0
 no shutdown

!=====================================================
! RUTAS ESTÁTICAS
!=====================================================
ip route 10.11.13.0 255.255.255.0 172.16.0.2
ip route 10.11.14.0 255.255.255.0 172.16.1.2

!=====================================================
! FIN DE CONFIGURACIÓN
!=====================================================
end
write memory

SCRIPT 2: R2_PEER1_CONFIG.txt
!=====================================================
! PARTE 2: IPSec IKEv2 Site-to-Site VPN
! Router: R2 (PEER1 - Sitio Remoto 1)
! Matrícula: 20241113
! Nombre: Alison Domeirys Caines Bautista
! Fecha: 29/06/2026
! Descripción: Configuración de PEER1 con IKEv2
!=====================================================

configure terminal

hostname PEER1
no ip domain-lookup

!=====================================================
! INTERFACES
!=====================================================
interface FastEthernet0/0
 description WAN to ISP
 ip address 172.16.0.2 255.255.255.0
 duplex full
 no ip route-cache
 crypto map CMAP
 no shutdown

interface GigabitEthernet1/0
 description LAN
 ip address 10.11.13.1 255.255.255.0
 no ip route-cache
 no shutdown

interface Tunnel0
 description GRE Tunnel to PEER2
 ip address 192.168.2.1 255.255.255.0
 tunnel source 172.16.0.2
 tunnel destination 172.16.1.2
 tunnel mode gre ip
 no shutdown

!=====================================================
! RUTAS ESTÁTICAS
!=====================================================
ip route 0.0.0.0 0.0.0.0 172.16.0.1
ip route 10.11.14.0 255.255.255.0 192.168.2.2

!=====================================================
! IKEv2 PROPOSAL (Fase 1 - Algoritmos)
!=====================================================
crypto ikev2 proposal PROP1
 encryption aes-cbc-128
 integrity sha1
 group 2

!=====================================================
! IKEv2 POLICY (Fase 1 - Política)
!=====================================================
crypto ikev2 policy POLICY1
 proposal PROP1

!=====================================================
! IKEv2 KEYRING (Pre-Shared Key)
!=====================================================
crypto ikev2 keyring KEYRING1
 peer 172.16.1.2
  address 172.16.1.2
  pre-shared-key CISCO123

!=====================================================
! IKEv2 PROFILE (Autenticación)
!=====================================================
crypto ikev2 profile PROFILE1
 match identity remote address 172.16.1.2
 authentication remote pre-share
 authentication local pre-share
 keyring local KEYRING1
 lifetime 86400
 dpd 10 3 on-demand

!=====================================================
! IPSec TRANSFORM SET (Fase 2)
!=====================================================
crypto ipsec transform-set TS_PEER1 esp-aes 128 esp-sha-hmac
 mode tunnel

!=====================================================
! ACCESS LIST (Tráfico a encriptar)
!=====================================================
access-list 101 permit gre host 172.16.0.2 host 172.16.1.2

!=====================================================
! CRYPTO MAP (Vinculación)
!=====================================================
crypto map CMAP 10 ipsec-isakmp
 set peer 172.16.1.2
 set ikev2-profile PROFILE1
 set pfs group2
 set transform-set TS_PEER1
 match address 101

!=====================================================
! FIN DE CONFIGURACIÓN
!=====================================================
end
write memory

SCRIPT 3: R3_PEER2_CONFIG.txt
!=====================================================
! PARTE 2: IPSec IKEv2 Site-to-Site VPN
! Router: R3 (PEER2 - Sitio Remoto 2)
! Matrícula: 20241113
! Nombre: Alison Domeirys Caines Bautista
! Fecha: 29/06/2026
! Descripción: Configuración de PEER2 con IKEv2 (INVERSO)
!=====================================================

configure terminal

hostname PEER2
no ip domain-lookup

!=====================================================
! INTERFACES
!=====================================================
interface FastEthernet0/0
 description WAN to ISP
 ip address 172.16.1.2 255.255.255.0
 duplex full
 no ip route-cache
 crypto map CMAP
 no shutdown

interface GigabitEthernet1/0
 description LAN
 ip address 10.11.14.1 255.255.255.0
 no ip route-cache
 no shutdown

interface Tunnel0
 description GRE Tunnel to PEER1
 ip address 192.168.2.2 255.255.255.0
 tunnel source 172.16.1.2
 tunnel destination 172.16.0.2
 tunnel mode gre ip
 no shutdown

!=====================================================
! RUTAS ESTÁTICAS
!=====================================================
ip route 0.0.0.0 0.0.0.0 172.16.1.1
ip route 10.11.13.0 255.255.255.0 192.168.2.1

!=====================================================
! IKEv2 PROPOSAL (Fase 1 - Algoritmos)
!=====================================================
crypto ikev2 proposal PROP1
 encryption aes-cbc-128
 integrity sha1
 group 2

!=====================================================
! IKEv2 POLICY (Fase 1 - Política)
!=====================================================
crypto ikev2 policy POLICY1
 proposal PROP1

!=====================================================
! IKEv2 KEYRING (Pre-Shared Key)
!=====================================================
crypto ikev2 keyring KEYRING1
 peer 172.16.0.2
  address 172.16.0.2
  pre-shared-key CISCO123

!=====================================================
! IKEv2 PROFILE (Autenticación)
!=====================================================
crypto ikev2 profile PROFILE1
 match identity remote address 172.16.0.2
 authentication remote pre-share
 authentication local pre-share
 keyring local KEYRING1
 lifetime 86400
 dpd 10 3 on-demand

!=====================================================
! IPSec TRANSFORM SET (Fase 2)
!=====================================================
crypto ipsec transform-set TS_PEER2 esp-aes 128 esp-sha-hmac
 mode tunnel

!=====================================================
! ACCESS LIST (Tráfico a encriptar)
!=====================================================
access-list 101 permit gre host 172.16.1.2 host 172.16.0.2

!=====================================================
! CRYPTO MAP (Vinculación)
!=====================================================
crypto map CMAP 10 ipsec-isakmp
 set peer 172.16.0.2
 set ikev2-profile PROFILE1
 set pfs group2
 set transform-set TS_PEER2
 match address 101

!=====================================================
! FIN DE CONFIGURACIÓN
!=====================================================
end
write memory

SCRIPT 4: VERIFICATION_COMMANDS.txt
!=====================================================
! PARTE 2: COMANDOS DE VERIFICACIÓN
! IPSec IKEv2 Site-to-Site VPN
!=====================================================

!=====================================================
! VERIFICACIÓN EN PEER1 (R2)
!=====================================================

! 1. Ver sesión criptográfica
show crypto session

! 2. Ver estado de IKEv2
show crypto ikev2 sa brief

! 3. Ver encriptación activa
show crypto ipsec sa | include encaps

! 4. Ver crypto map
show crypto map

! 5. Ver tabla de rutas
show ip route

! 6. Verificar interfaces
show ip interface brief

! 7. Ping hacia PEER2 (WAN)
ping 172.16.1.2

! 8. Ping hacia túnel PEER2
ping 192.168.2.2

! 9. Ver configuración completa
show running-config

! 10. Ver tunnels
show interfaces Tunnel0

!=====================================================
! VERIFICACIÓN EN PEER2 (R3)
!=====================================================

! 1. Ver sesión criptográfica
show crypto session

! 2. Ver estado de IKEv2
show crypto ikev2 sa brief

! 3. Ver encriptación activa
show crypto ipsec sa | include encaps

! 4. Ver crypto map
show crypto map

! 5. Ver tabla de rutas
show ip route

! 6. Ping hacia PEER1 (WAN)
ping 172.16.0.2

! 7. Ping hacia túnel PEER1
ping 192.168.2.1

!=====================================================
! COMANDOS DE DEBUG (SI ES NECESARIO)
!=====================================================

! Debug IKEv2
debug crypto ikev2

! Debug IPSec
debug crypto ipsec

! Detener debug
undebug all

SCRIPT 5: README.md (Para GitHub)
markdown# IPSec IKEv2 Site-to-Site VPN - PARTE 2

## Información del Proyecto

- **Estudiante:** Alison Domeirys Caines Bautista
- **Matrícula:** 20241113
- **Materia:** Seguridad de Redes
- **Instructor:** Jonathan Rondon
- **Fecha:** 29 de Junio, 2026
- **Versión:** 1.0

## Descripción

Implementación de una VPN IPSec Site-to-Site utilizando **IKEv2** (Internet Key Exchange versión 2) con encriptación AES-128 y autenticación SHA-HMAC. El túnel GRE encapsula y IPSec encripta la comunicación entre dos sitios remotos.

## Topología
                      R1 (ISP)
                    172.16.0.1
                  172.16.1.1
                     /    \
                    /      \
                R2 (PEER1)  R3 (PEER2)
              172.16.0.2   172.16.1.2
              10.11.13.1    10.11.14.1
                  |            |
                IOU1          IOU2
             10.11.13.0    10.11.14.0
                  |            |
                PC1           PC2
        10.11.13.10      10.11.14.10
Túnel GRE: 192.168.2.0/24 (192.168.2.1 ↔ 192.168.2.2)

## Parámetros de Seguridad

| Aspecto | Valor |
|---------|-------|
| Protocolo | IKEv2 |
| Encriptación (Fase 2) | AES-128 (CBC) |
| Integridad | SHA-1 HMAC |
| Grupo Diffie-Hellman | Grupo 2 (1024 bits) |
| Pre-Shared Key | CISCO123 |
| Lifetime | 86400 segundos (24 horas) |
| DPD | 10 segundos / 3 intentos |
| PFS | Grupo 2 |
| Modo | Túnel |
