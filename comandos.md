# R1 · Comercial Andina

Dos versiones de la configuración de R1: una con **RIPv2** y otra con **enrutamiento estático**. Usa solo una de las dos.

---

## R1 con RIPv2

```
enable
configure terminal
hostname R1
no ip domain-lookup

! ================================================
! SEGURIDAD BASICA - R1
! enable secret  -> clave cifrada para entrar a modo privilegiado
! line console 0 -> clave al conectarse por cable de consola
! line vty 0 4   -> clave para acceso remoto (Telnet)
! service password-encryption -> cifra las claves en el running-config
! banner motd    -> mensaje de aviso al conectarse
! ================================================
enable secret cisco123

line console 0
 password cisco123
 login
 exit

line vty 0 4
 password ciscovyt
 login
 exit

service password-encryption
banner motd #Acceso solo para personal autorizado de Comercial Andina#

interface g0/0
 description LAN A - Administracion
 ip address 192.168.10.1 255.255.255.128
 no shutdown
 exit

interface s0/0/0
 description WAN1 hacia R2
 ip address 192.168.10.241 255.255.255.252
 no shutdown
 exit

interface s0/0/1
 description WAN3 hacia R3
 ip address 192.168.10.249 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! ================================================
! ENRUTAMIENTO DINAMICO RIPv2 - R1
! version 2         -> envia la mascara (soporta VLSM)
! no auto-summary   -> no resume a /24, respeta las subredes
! network           -> activa RIP en todas las interfaces 192.168.10.x
! passive-interface -> no envia actualizaciones hacia la LAN A (solo PCs)
! ================================================
router rip
 version 2
 no auto-summary
 network 192.168.10.0
 passive-interface g0/0
 exit

end
write
```

Verificación: `show ip route rip` y `show ip protocols`. Para revisar claves y banner: `show running-config`.

---

## R1 con enrutamiento estático

> Si R1 ya tenía RIP, ejecuta primero `no router rip` en modo `configure terminal`.

```
enable
configure terminal
hostname R1
no ip domain-lookup

! ================================================
! SEGURIDAD BASICA - R1
! enable secret  -> clave cifrada para entrar a modo privilegiado
! line console 0 -> clave al conectarse por cable de consola
! line vty 0 4   -> clave para acceso remoto (Telnet)
! service password-encryption -> cifra las claves en el running-config
! banner motd    -> mensaje de aviso al conectarse
! ================================================
enable secret cisco123

line console 0
 password cisco123
 login
 exit

line vty 0 4
 password ciscovyt
 login
 exit

service password-encryption
banner motd #Acceso solo para personal autorizado de Comercial Andina#

interface g0/0
 description LAN A - Administracion
 ip address 192.168.10.1 255.255.255.128
 no shutdown
 exit

interface s0/0/0
 description WAN1 hacia R2
 ip address 192.168.10.241 255.255.255.252
 no shutdown
 exit

interface s0/0/1
 description WAN3 hacia R3
 ip address 192.168.10.249 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! ================================================
! ENRUTAMIENTO ESTATICO - R1
! Redes conectadas: LAN A, WAN 1, WAN 3
! Redes que faltan: LAN B, LAN C, LAN D, WAN 2
!
! 1. LAN B - Ventas  192.168.10.128/26 -> por R2 (.242)
! 2. WAN 2 (R2-R3)   192.168.10.244/30 -> por R2 (.242)
! 3. LAN C - Bodega  192.168.10.192/27 -> por R3 (.250)
! 4. LAN D - Soporte 192.168.10.224/28 -> por R3 (.250)
! ================================================
ip route 192.168.10.128 255.255.255.192 192.168.10.242
ip route 192.168.10.244 255.255.255.252 192.168.10.242
ip route 192.168.10.192 255.255.255.224 192.168.10.250
ip route 192.168.10.224 255.255.255.240 192.168.10.250

end
write
```

Verificación: `show ip route static` (deben salir 4 rutas **S**). Para revisar claves y banner: `show running-config`.
