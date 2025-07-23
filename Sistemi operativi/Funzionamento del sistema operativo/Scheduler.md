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

>[!tip] Meccanismi di base CFS
>Ad ogni task si assegna un peso iniziale, che quantifica l'importanza del task. La costante di sistema `L0` definisce il peso iniziale. Per ipotesi assumiamo che il peso per tutti i task t presenti nella runqueue sia `L0`, e che nessun task si autosospenda.
>
>Si ha che il numero di task nella runqueue a un certo istante è $\text{NRT}$ ed è almeno una. Si stabilisce un periodo di scheduling $\text{PER}$. Quindi ad ogni task si assegna un quanto di tempo $Q= \frac{\text{PER}}{\text{NRT}}$.
>
>I task vengono mantenuti nella coda ordinata RB, e il funzionamento dello scheduler CFS è schematizzabile nel modo seguente:
>1. Il task in testa a RB viene estratto e diventa corrente (CURR)
>2. Il task CURR viene eseguito fino a quando scade il quanto $Q$.
>3. Il task CURR viene sospeso e reinserito in fondo a RB.
>4. Si torna al passo 1.
>
>Il periodo $\text{PER}$ può essere visto come una finestra scorrevole nel tempo: non c'è suddivisione rigida nel tempo in intervalli disgiunti consecutivi, si può considerare ogni istante come l'inizio di un nuovo periodo di schedulazione.
>
>Il periodo di scheduling $\text{PER}$ varia dinamicamente al crescere o diminuire del numero di task $\text{NRT}$ presenti nella runqueue. È determinato da due parametri $\text{LT}$ (latenza) e $\text{GR}$ (granularità) secondo la formula: $$\text{PER}=\max(\text{LT},\text{NRT}\cdot\text{GR})$$

Il meccanismo base si basa su un quanto $Q$ costante e sulla politica round robin, tuttavia non garantisce fairness completa e quindi ci sono degli aspetti un po più complessi.

L'aspetto più importante di CFS è quello relativo alla misura virtuale del tempo, che ricalcola certe variabili per ogni task ad ogni impulso del real-time clock, ad ogni risveglio del task e ad ogni creazione di un task. La decisione su quale task mettere in esecuzione viene poi presa dalla funzione `schedule` quando viene chiamata.

Si valuta il peso relativo di ogni task $t$ rispetto a tutti i task: $$\text{RQL}=\sum\limits_{\forall t}t.\text{LOAD}$$
E si definisce quindi il `load_coff` come: $$t.\text{LC}= \frac{t.\text{LOAD}}{\text{RQL}}$$

Possiamo quindi stabilire la durata del quanto di tempo come: $$t.\text{Q}=\text{PER}\cdot t.\text{LC}$$
Si ha che per ogni task t $\sum\limits_{\forall t}t.\text{Q}=\text{PER}$.
### Virtual RunTime
>[!note]
>Lo scheduler CFS utilizza Virtual RunTime (VRT) per ordinare i task nella runqueue. VRT è una misura virtuale del tempo di esecuzione consumato da un processo, basato sulla modifica del tempo reale tramite coefficienti.
>
>La decisione su quale sia il prossimo task da mettere in esecuzione si basa sulla scelta dell task con $\text{VRT}$ minimo tra quelli nella runqueue. Il $\text{VRT}$ del task corrente viene ricalcolato ad ogni tick del real-time clock del sistema. Quando il task corrente termina l'esecuzione viene inserito in RB nella posizione determinata in base al valore di $\text{VRT}$ assunto durante l'esecuzione.
>

```
SUM = SUM + DELTA
t.VRT = t.VRT + DELTA * t.VRTC
```

Dove `SUM` è il tempo totale di esecuzione del task, `DELTA` è la durata dell'ultimo quanto consumato dal task e $t.\text{VRTC}= \frac{1}{t.\text{LOAD}}$ è il coefficiente di correzione di $\text{VRT}$.

Si definisce inoltre una variabile $\text{VMIN}$ ricalcolata assieme al task corrente che rappresenta il $\text{VRT}$ minimo tra i $\text{VRT}$ di tutti i task presenti nella runqueue. Questa variabile deve crescere in modo monotono. Per ora diciamo che: $$\text{VMIN}=\min(\text{CURR.VRT},\text{ LFT.VRT})$$
Dove $\text{LFT}$ è il primo task di RB (ha $\text{VRT}$ minimo).

Quando la funzione `wake_up` risveglia un task `tw`, deve ricalcolare la sua $\text{VRT}$. Siccome il risveglio di un processo lo fa reinserire nella runqueue e quindi modifica $\text{NRT}$ e $\text{RQL}$ è necessario ricalcolare i parametri $\text{LC}$ e $\text{Q}$. Ci sono quindi due formule utilizzate $$\begin{align*}
tw.\text{VRT}=\max\left(tw.\text{VRT},\text{VMIN}- \frac{\text{LT}}{2}\right)\qquad &\text{solo in }\mathtt{wake\_up}\\
\text{VMIN}=\max (\text{VMIN},\min(\text{CURR.VRT},\space\text{LFT.VRT}))\qquad&\text{solo in }\mathtt{tick}
\end{align*}$$

Risvegliando un task `tw` si richiede scheduling se almeno una delle due condizioni è verificata:
- Il task è in una classe di scheduling con diritto di esecuzione maggiore (RR).
- Il $\text{VRT}$ del task risvegliato è significativamente inferiore al $\text{VRT}$ corrente. In questo caso c'è un coefficiente correttivo $\text{WGR}$ (Wakeup granularity) affinché un task con attese brevissime non possa causare context switch troppo frequenti.

>[!tip] Assegnamento del peso ad un task
>L'assegnamento di un paso ad un task si basa sul parametro `nice_value` del task. L'utente può assegnare un `nice_value` diverso a ciascun task. Questo parametro va da $-20$ a $+19$. Un task con `nice_value` maggiore ottiene in proporzione meno tempo di CPU di uno con `nice_value`.
>
>Il `nice_value` di un task $t$ viene convertito nel suo task con la formula: $$t.\text{LOAD}= \frac{1024}{1.25^{\mathtt{nice\_{value}}}}$$

>[!tip] Riassunto
>![[Pasted image 20250720122424.png]]

