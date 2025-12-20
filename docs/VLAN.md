# VLAN

Le VLAN (Virtual Local Area Network) permettono di segmentare la rete Ethernet in più reti Layer 2 separate, senza switch distinti.
Permettono quindi di separare logicamente in più domini di broadcast, un gruppo di host connesso mediante la stessa infrastruttura fisica.

Voglio dividere un gruppo di hosts in due reti logiche, al fine di isolarne il traffico. Identifichiamo due gruppi, verde e arancio.

## Soluzione 1:

![images/image.png](images/image%2017.png)

Collego allo switch a SX solo i PC del gruppo arancione, e allo switch di destra solo i PC del gruppo verde. Per far si che due PC in LAN diverse possano comunicare, introduciamo un router per la comunicazione a livello 3.

Questa soluzione è poco scalabile: e se i PC appartenenti ad uno stesso gruppo si trovassero in uffici diversi? Dovrei tirare dei cavi lunghissimi.

## Soluzione 2:

![images/image.png](images/image%2018.png)

Collego gli switch con tanti cavi quante sono le VLAN. Ogni porta sullo switch, è configurata in modalità ACCESS. Su ogni cavo che collega due switch, viaggerà il traffico relativo ad una sola specifica VLAN.

Anche questa soluzione scala male all’aumentare del numero di VLAN.

## Trunking IEEE802.1Q

Lo standard IEEE 802.1Q definisce il VLAN Trunking, che permette al traffico di più VLAN di viaggiare lungo un singolo collegamento fisico (trunk).

![images/image.png](images/image%2019.png)

![images/image.png](images/image%2020.png)

802.1Q aggiunge 4 byte all’header di ogni frame ethernet, al fine di identificarne la VLAN di appartenenza (Frame tagging):

![images/image.png](images/image%2021.png)

- I primi 2 byte riguardano il *tag protocol identifier (***TPID)**.
    
    Il suo valore è impostato a 0x8100, ad indicare che il frame trasmesso è in formato IEEE 802.1Q.
    
- I successivi 2 byte riguardano il *tag control information* **TCI** (detto anche [**VLAN](https://it.wikipedia.org/wiki/VLAN) Tag**) così suddiviso:
    - **Priority Code Point (PCP)**: Questo campo a 3 bit può essere utilizzato per indicare un livello di priorità per il frame. L'utilizzo di questo campo è definito in: [IEEE 802.1p](https://it.wikipedia.org/wiki/IEEE_802.1p).
    - **Drop eligible indicator (DEI)**: (in precedenza CFI) Campo di 1 bit che indica la possibilità di ignorare il frame in caso di congestione.
    - **VLAN ID (VID)**: campo di 12 bit che indica l'ID delle VLAN (fino a 4096 VLAN identificabili)
        
        Di queste, la prima (VLAN 0) e l'ultima (VLAN 4095) sono riservate
        

Il resto del frame Ethernet rimane identico all'originale.

Capiamo allora che ogni interfaccia di uno switch, può operare in due modalità:

- Access - Su quell’interfaccia viaggia il traffico destinato ad una sola VLAN (frame untagged).
- Trunk - Accettano e trasmettono frame tagged, gestiscono traffico da/a più VLAN (IEEE 802.1Q)

## Configurazione Packet Tracer

Realizziamo la topologia in figura:

![images/image.png](images/image%2022.png)

**Configurazione switch di sinistra:**

```
**enable** # accesso alla modalità privilegiata
**configure terminal** # accesso alla configurazione globale

***Aggiungiamo le VLAN al database dello switch***

**vlan 10** # aggiunta della VLAN con ID 10 al database dello switch
**name verde1**0 # assegnazione del nome alla VLAN 10 (sub mode vlan)

**exit** # torna al livello configurazione globale

**vlan 20** # aggiunta della VLAN con ID 20 al database dello switch
**name arancio20** # assegnazione del nome alla VLAN 20 (submode vlan)
**exit** # torna al livello configurazione globale

***Configuriamo le interfacce in modalità access (verso gli host)***

**interface Fastethernet 0/1** # accesso alla submode di conf. dell’interfaccia 0/1
**switchport mode access** # configurazione dell’interfaccia in modalità access
**switchport access vlan 20** # indicazione della VLAN presente su quella porta

exit # torna al livello configurazione global

**interface Fastethernet 1/1** # accesso alla submode di conf. dell’interfaccia 1/1
**switchport mode access** # configurazione dell’interfaccia in modalità access
**switchport access vlan 10** # indicazione della VLAN presente su quella porta

**exit** # torna al livello configurazione globale

***Configuriamo l'interfaccia trunk (verso l'altro switch)*

interface Fastethernet 2/1** # accesso alla submode di conf. dell’interfaccia 2/1
**switchport mode trunk** # configurazione dell’interfaccia in modalità trunk
**switchport trunk allowed vlan 10** # consentire il passaggio del traffico della VLAN 10
**switchport trunk allowed vlan add 20** # aggiunta della VLAN 20 a quelle permesse

**exit** # torna al livello configurazione globale
**exit** # esci dalla configurazione globale
**write** # salva
```

**Configurazione switch di destra:**

```
Switch>enable
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#vlan 10
Switch(config-vlan)#name verde10
Switch(config-vlan)#exit
Switch(config)#vlan 20
Switch(config-vlan)#name arancio20
Switch(config)#interface FastEthernet0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 20
Switch(config-if)#exit
Switch(config)#interface FastEthernet1/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#exit
Switch(config)#interface FastEthernet2/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#exit
Switch(config)#interface FastEthernet3/1
Switch(config-if)#switchport mode trunk
Switch(config-if)#switchport trunk allowed vlan 10
Switch(config-if)#switchport trunk allowed vlan add 20
Switch(config-if)#exit
Switch(config)#exit
Switch#
%SYS-5-CONFIG_I: Configured from console by console
write
Building configuration...
[OK]
```

## Routing Inter-VLAN

Il Routing Inter-VLAN permette la comunicazione tra due o più VLAN. Per fare ciò, è necessario un dispositivo di livello 3 (Che svolga la funzionalità di Routing, quindi un Router o Switch di livello 3).

Ad ogni VLAN, è assegnato un Indirizzo IP “Gateway”, che permette al traffico di uscire al di fuori del dominio di broadcast.

### Metodo 1: Collegamenti fisici dedicati

![images/image.png](images/image%2023.png)

![images/image.png](images/image%2024.png)

Per ogni VLAN, si riserva un’interfaccia sul router (ed un conseguente link fisico) dedicata, il cui indirizzo IP sarà il default gateway per la singola VLAN.

Soluzione poco scalabile, utilizzata solo in caso di gestione di 2/3 VLAN.

### Metodo 2: Router on a stick

![images/image.png](images/image%2025.png)

Si utilizza un singolo link fisico (Stick, o Trunk Link) tra switch e router, per gestire il traffico di tutte le VLAN.

![images/image.png](images/image%2026.png)

Questo approccio si basa sulla capacità del router di creare **sottointerfacce** logiche su una singola interfaccia fisica, ciascuna delle quali agisce come un gateway per una VLAN specifica.

La porta dello switch a cui è collegato il router, deve essere configurata in modalità Trunk, per poter veicolare il traffico di più VLAN contemporaneamente.

L’interfaccia fisica del router, viene divisa logicamente in sottointerfacce.

Nel disegno, l’interfaccia FastEthernet0/0, viene divisa in FastEthernet0/0.50 per la VLAN 50, FastEthernet0/0.60 per la VLAN 60,FastEthernet0/0.70 per la VLAN 70.

Ognuna, viene configurata con un indirizzo IP, che funge da default gateway per la sua specifica VLAN.

**Esempio di comunicazione:**

![images/image.png](images/image%2027.png)

Il PC “Source” (IP: 192.168.1.3), nella VLAN “Magazzino” (ID 10), vuole inviare un pacchetto ICMP (Protocollo di livello 3, pertanto verrà generato un pacchetto IP) al PC Destination (IP: 192.168.0.4), nella VLAN “Uffici” (ID 20).

![images/image.png](images/image%2028.png)

“Source”, verificando l’IP destinazione con la subnet mask, rileva che non è nella sua stessa rete. 

Imposta il campo DEST ADDR, con l’indirizzo MAC del suo default gateway (DST IP rimane l’indirizzo IP di “Destination”). 

Lo switch riceve il frame (Livello 2).

Esiste già una corrispondenza MAC-Interface, pertanto non aggiorna la tabella ARP.

![images/image.png](images/image%2029.png)

Supponendo lo switch abbia già in memoria la corrispondenza tra il MAC del default gateway e la relativa interfaccia, esso saprà di dover inoltrare il frame su un link Trunk: il frame viene taggato con l’ID 10 della VLAN “Magazzino”.

Il Router riceve il pacchetto (Livello 3).

Controlla l’ID della VLAN: L’interfaccia FastEthernet0/0.10 accetta frame da quella VLAN.

All’interno dell’header, il MAC Address di destinazione è il suo, ma l’indirizzo IP no. Il Router così sa che questo pacchetto non è destinato a lui, ma dovrà inoltrarlo.

![images/image.png](images/image%2030.png)

Attraverso un lookup della tabella ARP, riconosce una corrispondenza tra l’indirizzo IP destinazione, e la coppia MAC Address e interfaccia.

Il Router sostituisce il MAC Address del PC “Destination”, inserisce l’ID 20 per la VLAN (Questo viene ricavato dall’ID associato all’interfaccia), e inoltra il pacchetto all’interfaccia FastEthernet0/0.20.

Lo switch riceve nuovamente indietro il frame. Questo è taggato con l’ID 20. 

Controlla la MAC Table, ha una corrispondenza per l’indirizzo destinazione.

Rimuove il tag VLAN, e inoltra correttamente il frame untagged a destinazione.

**Configurazione Packet Tracer:**

- Configurare l’interfaccia che va dallo switch al router in modalità Trunk per tutte le VLAN.
- Sul Router, suddividere l’interfaccia connessa, e impostarla come default gateway per ogni VLAN:
    
    ```
    **Router(config)# interface FastEthernet 0/0.20** # Suddivido l'interfaccia logica 0/0, nell'interfaccia 0/0.20
    **Router(config-subif)# encapsulation dot1Q 20 #** Abilito incapsulamento 802.1Q. Questa interfaccia logica, sarà dedicata alla VLAN 20.
    **Router(config-subif)# ip address <addr> <netmask>** # Imposto questa interfaccia come default gateway per la VLAN 20, assegnandone un indirizzo IP.
    **Router(config-subif)# no shutdown** # Accendo interfaccia
    **Router(config-subif)# exit**
    ```
    
    Ripeto per tutte le VLAN.
    
