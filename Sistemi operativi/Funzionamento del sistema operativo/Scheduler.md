>[!note]
>Il SO è caratterizzato dalle politiche adottate per decidere quali task eseguire e per quanto tempo devono essere eseguite (politiche di scheduling). Il componente del SO che realizza le politiche di scheduling è detto scheduler.
>
>Lo scheduler dovrebbe garantire che i task più importanti vengano eseguiti prima di quelli meno importante, che i task di pari importanza vengano eseguiti in maniera equa, e in particolare, che nessun task debba attendere un turno di esecuzione per molto più tempo degli altri task.
ou
Lo scheduler è un componente critico nel funzionamento di un sistema operativo ed è oggetto di molta ricerca per migliorarne le caratteristiche e le prestazioni.

>[!tip] Funzioni dello scheduler
>Per un sistema multi-programmato, la gestione dell'esecuzione dei task più immediata, semplice e ragionevolmente equa è la politica del round robin. Dati $n\geq 1$ task di pari importanza, la politica di scheduling round robin assegna un uguale quanto di tempo a ciascun task circolarmente. La politica round robin è equa e garantisce che un task non resti fermo indefinitivamente, cioè che non vada incontro a starvation.
>
>Lo scheduler interviene in certi momenti per determinare quale task rimettere in esecuzione, e contestualmente toglie un task dall'esecuzione. La scelta del task da rimettere in esecuzione avviene tra tutti i task in stato di pronto esistenti nel sistema, cioè tra tutti quelli nella runqueue. Il task scelto è quello che in quel momento ha il diritto di esecuzione maggiore.
>
>Ci sono tre casi in cui lo scheduler deve scegliere un task corrente:
>- Quando un task si autosospende e lascia l'esecuzione.
>- Quando un task in stato di attesa viene risvegliato da parte di un altro task e passa in stato di pronto con maggiore diritto di esecuzione.
>- Quando il task correntemente in esecuzione è gestito con politica di scheduling di tipo round robin e il quanto di tempo assegnato a tale task scade.

I task possono avere requisisti di scheduling molto diversificati per applicazioni. Suddividiamo quindi i task in tre categorie:
- Task real-time: devono soddisfare vincoli di tempo molto stringenti e dunque vanno schedulati con grande rapidità.
- Task semi-real-time: possono reagire con una discreta rapidità, ma non garantiscono di non superare un ritardo massimo fissato.
- Task normali: tutti gli altri task, che a loro volta in I/O bound (si autosospendono frequentemente per necessità di I/O) e CPU bound (si autosospendono raramente).

Per gestire ciascuna categoria di task secondo le rispettive caratteristiche distintive, lo scheduler realizza varie politiche di scheduling. Ogni politica è realizzata da una classe di scheduling diversa (contenuta nel descrittore del task).

Lo scheduler è quindi l'unico gestore dei task in stato di pronto, cioè della runqueue. Per questo motivo tutte le altre funzioni del SO e in particolare quelle del nucleo devono chiedere allo scheduler di eseguire operazioni sulla runqueue.

>[!tip] Politiche di scheduling fondamentali
>Attualmente le tre classi di scheduling più importanti del SO Linux sono SCHED_FIFO, SCHED_RR, SCHED_NORMALE.
>
>Dove i task ti classe FIFO hanno sempre precedenza su tutti quelli delle altre due classi, i task di classe RR hanno sempre precedenza su tutti quelli della classe NORMAL, e i task di classe NORMAL danno sempre precedenza a tutti quelli delle altre due classi.

### Scheduling dei task soft real-time
>[!note]
>Le classi SCHED_FIFO e SCHED_RR sono usate per i task di tipo soft real-time. Il SO linux non supporta i processi RT in senso stretto.
>
>A ciascun task di queste due classi viene attribuita alla creazione una priorità detta statica, perché è assegnata all'inizio e poi non varia più.
>
>Solitamente un task figlio eredità la priorità statica del task padre, e si può cambiare la priorità statica di un task utilizzando comandi di amministrazione.
>
>La priorità statica del task è memorizzata in `task_struct` nel campo `static_prio`.

### Scheduling dei task di classe NORMAL (CFS)
>[!note]
>Lo scheduler Linux candida all'esecuzione i task di classe NORMAL seol se in stato di pronto non ci sono task delle classi FIFO e RR, che li precedono sempre. Lo scheduler di Linux per i task di classe di scheduling SCHED_NORMAL è chiamato Completely Fair Scheduler (CFS).
>
>CFS ambisce a raggiungere questo obiettivo: dati $n\geq1$ task assegnati a una CPU di potenza $1$, dedicare a ciascun task una CPU virtuale di potenza $\frac{1}{n}$.
>
>Se il sistema è multiprocessore ciascuna CPU ha una sua runqueue da gestire modo CFS.

Per raggiungere il suo obiettivo lo scheduler CFS deve:
1. Determinare ragionevolmente la durata del quanto di tempo
2. Assegnare un peso a ciascun task
3. Permettere a un task rimasto a lungo in stato di attesa di tornare rapidamente in esecuzione quando viene risvegliato, ma senza favorirlo troppo.

Ad ogni task si assegna un peso iniziale, che quantifica l'importanza del task. La costante di sistema `L0` definisce il peso iniziale. Per ipotesi assumiamo che il peso per tutti i task t presenti nella runqueue sia `L0`, e che nessun task si autosospenda.
