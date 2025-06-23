>[!note]
>Esiste una ben nota e ormai libreria di blocchi funzionali predefiniti di tipo sequenziali, che contiene i blocchi per tutte le funzioni sequenziali base.
>
>I tipici blocchi funzionali sequenziali sono:
>- Registro parallelo
>- Registro a scorrimento
>- Banco di registri (register file)
>- Memoria

### Registro parallelo
>[!note]
>Un registro parallelo è un vettore di $n\geq1$ flip-flop di tipo $D$ e ha $n\geq1$ ingressi, $n\geq1$ uscite e un ingresso di clock $C$.
>
>Ad ogni ciclo di clock, il registro legge e memorizza nel suo stato la parola di $n\text{ bit}$ presente in ingresso, e la presenta sulle $n$ uscite nel ciclo successivo.
>
>![[Pasted image 20250622144140.png|center]]

Se si usassero dei bistabili D trasparenti, durante il livello alto del clock il registro sarebbe esso stesso del tutto trasparente, e dunque non si comporterebbe come registro.

>[!tip] Registro parallelo con comando di caricamento
>Un registro parallelo con comando di caricamento funziona esattamente come un registro parallelo, ma ha in aggiunta un ingresso di comando di caricamento $L$. Se $L$ è attivo la parola in ingresso al registro viene memorizzata nel registro stesso e presentata in uscita nel ciclo successivo. Altrimenti, il registro mantiene il suo stato corrente di memorizzazione.
>
>![[Pasted image 20250622144427.png|center]]

### Registro a scorrimento
>[!note]
>Un registro a scorrimento è una successione di $n\geq1$ flip-flop D collegati in cascata e ha un ingresso seriale $S$, $n\geq1$ uscite parallele e un ingresso di clock $C$.
>
>Ad ogni ciclo di clock, fa scorrere di un bit verso destra la parola memorizzata, perdendo il bit più a destra e aggiungendo a sinistra il bit presente sull'ingresso seriale.
>
>![[Pasted image 20250622144637.png|center]]

Esistono diverse varianti e integrazioni, tra cui:
- Registro a scorrimento a sinistra
- Registro a scorrimento universale
- Registro a scorrimento con funzione di caricamento parallelo
- Registro parallelo
- Registro IN seriale/OUT seriale
- Registro IN parallelo/OUT seriale
- Registro IN parallelo/seriale OUT parallelo/seriale

### Uscite condivise
>[!note]
>L'organizzazione interna della memoria e la struttura dei banche di memoria e dei banchi di registri prevedono generalmente che le uscite di due o più componenti siano collegate alle stesse linee di uscita.
>
>Sono necessari opportuni elementi funzionali che garantiscano la non interferenza tra i moduli che condividono le stesse linee di uscita.
>
>Il buffer tri-state è un circuito elementare modellabile come un contatto a tre posizioni:
>- In uno stato di bassa impedenza consente di avere in uscita o il livello alto $1$ o il livello basso $0$.
>- In uno stato di alta impedenza isola elettricamente l'uscita.
>
>L'uscita tri-state viene gestita da un apposito ingresso di controllo (Output Enable), che, se non attivo, forza lo stato di alta impedenza.
>
>![[Pasted image 20250622145450.png|center]]

### Banco di registri
>[!note]
>Definiamo come banco di registri i registri organizzati in una struttura a vettore.
>
>Ogni registro è identificato da un indirizzo specificato su $\lceil\log_{2}n\rceil\text{ bit}$, con $n\geq1$ registri. (In RISC-V sono $5\text{ bit}$).
>
>Le operazioni eseguibili sul banco sono lettura e scrittura, dove si presentano/memorizzano in uscita i $64\text{ bit}$ memorizzati nel registro indirizzato.

>[!tip] Banco di registri RISC-V
>Nel register file di RISC-V ci sono $2$ porte di lettura indipendenti, e $1$ porta di scrittura, quindi con le opportune temporizzazioni è possibile accedere in parallelo a $3$ registri distinti.
>
>Per quanto riguarda l'accesso in lettura, è sufficiente fornire l'indirizzo dei registri coinvolti e la lettura avviene ogni ciclo di clock, mentre per l'accesso di clock è necessario un segnale di scrittura esplicito $\text{Write}$.
>
>![[Pasted image 20250622150143.png|center]]
>
>Di seguito lo schema delle porte di lettura e di scrittura.
>
>![[Pasted image 20250622150317.png|center]]
>
>![[Pasted image 20250622150331.png|center]]

La scrittura avviene sul fronte di salita del ciclo e quindi la lettura fornisce il valore scritto a ciclo precedente.