# DHCP

DHCP (Dynamic Host Configuration Protocol) è un protocollo applicativo per la configurazione automatica dei parametri di rete degli host in una rete locale.

Obiettivo pratico: permettere a un client senza indirizzo (o con indirizzo temporaneo) di ricevere indirizzi e parametri in modo scalabile e centralizzato.

Esempio: il computer “pippo” si collega fisicamente alla rete (Cavo Ethernet, o via etere).

![images/image.png](images/image%2031.png)

Nello stato iniziale esso non ha un indirizzo IP, potrà inviare solo frame ethernet a livello 2 (datalink, identificandosi utilizzando l’indirizzo MAC).

1. Il software DHCP in esecuzione sul PC “pippo”, rileva la connessione e comincia la trasmissione di frame ethernet DHCP DISCOVER all’indirizzo fisico di broadcast FF:FF:FF:FF:FF:FF, con l’intento di identificare un eventuale server DHCP presente sulla rete.
2. Il server DHCP raggiungo da un pacchetto DHCP DISCOVER, risponde con un frame ethernet DHCP OFFER, inoltrato anch’esso in modalità broadcast, contenente:
    - Indirizzo IP (libero, non in uso) proposto per il client
    - Maschera di sottorete (Subnet Mask)
    - Default gateway
    - Indirizzo del server DNS
    - Lease duration
    - Altri eventuali parametri specifici della rete (Esempio: nome di dominio di default)
3. Il client “pippo”, ricevuta la configurazione, invia in broadcast una DHCP REQUEST, chiedendo di riservare per lui l’indirizzo indicato.
4. Il server DHCP risponde con un frame ethernet inviato in broadcast, con un DHCP ACK, per confermare al client che può procedere a salvare la configurazione di rete pattuita.

![images/image.png](images/image%2032.png)

Al termine della procedura, il client sarà libero di utilizzare l’indirizzo IP fornitogli per un quanto di tempo “Lease time”.

A metà del Lease time, il client chiederà di rinnovare il permesso di utilizzare quell’indirizzo, mandando al server DHCP una nuova DHCP REQUEST, trasmessa in Unicast, a cui il server risponderà con un DHCP ACK, sempre in Unicast al client.

![images/image.png](images/image%2033.png)

L’indirizzo IP viene rilasciato o esplicitamente dal client, quando esso si disconnette dalla rete, tramite un pacchetto DHCP RELEASE, oppure alla scadenza del time lease.

![images/image.png](images/image%2034.png)

## Configurazione Packet Tracer:

- Aggiungere un server nella LAN, ed assegnargli dal menu “Config”, un indirizzo IP statico, subnet mask, e default gateway.
    
    Se il server è connesso ad uno switch, la relativa interfaccia sullo switch deve essere configurato in modalità Access.
    

![images/image.png](images/image%2035.png)

![images/image.png](images/image%2036.png)

- Attivare il servizio DHCP sul server, e creare un pool (o modificare il serverPool di default) in cui specifichiamo le impostazioni comuni della rete, da assegnare ad ogni host al suo interno.
    
    ![images/image.png](images/image%2037.png)
    
- Ora, ogni host della rete in cui si trova il server DHCP, potrà essere automaticamente configurato.
    
    ![images/image.png](images/image%2038.png)
    

## DHCP Relay Agent (Proxy)

![images/image.png](images/image%2039.png)

Vogliamo ora permettere anche agli host nella VLAN blu di utilizzare il servizio DHCP.

Utilizziamo un DHCP Relay Agent, un dispositivo di rete intermediario tra il server DHCP ed i client, che permette di inoltrare le richieste DHCP dei client in una rete, ad un server DHCP appartentente ad un’altra rete.

> The concept of DHCP proxy emerged as networks grew in complexity and size. Originally, DHCP was designed for simple, flat networks where clients and servers resided on the same segment. However, as networks expanded, spanning multiple segments and subnets, the need for a more versatile DHCP mechanism became apparent. The DHCP proxy was developed to bridge this gap, facilitating DHCP communication across different network segments, which was previously unattainable with standard DHCP operations.
> 

Come funziona:

- Un client della rete A invia un frame DHCP DISCOVER in broadcast.
- Il DHCP Relay Agent della rete A riceve il frame, e inoltra un nuovo pacchetto contenente:
    - Indirizzo IP di destinazione pari a quello del server DHCP
    - Indirizzo IP mittente pari al proprio IP
    - Un informazione relativa alla rete in cui si trova il client che ha richiesto il servizio DHCP: questa viene generalmente ricavata dall’IP dell’interfaccia da cui il Relay Agent ha ricevuto il frame DHCP DISCOVER
- Il Server DHCP riceve il pacchetto, tramite l’informazione della rete del client, consulta l’elenco dei propri pool e procede a proporre una configurazione di rete (DHCP OFFER), inoltrandola al Relay Agent.
- Il Relay Agent inoltra l’offer al client

Il processo **DORA** (Discover, Offer, Request, Ack) continua passando per il relay agent.

## Configurazione Packet Tracer

Per far si che un router operi come DHCP Proxy:

- Nella modalità di configurazione della sotto-interfaccia della VLAN servita (Quella in cui non è presente il server DHCP)
    
    ```
    **Router(config-subif)# ip helper-address <indirizzo DHCP server>**
    ```
    
    Ora anche i client della rete blu (192.168.0.0) potranno utilizzare il servizio DCHP.
    
    ![images/image.png](images/image%2040.png)
    