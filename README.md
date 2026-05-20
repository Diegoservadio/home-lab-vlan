# Router-on-a-Stick - Segmentazione di Rete con VLAN

## Obiettivo
Simulare una rete aziendale con due segmenti separati (Ufficio e Server)
usando VLAN e un singolo router per il routing inter-VLAN.

## Topologia
- 1 Router (R1)
- 1 Switch Core (SW-Core)
- 1 Switch Ufficio (SW-Ufficio) - VLAN 10
- 1 Switch Server (SW-Server) - VLAN 20
- 4 PC (2 per VLAN)

![Topologia](topologia.png)

## Schema IP

| Dispositivo     | IP             | VLAN |
|-----------------|----------------|------|
| PC-Ufficio-1    | 192.168.10.2   | 10   |
| PC-Ufficio-2    | 192.168.10.3   | 10   |
| PC-Server-1     | 192.168.20.2   | 20   |
| PC-Server-2     | 192.168.20.3   | 20   |
| Gateway VLAN 10 | 192.168.10.1   | 10   |
| Gateway VLAN 20 | 192.168.20.1   | 20   |

![router-interfaces](router-interfaces.png)

## Concetti chiave

### Perche le VLAN?
Le VLAN separano il traffico a livello logico anche su una rete fisica condivisa.
I PC nella VLAN 10 non possono comunicare con quelli nella VLAN 20 senza passare dal router.

### Cos e Router-on-a-Stick?
Tecnica che permette il routing inter-VLAN usando una sola interfaccia fisica del router,
divisa in sottointerfacce virtuali - una per ogni VLAN. Il traffico viaggia taggato (802.1Q)
su un link trunk verso lo switch core.

### Switch vs Router
Lo switch lavora a livello 2 (MAC address) e gestisce il traffico dentro la stessa VLAN.
Il router lavora a livello 3 (IP) ed e l unico che puo instradare traffico tra VLAN diverse.

## Strumenti
- Cisco Packet Tracer
- Protocollo 802.1Q (trunk)
- Routing inter-VLAN con sottointerfacce

## DHCP

Il router R1 funge da server DHCP per entrambe le VLAN.
Gli indirizzi vengono assegnati automaticamente ai PC al momento della connessione.

| Pool | Network | Gateway | Range assegnabile |
|------|---------|---------|-------------------|
| VLAN10-Ufficio | 192.168.10.0/24 | 192.168.10.1 | .2 - .254 |
| VLAN20-Server | 192.168.20.0/24 | 192.168.20.1 | .2 - .254 |

Il gateway e escluso dal pool con il comando `ip dhcp excluded-address`
per evitare conflitti di indirizzamento.

![DHCP Test](dhcp-test.png)

## ACL - Protezione del Server

E stata configurata una ACL extended su R1 per proteggere SRV-1 (192.168.20.4)
dal traffico HTTP proveniente dalla VLAN 10.

| Regola | Protocollo | Sorgente | Destinazione | Porta | Azione |
|--------|------------|----------|--------------|-------|--------|
| 1 | TCP | 192.168.10.0/24 | 192.168.20.4 | 80 | DENY |
| 2 | IP | any | any | any | PERMIT |

La ACL e applicata sulla sottointerfaccia GigabitEthernet0/0.10 in direzione inbound.

Risultato:
- I PC della VLAN 10 non possono accedere al server web (HTTP bloccato)
- Il ping verso SRV-1 funziona regolarmente (ICMP permesso)
- Il traffico tra VLAN continua a funzionare normalmente