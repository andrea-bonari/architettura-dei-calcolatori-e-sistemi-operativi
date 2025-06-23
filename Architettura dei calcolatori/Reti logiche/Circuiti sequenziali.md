>[!note]
>Un circuito digitale è di tipo sequenziale se le sue uscite dipendono non solo dai valori correnti degli ingressi, ma anche da quelli passati. Un circuito digitale sequenziale è pertanto dotato, in ogni istante di tempo, di uno stato che, insieme ai valori degli ingressi, ne determina il comportamento futuro.
>
>L'elemento funzionale elementare per la realizzazione di circuiti sequenziali è il bistabile, che è in grado di memorizzare un bit di informazione.

Il circuito sequenziale ha, in ogni istante, uno stato: il valore dei bit memorizzati nei bistabili facenti parte del circuito.

I bistabili mantengono lo stato memorizzato finché uno o più segnali di ingresso forzano il cambiamento di stato. Vengono classificati in base al numero di ingressi previsti per comandare il bistabile, o in base al modo in cui tali ingressi determinano il cambiamento di stato.

Tutti i bistabili, presentano un ritardo di commutazione dell'uscita, che dipende dalla tecnologia usata.

### Bistabili asincroni
>[!note]
>I bistabili asincroni sono privi di un segnale di sincronizzazione e modificano lo stato rispondendo direttamente a eventi sui segnali di ingresso.
>
>Un bistabile SR è dotato di $2$ ingressi $S$ (Set) e $R$ (Reset) e di $2$ uscite $Q$ e $!Q$, dove l'uscita $Q$ rappresenta lo stato memorizzato. Di seguito mostriamo lo schema circuitale e il blocco funzionale.
>
>![[Pasted image 20250622140942.png|center]]
>
>Se $S=0$ e $R=1$, qualunque sia il valore dello stato precedente, $Q$ viene portata a $0$ e $!Q$ a $1$.
>
>Se $S=1$ e $R=0$, qualunque sia il valore dello stato precedente $Q$ viene portata a $1$ e $!Q$ a $0$.
>
>Se $S=R=0$, l'uscita $Q$ mantiene lo stesso stato. Nel caso di configurazione di ingresso $S=R=1$ il comportamento del bistabile è indefinito.

>[!tip] Diagramma temporale
>Il diagramma temporale è un sistema di assi cartesiani con ascissa in tempo (discreta) e in ordinata i vari segnali i cui valori logici si succedono al trascorrere del tempo.

Il diagramma temporale di questo bistabile è il seguente:

![[Pasted image 20250622141025.png|center]]

Si supponga di avere una periferica che deve mandare un segnale di richiesta a un processore. La periferica genera solo un breve impulso di richiesta e il processore potrebbe essere occupato e non in grado di rispondere subito alla richiesta, onorandola. È dunque necessario interporre tra periferica e processore un circuito digitale adattatore che riceva l'impulso, lo memorizzi e lo stabilizzi per poi mandarlo al processore.

Questo adattatore mantiene la richiesta pendente fino a quando il processore non sia disponibile a onorarla. La cancellerà quando il processore segnalerà di averla acquisita ed essere pronto ad onorarla.

![[Pasted image 20250622141440.png|center]]

### Segnale di sincronizzazione
>[!note]
>Il segnale di clock è un segnale binario con andamento periodico nel tempo, ed è una successione di impulsi con ogni impulso a larghezza costante, e con due impulsi consecutivi a distanza costante. Serve a scandire gli istanti di tempo in cui le transizioni possono avvenire.
>
>Il ciclo di clock mantiene $3$ intervalli, l'intervallo iniziale, il fronte di salita e il fronte di discesa.

### Bistabili sincroni
>[!note]
>A differenza dei bistabili asincroni, nei bistabili sincroni ci sono due aspetti differenti: la relazione ingresso-stato e la relazione stato-uscita.
>
>La relazione ingresso-stato definisce quando gli ingressi modificano lo stato interno del bistabili (temporizzazione basata sul livello o temporizzazione basata sul fronte del segnale di controllo)
>
>La relazione stato-uscita definisce quando lo stato aggiorna le uscite (commutazione basata sul livello o commutazione basata sul fronte del segnale di controllo).

In caso di commutazione basata sul livello del segnale di controllo si parla di Latch, mentre in caso di commutazione basata sul fronte del segnale di controllo si parla di flip-flop.

### Bistabile SR sincronizzato (SR-Latch)
>[!note]
>Un bistabile SR sincronizzato ha $2$ ingressi $S$ e $R$, un ingresso di sincronizzazione $C$, e $2$ uscite $Q$ e $!Q$.
>
>Nel bistabile SR sincronizzato se il clock vale $0$, gli ingressi $S$ e $R$ non hanno alcun effetto, mentre se il clock vale $1$ gli ingressi $S$ e $R$ sono efficaci.
>
>![[Pasted image 20250622142534.png|center]]

### Bistabile D sincronizzato (D-Latch)
>[!note]
>Un bistabile D ha un ingresso $D$, un ingresso di sincronizzazione $C$ e $2$ uscite $Q$ e $!Q$.
>
>Se il clock vale $0$, l'ingresso $D$ non ha alcun effetto e il bistabile mantiene memorizzato il suo stato corrente. Se il clock vale $1$, l'ingresso $D$ è efficace e il bistabile memorizza il valore logico presente sull'ingresso $D$.
>
>![[Pasted image 20250622142730.png|center]]

Il diagramma temporale è il seguente, notiamo che le variazioni di uscita sono sincronizzate con il clock.

![[Pasted image 20250622142819.png|center]]

>[!tip] Comando di ripristino e precarica
>Tutti i bistabili dispongono di varianti dotate di un comando di ripristino $\text{CLR}$ che forza lo stato del bistabile a $0$, mentre alcuni bistabili dispongono del comando di precarica $\text{PR}$ che forza lo stato del bistabile a $1$.

>[!tip] Fenomeno di trasparenza
>I latch sincroni (SR o D) presentano, durante l'intervallo di tempo in cui il clock è attivo, il fenomeno di trasparenza delle uscite. In questo intervallo, se gli ingressi si modificano le uscite seguono questa modifica. 
>
>Per evitare il fenomeno di trasparenza si utilizzano i flip-flop (D o SR) che sono costituiti da due latch in cascata in modo che lo stato possa modificare le uscite solo in corrispondenza di un evento di fronte del segnale di controllo.

### Flip-flop D master-slave
>[!note]
>In un flip-flop master slave la relazione ingresso-stato è a livello, mentre la relazione stato uscita è sul fronte.
>
>![[Pasted image 20250622143456.png|center]]
>
>Il bistabile principale campiona l'ingresso $D=D_{1}$ durante l'intervallo alto del clock, lo emette sull'uscita $Q_{1}$ e lo manda all'ingresso $D_{2}$ del bistabile ausiliario. Il bistabile ausiliario campiona l'ingresso $D_{2}$ durante l'intervallo basso del clock e lo emette sull'uscita $Q_{2}=Q$.
>
>L'uscita generale $Q$ può variare solo nell'istante del fronte di discesa del clock.

I flip-flop master-slave e edge-triggered sono esternamente identici. Entrambi leggono i segnali in ingresso sul fronte di discesa e lo rendono disponibile in uscita.