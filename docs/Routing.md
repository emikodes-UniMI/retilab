# Routing

Il Routing è il processo di selezione da parte di un router, del percorso migliore per instradare i pacchetti di dati attraverso una rete.

Per capire meglio in cosa consiste il routing, immaginiamo di modellare la rete come un grafo pesato $G=(V,E)$, dove $V$ è l’insieme dei router, $E$ l’insieme dei link, con i relativi costi associati.

Sotto questa rappresentazione il compito del routing è quello di determinare il cammino minimo tra due nodi sorgente e destinazione.

<aside>
<img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />

**Problema**: per poter calcolare un cammino minimo tra due nodi, ogni router deve avere una conoscenza (completa o limitata) della rete. Dove teniamo traccia di tale conoscenza?

</aside>

Un Router, è un sistema diviso concettualmente in due piani funzionali distinti:

- **Control Plane** - La CPU e la RAM eseguono un sistema operativo per router, che si occupa di gestire le configurazioni statiche e i protocolli di routing dinamico.
    
    L’output di questo piano è la **RIB (Routing Information Base), un database che racchiude la conoscenza che un router ha della rete.**
    
    E’ spesso ridondante (Potrebbe contenere più rotte diverse per raggiungere la stessa destinazione, per esempio perchè apprese secondo modalità/protocolli differenti), non ottimizzata per la velocità.
    
    Per questo, il control plane **“distilla” la RIB** in una seconda tabella detta **FIB (Forwarding Information Base), che contiene le sole rotte “migliori” per ogni destinazione.**
    
    Esempio di tabella di routing:
    
    | **Codice** | **Network Destination** | **Prefisso (Mask)** | **Next Hop / Gateway** | **Interface** | **AD** | **Metric** |
    | --- | --- | --- | --- | --- | --- | --- |
    | **C** | `192.168.0.0` | `/24` | *Directly Connected* | `Gig0/1` | 0 | 0 |
    | **L** | `192.168.0.100` | `/32` | *Local (My IP)* | `Gig0/1` | 0 | 0 |
    | **S** | `0.0.0.0` | `/0` | `10.10.10.1` | `Gig0/0` | 1 | 0 |
    | **O** | `172.16.0.0` | `/16` | `10.10.10.2` | `Gig0/0` | 110 | 20 |
    
    *(Nota: C=Connessa, L=Locale, S=Statica, O=OSPF)*
    
    Le rotte “migliori” da installare nella FIB, sono selezionate considerando in ordine:
    
    - Validità del next hop - Il router verifica se l’indirizzo del next hop è risolvibile: in caso negativo, la rotta viene scartata.
    - Longest Prefix Match - Supponiamo un router abbia due rotte per la stessa rete di destinazione:
        - Rotta A: `192.168.10.0/24` (comprende gli hosts da .0 a .255)
        - Rotta B: `192.168.10.0/28` (comprende gli hosts da .0 a .15)
        
        Arriva un pacchetto per `192.168.10.5`: entrambe le rotte sono valide, ma la Rotta B è più “specifica” (maschera di sottorete più lunga).
        
        Il router sceglierà **sempre** la rotta più specifica.
        
    - Distanza Amministrativa - A Parità di “precisione” di una rotta, si considera il valore del campo AD nella routing table.
        
        Questo numero indica al router “quanto è affidabile” la fonte da cui ha appreso tale rotta.
        
        - **0:** Connessione diretta
        - **1:** Rotta statica (Impostata dall’amministratore di rete)
        - **90:** EIGRP
        - **110:** OSPF
        - **200:** BGP
        
        Valori più bassi = affidabilità più elevata.
        
    - Metrica - Costo associato ad una rotta, in termini di numero di hops, larghezza di banda. ritardo o affidabilità del collegamento.
- **Data Plane** - Quando un pacchetto entra in un router, una porzione di hardware dedicata al forwarding sceglie dove inoltrarlo, riscrivendo l’header di livello 2 cambiando il MAC sorgente con il proprio (del router), ed il MAC destinazione col MAC corrispondente al next hop IP preso dalla FIB.
    
    Decrementa il TTL, ricalcola checksum, inoltra il pacchetto sull’interfaccia corrispondente al Next Hop IP.
    
    <aside>
    💡
    
    Questa divisione in livelli, permette una robusta resistenza agli errori: se il Control Plane crasha (il software di routing si blocca), il router può continuare a inoltrare pacchetti (compito del Data Plane) usando l'ultima FIB conosciuta.
    
    </aside>
    

Capito dove memorizziamo la conoscenza della rete, esistono due modalità con cui un router può apprenderla:

- Routing statico - L’amministratore di rete predispone manualmente i percorsi che i pacchetti dovranno seguire per arrivare ad una determinata rete, e li configura nel router sotto forma di voci nella RIB.
    
    Questo approccio prevede controllo totale sul traffico, oltre a prestazioni maggiori (I singoli router non hanno bisogno di calcolare da se le rotte del traffico, sono statiche ed impostate a priori).
    
    Tuttavia, scala molto male, sia in termini di complessità di configurazione iniziale (configurare rotte statiche su di una rete con 1000 router, è sicuramente dispensioso), sia in evoluzione (ogni modifica della rete, richiede riconfigurazione manuale delle rotte).
    
    <aside>
    <img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />
    
    Il Routing statico può essere comunque utile in reti grandi, come “piano di backup” in caso di crash dei protocolli di routing dinamico.
    Un amministratore di rete può impostare delle rotte statiche con AD “artificialmente” molto alto (es: 200), che verranno utilizzate solo in caso il routing dinamico fallisse.
    
    </aside>
    
- Routing dinamico - I Router automaticamente prendono conoscenza della topologia della rete, scambiandosi informazioni e calcolando i percorsi migliori.

---

## Routing Statico

![images/image.png](images/image%2041.png)

Nella topologia sopra illustrata, ogni coppia di router, appartiene ad una sottorete a se, con numero massimo di hosts pari a 2.

La sottomaschera di queste reti sarà /30 (Vogliamo configurare 2 hosts, riservando due indirizzi per base e broadcast, necessitiamo di un Host ID pari a 2 bit).

Vogliamo permettere la comunicazione tra i due PC nelle LAN verde e lilla.

Il Router 6 è a conoscenza delle reti verde e gialla (poichè direttamente connesso ad esse), ma non sa come raggiungere le reti azzurre e lilla.

Impostiamo due rotte statiche per esse:

**Sintassi del comando di configurazione di una rotta statica**: **`router(config)# ip route <Dst Net Base addr> <Dst Net mask> <Next hop IP addr>`**

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#ip route 192.168.0.4 255.255.255.252 192.168.0.2
Router(config)#ip route 192.168.10.0 255.255.255.0 192.168.0.2**
```

Router 5 conosce le reti gialla e azzurra, ma non conosce verde e lilla.

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#ip route 192.168.10.0 255.255.255.0 192.168.0.6
Router(config)#ip route 192.168.1.0 255.255.255.0 192.168.0.1**
```

Router 7 conosce le reti azzurra e lilla, ma non conosce verde e gialla.

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#ip route 192.168.0.0 255.255.255.252 192.168.0.5
Router(config)#ip route 192.168.1.0 255.255.255.0 192.168.0.5**
```

![images/image.png](images/image%2042.png)

Ogni router conosce ora l’intera topologia della rete, la comunicazione tra i due PC è consentita.

---

## Routing Dinamico

### RIP (Routing Information Protocol)

RIP è un protocollo **Distance Vector** basato sull'algoritmo di **Bellman-Ford**.
A differenza dei protocolli Link State, i router RIP **NON hanno una mappa completa della rete: c**onoscono solo il costo totale per raggiungere una destinazione e il prossimo salto (Next Hop).

Utilizza il protocollo di trasporto **UDP.**

3 Versioni:

- RIPv1
    
    **Routing Classful:** Non invia la Subnet Mask negli aggiornamenti di routing, per questo non supporta Subnetting **VLSM** (Variable Length Subnet Mask). E’ in grado di annunciare sole reti /8, /16 o /24 (Classi A,B,C pure).
    
    **Trasmissione:** Invia gli aggiornamenti in **Broadcast** (255.255.255.255) ogni 30 secondi.
    
    Questo disturba ogni dispositivo nel dominio di broadcast (PC, stampanti ricevono il pacchetto, la NIC invia il pacchetto al S.O, rileva che non c’è nessun SW in ascolto per pacchetti RIP, lo scartano).
    
    **Sicurezza:** Nessuna autenticazione. Chiunque colleghi un router pirata può iniettare rotte false e dirottare il traffico.
    
- RIPv2
    
    **Routing Classless:** Invia la **Subnet Mask** insieme all'indirizzo IP. Supporta pienamente **VLSM** e **CIDR**.
    
    **Trasmissione:** Trasmette gli aggiornamenti in multicast ai soli router RIP (Gruppo multicast)
    
    **Sicurezza:** Supporta autenticazione (MD5 o Clear Text). Il router accetta aggiornamenti solo se la password corrisponde.
    
- RIPng
    
    Aggiunge supporto ad IPv6
    

### **Funzionamento di RIP:**

![images/image.png](images/image%2043.png)

Analizziamo la topologia in figura: all’avvio, ogni router riempie la propria tabella di routing inserendo le informazioni relative alle reti direttamente connesse. (Type = C (Connected), Metric=0)

![images/image.png](images/image%2044.png)

Ogni 30 secondi, i router inviano e ricevono un’informazione ridotta della tabella di routing dei vicini, che contiene:

- Reti conosciute
- Costo (Metrica, espressa in “Numero di router da attraversare” per poter arrivare alla rete destinazione)

Per ogni destinazione nella tabella del vicino, il router calcola:
$HopCount_{new} = HopCount_{vicino} + 1$
(Dove "+1" è il costo per raggiungere il vicino stesso).

Fatto questo calcolo, applica due regole logiche:

- **Discovery**
    
    Se la destinazione che sto analizzando, è sconosciuta per la mia tabella di routing, procedo ad “impararla”.
    
    Aggiungo alla mia tabella una entry contenente la rete che sto imparando, l’interfaccia che mi collega al vicino, la metrica calcolata.
    
- **Update**
    
    Se la destinazione che sto analizzando è già conosciuta, ma la metrica calcolata è minore, vuol dire che ho appreso una nuova strada più veloce.
    
    Aggiorno la rotta con il nuovo valore per HopCount, ed impostando come next hop il vicino dal quale ho ricevuto la tabella.
    

![images/image.png](images/image%2045.png)

Ora ogni router ha le informazioni necessarie per comunicare con l’intera rete.

Provando a trasmettere un messaggio tra PC2 e PC0, esso seguirà il percorso: PC2 → Router2 → Router0→ PC0

![images/image.png](images/image%2046.png)

Simuliamo un guasto tra Router2 e Router0

![images/image.png](images/image%2047.png)

Viene immediatamente segnalato il guasto del link (Triggered Update, senza dover aspettare il timer standard di 30 secondi), attraverso lo scambio di informazioni standard. 

Router2 Aggiorna la propria rotta verso la rete del PC0, impostando Router1 come Next Hop.

### **Configurazione RIPv1 in Packet Tracer:**

Situazione iniziale: Ogni router conosce le sole reti direttamente connesse.

![images/image.png](images/image%2048.png)

Partiamo dal Router0. Esso dovrà annunciare la conoscenza della rete 192.168.1.0 (Quella in cui si trova PC0), e della rete 192.168.0.0 (Quella che collega i 3 router assieme).

<aside>
<img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />

RIPv1 è classful, pertanto dobbiamo annunciare l’intera rete 192.168.0.0/24, e non le singole sottoreti che collegano le coppie di router.

</aside>

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#network 192.168.1.0
Router(config-router)#network 192.168.0.0
Router(config-router)#passive-interface GigabitEthernet 2/0**
```

Il comando **`router rip`** ordina al sistema operativo di avviare il demone software di **RIP**. Di default, la versione eseguita è RIPv1.

Col comando **`network A.B.C.D`**, il demone RIP esegue due azioni:

- Scansiona tutte le porte fisiche del router, rileva le interfacce associate alla rete indicata, e le attiva per la ricezione di pacchetti RIP.
- Comincia ad annunciare tale rete ai vicini.

Eseguendo il comando **`network 192.168.1.0`**, il router comincerà quindi contemporaneamente ad annunciare la conoscenza di tale rete ai vicini (corretto), ma anche ad inoltrare i pacchetti RIP all’interno della LAN (comportamento indesiderato).

Con **`passive-interface GigabitEthernet 2/0`**, dove **`GigabitEthernet 2/0`** è l’interfaccia associata alla LAN 192.168.1.0, indichiamo al router di smettere di inviare pacchetti RIP ad essa, pur continuando ad annunciare la rete agli altri router.

Ripetiamo per Router1:

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#network 192.168.10.0
Router(config-router)#network 192.168.0.0
Router(config-router)#passive-interface GigabitEthernet 1/0**
```

Router2:

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#network 192.168.20.0
Router(config-router)#network 192.168.0.0
Router(config-router)#passive-interface Ethernet 2/0**
```

![images/image.png](images/image%2045.png)

I Router hanno correttamente cominciato a scambiarsi messaggi RIP.

### **Configurazione RIPv2 in Packet Tracer:**

Operiamo sulla stessa topologia indicata sopra.

RIPv2 è classless, dobbiamo quindi annunciare singolarmente tutte le sottoreti (anche le /30 tra router).

Router0:

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#version 2
Router(config-router)#network 192.168.1.0
Router(config-router)#network 192.168.0.4
Router(config-router)#network 192.168.0.0**
```

Router1:

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#version 2
Router(config-router)#network 192.168.10.0
Router(config-router)#network 192.168.0.0
Router(config-router)#network 192.168.0.8**
```

Router2:

```
**Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#router rip
Router(config-router)#version 2
Router(config-router)#network 192.168.0.4
Router(config-router)#network 192.168.0.8
Router(config-router)#network 192.168.20.0**
```

<aside>
<img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />

Nota! RIPv2 trasmette gli aggiornamenti in modalità multicast.
La trasmissione in modalità multicast, implica comunque che i pacchetti vengono inoltrati in modalità flooding su tutte le interfacce, ma vengono scartati dalla NIC del destinatario (dopo un check sul MAC destinatario).

I PC nella LAN quindi ricevono comunque i pacchetti RIP, e seppur scartati dalla NIC, un utente smaliziato potrebbe usare un analizzatore di rete (es: Wireshark) per catturare i pacchetti RIP e acquisire conoscenza della rete.

**E’ pertanto comunque consigliato eseguire il comando passive-interface anche in RIPv2!**

</aside>

![images/image.png](images/image%2049.png)

---