# Configuració d'IPsec en R1 i R3

# Contrasenyes
Contra per la linea: ciscoconpa55

Contra per les linies VTY: ciscovtypa55

Contra enable: ciscoenpa55

Usuari i contra SSH: SSHadmin / ciscosshpa55

### Pas 1: Comprovar la connectivitat

Si vols executa un ping des de PC-A cap a PC-C per verificar la connectivitat.

### Pas 2: Activar el paquet de tecnologia de seguretat

a. A R1, executa la comanda següent per veure la informació de la llicència del paquet de seguretat:

```bash
show version
```

b. Si el paquet de seguretat no està activat, activa'l amb la comanda següent:

```bash
R1(config)# license boot module c1900 technology-package securityk9
```

c. Accepta l'acord de llicència d'usuari final.

d. Desa la configuració actual i reinicia el router per aplicar els canvis.

e. Verifica que el paquet de seguretat s'ha activat:

```bash
show version
```

### Pas 3: Identificar el trànsit interessant en R1

Configura l'ACL 110 per identificar el trànsit entre la LAN de R1 i la LAN de R3 com a interessant:

```bash
R1(config)# access-list 110 permit ip 192.168.1.0 0.0.0.255 192.168.3.0 0.0.0.255
```

### Pas 4: Configurar la fase 1 d'IKE (ISAKMP) en R1

Configura la política ISAKMP amb els següents paràmetres:

```bash
R1(config)# crypto isakmp policy 10
R1(config-isakmp)# encryption aes 256
R1(config-isakmp)# authentication pre-share
R1(config-isakmp)# group 5
R1(config-isakmp)# exit
R1(config)# crypto isakmp key vpnpa55 address 10.2.2.2
```

### Pas 5: Configurar la fase 2 d'IPsec en R1

a. Crear el conjunt de transformació VPN-SET:

```bash
R1(config)# crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
```

b. Crear el mapa criptogràfic VPN-MAP:

```bash
R1(config)# crypto map VPN-MAP 10 ipsec-isakmp
R1(config-crypto-map)# description VPN connection to R3
R1(config-crypto-map)# set peer 10.2.2.2
R1(config-crypto-map)# set transform-set VPN-SET
R1(config-crypto-map)# match address 110
R1(config-crypto-map)# exit
```

### Pas 6: Assignar el mapa criptogràfic a la interfície de sortida

```bash
R1(config)# interface s0/0/0
R1(config-if)# crypto map VPN-MAP
```

## Part 2: Configuració d'IPsec en R3

### Pas 1: Activar el paquet de seguretat

a. A R3, verifica si el paquet de seguretat està activat:

```bash
show version
```

b. Si no està activat, activa'l i reinicia el router.

### Pas 2: Configurar la VPN entre R3 i R1

Defineix el trànsit interessant en R3:

```bash
R3(config)# access-list 110 permit ip 192.168.3.0 0.0.0.255 192.168.1.0 0.0.0.255
```

### Pas 3: Configurar la fase 1 d'IKE en R3

```bash
R3(config)# crypto isakmp policy 10
R3(config-isakmp)# encryption aes 256
R3(config-isakmp)# authentication pre-share
R3(config-isakmp)# group 5
R3(config-isakmp)# exit
R3(config)# crypto isakmp key vpnpa55 address 10.1.1.2
```

### Pas 4: Configurar la fase 2 d'IPsec en R3

a. Crear el conjunt de transformació VPN-SET:

```bash
R3(config)# crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
```

b. Crear el mapa criptogràfic VPN-MAP:

```bash
R3(config)# crypto map VPN-MAP 10 ipsec-isakmp
R3(config-crypto-map)# description VPN connection to R1
R3(config-crypto-map)# set peer 10.1.1.2
R3(config-crypto-map)# set transform-set VPN-SET
R3(config-crypto-map)# match address 110
R3(config-crypto-map)# exit
```

### Pas 5: Assignar el mapa criptogràfic a la interfície de sortida

```bash
R3(config)# interface s0/0/1
R3(config-if)# crypto map VPN-MAP
```

## Part 3: Verificació de la VPN d'IPsec

### Pas 1: Verificar el túnel abans del trànsit interessant

Executa la comanda següent a R1:

```bash
show crypto ipsec sa
```

Els comptadors de paquets hauran de ser 0.

### Pas 2: Generar trànsit interessant

Fes un ping des de PC-A cap a PC-C.

### Pas 3: Verificar el túnel després del trànsit interessant

Executa novament la comanda a R1:

```bash
show crypto ipsec sa
```

Si el túnel funciona correctament, els comptadors hauran augmentat.

### Pas 4: Generar trànsit no interessant

Fes un ping des de PC-A cap a PC-B.

### Pas 5: Verificar el túnel

Executa la comanda següent a R1:

```bash
show crypto ipsec sa
```

Els comptadors no haurien de canviar, verificant que el trànsit no interessant no s'encripta.

### Pas 6: Comprovar els resultats

Si la configuració és correcta, la finalització hauria de ser del 100%. Fes clic a "Check Results" per veure la verificació dels components requerits.

