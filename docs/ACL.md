Le **ACL (Access Control Lists)** sono elenchi di regole applicabili alle interfacce di un router, per definire la tipologia di traffico da scartare/inoltrare sia in uscita che in entrata.

Per definire quali pacchetti scartare e quali inoltrare, è possibile analizzare i seguenti criteri:

- Indirizzo IP **Sorgente (Concedere solo determinati mittenti fidati)**
- Indirizzo IP **Destinazione**
- **Protocollo:** TCP, UDP, ICMP (Ping).
- **Porta Sorgente o Destinazione:** 80 (HTTP), 443 (HTTPS), 23 (Telnet), 53 (DNS).
- **Flag TCP:** (es. bloccare solo l'inizio di una connessione SYN, ma permettere le risposte ACK -  Keyword `Established`).

Tipologie di ACL configurabili:

- ACL Standard (Identificate da un numero compreso tra 1 e 99) - Filtrano il traffico esclusivamente in base all’indirizzo IP sorgente.
    
    **Sintassi:** `access-list <ID 1-99> {permit|deny} <sorgente|wildcard-mask|any>`
    
- ACL Extended (ID 100-199) - Filtraggio per IP Sorgente e/o destinazione, protocollo di livello 3 o 4, numeri di porta, `Established`.
    
    **Sintassi:**`access-list <ID 100-199> {permit|deny} <protocollo> <sorgente|wildcard|any> <destinazione|wildcard|any> [eq/neq/lt/gt/range <porta/range porte>] [established] [log]`
    
- ACL Extended Named - Le ACL Standard ed Extended hanno una limitazione: sono estensibili, ma non modificabili. Ciò significa che se ho sbagliato ad impostare una regola di filtraggio, non è possibile modificarla, devo cancellare l’ACL e ricrearla.
    
    Sintassi:  `ip access-list extended NOME`
    
    Le ACL Named, oltre ad essere più comodamente identificate da un nome, e non un numero, associano ad ogni regola aggiunta, un ID progressivo multiplo di 10.
    
    Questo rende possibile sia eliminare regole precedentemente aggiunte, con la sintassi `no IDRegola`, sia inserire una nuova regola tra due precedentemente inserite.
    
    Esempio: 
    
    ```
    Router(config)# ip access-list extended BLOCCO_SOCIAL
    Router(config-ext-nacl)# deny tcp 192.168.1.0 0.0.0.255 host 10.0.0.50 eq 80 //ID 10
    Router(config-ext-nacl)# permit ip any any //ID 20
    Router(config-ext-nacl)# 15 permit icmp any any //Ho inserito una regola tra la prima e la seconda.
    Router(config-ext-nacl)# exit
    
    ```
    

Creata l’ACL, è possibile applicarla ad un interfaccia di un router tramite i comandi:

```
Router(config)# interface <interfaccia> (es. GigabitEthernet0/0)
Router(config-if)# ip access-group <NUMERO o NOME ACL> <in | out>
```

- IN - Il Router controlla le regole sul traffico in arrivo sull’interfaccia
- OUT - Controllo delle regole sul traffico in uscita dall’interfaccia

Quando un pacchetto raggiunge il router su di un interfaccia configurata con ACL, il router controlla le regole in ordine di scrittura, per verificare se il pacchetto fa match con una delle regole (siano esse di permit o deny).

In caso un pacchetto non faccia match con nessuna delle regola, vale allora esso verrà implicitamente scartato.

Per semplificare i comandi di inserimenti delle regole, è possibile utilizzare degli alias al posto delle porte TCP:

- **`www`**  -  Porta **80** (HTTP)
- **`telnet`** - Porta **23**
- **`smtp`** - Porta **25** (Invio email)
- **`pop3`** - Porta **110** (Ricezione email)
- **`ftp`** - Porta **21** (File Transfer)
- **`domain`** - Porta **53** (DNS - attenzione, usa spesso UDP)

Esempio: `access-list 110 permit tcp any host 192.168.200.200 eq www` corrisponde a `access-list 110 permit tcp any host 192.168.200.200 eq 80`

## Configurazione ACL in Packet Tracer

### Esempio 1:

Vogliamo permettere l’accesso al Web Server nella VLAN Gialla, ma bloccare il ping (ICMP).

![image.png](images/image%2059.png)

```
Router(config)#ip access-list extended ALLOW_WEB_SERVER
Router(config-ext-nacl)#permit TCP any host  192.168.200.200 eq www
Router(config-ext-nacl)#deny ICMP any any
Router(config-ext-nacl)#exit
Router(config)#interface fastEthernet 0/0.200
Router(config-subif)#ip access-group ALLOW_WEB_SERVER out
```

Nota: avrei potuto omettere la regola `deny ICMP any any`, facendo affidamento alla regola di default `deny IP any any`.

Con il comando `show running-config`, notiamo come la ACL sia stata correttamente applicata all’interfaccia

```
Router#show running-config 
Building configuration...

Current configuration : 1072 bytes
...
interface FastEthernet0/0
 no ip address
 duplex auto
 speed auto
!
interface FastEthernet0/0.100
 encapsulation dot1Q 100
 ip address 192.168.100.254 255.255.255.0
!
interface FastEthernet0/0.200
 encapsulation dot1Q 200
 ip address 192.168.200.254 255.255.255.0
 ip access-group ALLOW_WEB_SERVER out
!
...
```

---

### Esempio 2:

Permettere a tutti gli host di accedere al Server Web pubblico, solo agli host interni alla VLAN di accedere anche a quello privato.

![image.png](images/image%2060.png)

```
Router(config)#ip access-list extended ALLOW_PUBLIC_SERVER
Router(config-ext-nacl)#permit TCP any host 192.168.0.1 eq www
Router(config-ext-nacl)#exit
Router(config)#interface fastEthernet 0/0
Router(config-if)#ip access-group ALLOW_PUBLIC_SERVER in
Router(config-if)#exit
```

Da questa configurazione, nessun host nella VLAN verde può comunicare col server nella rete grigia: le richieste arrivano correttamente al server, ma le risposte vengono bloccate dalla ACL sul router (è concesso solo il traffico TCP verso il server pubblico della VLAN verde, tutto il resto viene scartato)

Modifichiamo così la ACL:

```
Router(config)#ip access-list extended ALLOW_PUBLIC_SERVER
Router(config-ext-nacl)#permit TCP any any established 
Router(config-ext-nacl)#exit
```

Ora tutte le risposte (ma non le richieste) sono concesse.