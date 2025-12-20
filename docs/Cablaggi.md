# Cablaggi

## Cavi Ethernet

![images/image.png](images/image%202.png)

Un cavo ethernet, è composto da 4 coppie di cavi intrecciati (Twisted Pair), connesse alle estremità da connettori RJ45.

![images/image.png](images/image%203.png)

I cavi sono disposti in coppie ritorte, al fine di ridurre le interferenze con le coppie vicine, annullandosi a vicenda gli effetti dei campi magnetici (diafonia).

Il segnale viene inviato su un solo cavo della coppia: sull’altro, viaggia un segnale costante a 0 (terra).

Questo meccanismo, permette al destinatario di ricavare un segnale privo di rumore, operando per differenza tra i due segnali sui due cavi.

![images/image.png](images/image%204.png)

Supponiamo un trasmettitore voglia inviare un’informazione s(t).

Durante la trasmissione lungo il canale, vengono introdotte delle imperfezioni all’interno del dato, che apparirà modificato (s’(t))

Come fa il ricevitore a “ripulire” il segnale, e ricavare il dato originale?

![Segnale originale, immesso dal trasmettitore sul canale.](images/image%205.png)

Segnale originale, immesso dal trasmettitore sul canale.

![Imperfezioni introdotte dal canale. Notiamo come il rumore altera il segnale dei due cavi allo stesso modo!](images/image%206.png)

Imperfezioni introdotte dal canale. Notiamo come il rumore altera il segnale dei due cavi allo stesso modo!

![images/image.png](images/image%207.png)

![images/image.png](images/image%208.png)

Il ricevitore, applicando una differenza tra i due segnali, ricostruisce correttamente s(t).

### Tipologie di cavo

![images/image.png](images/image%209.png)

- UTP - Unshielded Twisted Pairs, non forniscono protezione da EMI
- FTP - Foiled Twisted Pairs, le 4 coppie ritorte, sono racchiuse da un foglio di alluminio.
- STP - Shielded Twisted Pairs, le singole coppie ritorte sono racchiuse da un foglio di alluminio o maglia in rame.
- SFTP - Shielded Foiled Twisted Pair, le singole coppie ritorte sono schermate, e infine un unico foglio di alluminio le racchiude assieme. Massimo livello di protezione EMI.

### Standard di terminazione/crimpatura connettori

Esistono due standard che definiscono l’ordine dei fili all’interno del connettore RJ45.

![images/image.png](images/image%2010.png)

- T568A - Assegna le coppie verde e arancione in modo compatibile con vecchi impianti telefonici USOC, affinché fosse più semplice ricordare l’ordine di terminazione.
- T568B - Standard commerciale oggi più utilizzato.

<aside>
<img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />

Non cambia assolutamente nulla utilizzare uno piuttosto che l’altro standard. L’importante ovviamente è usare lo stesso standard alle due estremità del cavo, per il resto l’ordine dei colori cambia nulla.

</aside>

### Cavi Ethernet Straight e Crossed

<aside>
<img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />

Faremo riferimento allo standard di terminazione T568B, in quanto T568A è ora in disuso.

</aside>

Gli standard **BASE-T** (10BASE-T, 100BASE-TX, 1000BASE-T) definiscono **come Ethernet trasmette i bit sul doppino**

10BASE-T e 100BASE-T, prevedono la comunicazione dei dati (in modalità unidirezionale) su sole due delle 4 coppie presenti nel cavo.

Coppia 1 (Composta dai pin 1 e 2),  Coppia 2 (Composta dai pin 3 e 6)

Esistono due standard che descrivono come le porte ethernet dei dispositivi sono cablate:

MDI (Medium Dependent Interface) - Usato dai dispositivi terminali (PC, Server, Stampanti…), usano la coppia 1 per trasmissione, coppia 2 per ricezione.

MDIX (MDI eXchanged) - Usato dai dispositivi infrastrutturali (Switch, Hub..), usano coppia 1 per ricezione, coppia 2 per trasmissione.

I ruoli sono speculari, mettendo “una davanti all’altra” due porte MDI e MDIX, i pin di trasmissione di una finiscono su quelli di ricezione dell’altra, e idem per l’altra coppia.

![images/image.png](images/image%2011.png)

Ma se volessi far comunicare due dispositivi le cui porte ethernet sono cablate attraverso lo stesso schema? (Entrambe MDI o MDIX)

A quel punto, dovrei “manualmente” effettuare l’inversione delle coppie di pin TX ed RX. Ecco da cosa deriva l’uso di cavi straight e crossed.

![images/image.png](images/image%2012.png)

Se i due dispositivi che voglio mettere in comunicazione, possiedono porte ethernet internamente collegate secondo standard opposti (MDI e MDIX), uso un cavo straight (l’inversione delle coppie RX e TX è già effettuata), altrimenti uso un cavo crossed (l’inversione RX e TX avviene modificando lo schema di terminazione dei pin all’interno del cavo)

Con lo standard 1000BASE-T, la comunicazione avviene utilizzando simultaneamente tutte e 4 le coppie ritorte bi-direzionalmente (sia in trasmissione che ricezione). 

Non esiste più la distinzione tra coppie di trasmissione e coppie di ricezione.

Tale standard, prevede l’utilizzo della tecnologia Auto MDI/MDI-X, che permette alle PHY di identificare automaticamente com’è cablato il dispositivo che si trova all’altra estremità del cavo, invertendo eventualmente la mappatura dei pin tramite switch analogici.

Esempio: Collego un PC con NIC Auto-MDI/MDI-X, ad un dispositivo MDI. La NIC del PC si autoconfigura come MDI-X. Il link funziona con qualsiasi cavo (straight o cross)

Collego un PC con NIC Auto-MDI/MDI-X ad un dispositivo MDI-X. Il PC si configura come MDI, anche qui il link funziona indipendentemente dal cavo

**Collegando un PC con Auto-MDI/MDI-X con un dispositivo senza Auto-MDI/MDI-X, il link funziona nel 100% dei casi**, indipendentemente dal cavo.

Tabella riassuntiva sull’uso corretto dei cavi cross e straight:

![images/image.png](images/image%2013.png)

<aside>
<img src="https://www.notion.so/icons/bookmark_lightgray.svg" alt="https://www.notion.so/icons/bookmark_lightgray.svg" width="20px" />

“Le NIC oggi sono ancora “cablate” come MDI e MDI-X?”

**Tecnicamente sì**, ma con un’importante precisazione.

A livello FISICO (cablatura dei pin):

- Le NIC dei PC seguono ancora la mappatura storica **MDI**:
    - coppia 1 (pin 1–2) → TX locale
    - coppia 2 (pin 3–6) → RX locale
- Quelle dei dispositivi intermedi, sono ancora collegati MDI-X

Questo *schema di cablatura interna* rimane per compatibilità storica, perché lo standard RJ-45 e il pinout fisico non sono stati cambiati. Tuttavia **la cablatura fisica non determina più il comportamento del link,** perché entra in gioco la logica di **Auto-MDI/MDI-X nella PHY**.

La PHY moderna:

- guarda come sono effettivamente cablate le coppie,
- analizza i FLP (Fast Link Pulses) di autonegotiation,
- capisce se il dispositivo remoto è MDI o MDI-X,
- e se serve, **inverte internamente TX/RX tramite switch analogici + DSP**.
</aside>

## Fibra ottica

![images/image.png](images/image%2014.png)

La trasmissione di segnali mediante tecnologia fibra ottica, consiste nell’invio di impulsi luminosi, che possono essere elaborati dal computer.

La luce viaggia seguendo una traiettoria rettilinea, finché non colpisce un oggetto che la riflette, rifrange, o assorbe.

**Riflessione**

- **Definizione:** La luce incontra una superficie (come uno specchio) e viene deviata indietro nello stesso mezzo.

**Riflessione**

- **Definizione:** La luce incontra una superficie (come uno specchio) e viene deviata indietro nello stesso mezzo.

La fibra ottica è studiata per minimizzare la riflessione, e massimizzare la rifrazione (eliminando l’assorbimento).

Un cavo in fibra ottica, è composto da:

![images/image.png](images/image%2015.png)

- Core - In cui gli impulsi luminosi viaggiano, composto da vetro al silicio estremamente puro (privo di contaminazioni). Questa proprietà è necessaria affinché la luce possa viaggiare senza degradarne la qualità.

![images/image.png](images/image%2016.png)

- Cladding - Un ulteriore strato di vetro, meno puro rispetto al core, che lo racchiude per l’intera lunghezza del cavo. Agisce come riflettore, mantenendo la luce all’interno del core, impedendo che esca al di fuori.
- Coating - Strato in gomma, che protegge il vetro della fibra da graffi e agenti atmosferici.
