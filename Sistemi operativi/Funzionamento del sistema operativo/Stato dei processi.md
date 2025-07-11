>[!note]
>Normalmente un processo è in esecuzione in modo U. Se un processo corrente richiede un servizio di sistema viene attivata una funzione del SO che esegue il servizio per conto di tale processo. Faremo riferimento a questo fatto dicendo che un servizio è svolto nel contesto di un certo processo. Si usa quindi dire che un processo è in esecuzione in modo S quando il SO è in esecuzione nel contesto di tale processo, sia per eseguire un servizio, sia per servire un interrupt.
>
>Un processo può trovarsi in uno dei due stati fondamentali seguenti:
>- Attesa: un processo in questo stato non può essere messo in esecuzione, perché deve attendere il verificarsi di un certo evento.
>- Pronto: un processo pronto è un processo che può essere messo in esecuzione se lo scheduler lo selezione
>- Corrente: Uno dei processi in stato di pronto che è effettivamente in esecuzione.
>
>Lo stato di un processo è registrato nel suo descrittore.

![[Pasted image 20250710003910.png]]

>[!tip] Scheduler
>Lo scheduler è il componente del SO che decide quale processo mettere in esecuzione. Svolge due tipi di funzioni:
>- Determina quale processo deve essere messo in esecuzione, quando e per quanto tempo, cioè realizza la politica di scheduling del SO.
>- Esegue l'effettiva commutazione di contesto, cioè la sostituzione del processo corrente con un altro processo in stato di pronto. (Questa operazione è svolta dalla funzione `schedule`).

### Runqueue a waitqueue
>[!note]
>Lo scheduler gestisce la lista dei processi pronti (runqueue) e le liste dei processi in attesa (waitqueues) di un particolare evento.
>
>La runqueue contiene due campi:
>- RB: lista di puntatori ai descrittori dei processi pronti.
>- CURR: puntatore al descrittore del processo corrente.
>Ed è implementata da una lista doppia circolare nella quale gli elementi sono inseriti/rimossi dinamicamente.
>
>La waitqueue è una lista contenente i puntatori ai descrittori dei processi in attesa di un certo evento. Esiste una waitqueue separata per ogni evento. I processi in una waitqueue saranno risvegliati da una wake up quando arriverà l'evento specifico e sarà posta nella runqueue.

Linux assegna ad ogni processo una sPila di $8\text{ kB}$. Durante l'esecuzione di un servizio di sistema per conto di un processo per la sua sPila contiene una parte del contesto hardware del processo. Il meccanismo hardware permette la commutazione corretta da uPila a Spila e viceversa, a condizione che `ssp` e `usp` contengano i valori corretti da assegnare al registro `sp`.

Dato che il SO mantiene una diversa sPila per ogni processo, la gestione di questo meccanismo diventa più complesso: è necessario salvare i valori di `ssp` e `usp` durante la sospensione tra una esecuzione di un processo e la successiva. Proprio per questo motivo il descrittore di un processo P contiene i campi `sp0` e `sp`.

>[!tip] Gestione del context switch
>Quando il processo è in esecuzione in modo U, la sPila è vuota, e quindi in `ssp` viene messo il valore di base preso da `sp0` del descrittore di P. Quando la CPU passa al modo S, in `usp` viene scritto automaticamente dall'hardware il valore di `sp` corrente, per il ritorno al modo U.
>
>Se, durante l'esecuzione in modo S, viene eseguita una commutazione di contesto, Linux opera nel modo seguente: salva `usp` sulla sPila di P e poi salva il valore del registro `sp` nel campo `sp` del descrittore di P.
>
>Quando poi P riprenderà l'esecuzione `sp` verrà ricaricato dal campo `sp` del descrittore, puntando alla cima della sPila, `usp` verrà ricaricato prendendolo dalla sPila, e `ssp` verrà ricaricato prendendolo dal campo `sp0` del descrittore.
>
>Se, durante l'esecuzione di un processo P in modo S viene eseguita una commutazione di contesto, Linux esegue il salvataggio di contesto:
>- Salva il valore del `pc` di System Call in esecuzione sulla sPila di P.
>- Salva `usp` sulla sPila di P.
>- Salva il valore del registro `sp` nel campo `sp` del descrittore di P.
>
>Quando poi P riprenderà l'esecuzione, Linux esegue il ripristino di contesto:
>- `sp` è caricato dal campo `sp` del descrittore di P, puntando alla cima della sPila di P.
>- `ssp` è caricato dal campo `sp0` del descrittore di P, puntando alla base della sPila di P.
>- `usp` è caricato dalla sPila di P.
>- `pc` è caricato dal valore del `pc` presente in sPila di P.

>[!tip] Gestione degli interrupt
>Quando si verifica un interrupt esiste sempre un processo in stato di esecuzione, e possono verificarsi i seguenti casi:
>- L'interrupt interrompe il processo mentre funziona in modo U.
>- L'interrupt interrompe un servizio di sistema che è stato invocato dal processo in esecuzione.
>- L'interrupt interrompe una routine di interrupt relativa ad un interrupt con priorità inferiore.
>
>La routine di interrupt svolge la propria funzione senza disturbare il processo in esecuzione, e vengono eseguiti nel contesto del processo in esecuzione.
>
>Se la routine di interrupt è associata al verificarsi di un evento E sul quale è in stato di attesa un certo processo P la routine di interrupt risveglia il processo P, passandolo dallo stato di attesa allo stato di pronto.

### Gestione dello stato di attesa
>[!note]
>I tipi di eventi che possono essere oggetto di attesa appartengono a diverse categorie che richiedono una gestione differenziata, in particolare:
>- Attesa del completamento di un operazione I/O.
>- Attesa dello sblocco di un lock.
>- Attesa dello scadere di un timeout.
>  
>In alcuni casi conviene risvegliare tutti i processi presenti nella coda associata ad un evento. In altri casi conviene risvegliarne uno solo, poiché uno solo potrà utilizzare la risorsa e gli altri rimanere in attesa. 
>
>I processi per i quali deve esserne risvegliato uno solo sono messi in stato di attesa esclusiva. Per mettere un processo in attesa non esclusiva si usa la funzione `wait_event_interruptible`, mentre per l'attesa esclusiva si usa `wait_event_interruptible_exclusive`.
>
>La routine di risveglio risveglia tutti i processi dall'inizio della lista fino al primo processo in attesa esclusiva.

### Segnali
>[!note]
>Un segnale è un avviso asincrono inviato ad un processo dal SO oppure da un altro processo. Ogni segnale è identificato da un numero (compreso tra 1 e 31) ed un nome simbolico.
>
>Un segnale causa l'esecuzione di un'azione da parte del processo che lo riceve. Questa azione può essere svolta solamente quando il processo che riceve il segnale è in esecuzione in modo U.
>
>Se il processo è in esecuzione in modo S, il segnale verrà processato immediatamente al ritorno al modo U.
>
>Se il processo ha definito una propria funzione destinata a gestire quel segnale, questa viene eseguita, altrimenti viene eseguito il default signal handler.
>
>La maggior parte dei segnali può essere bloccata da un processo, un segnale bloccato rimarrà pendente finché non sarà sbloccato.
>
>Esistono due signal che non possono essere bloccati dal processo: `SIGKILL` (terminazione immediata del processo) e `SIGSTOP` (blocco del processo per riprenderlo più tardi).
>
>Oltre a questi alcuni signal speciali sono `SIGINT` (simile a `SIGKILL`, ma il processo può definire il suo handler, inviato da `ctrl+c`) e `SIGTSTP` (simile a `SIGSTOP`, ma il processo può definire il suo handler, inviato da `ctrl+z`).

>[!tip] Attesa interrompibile
>Se un signal viene inviato ad un processo che non è in esecuzione in modo U:
>- Se il processo è in esecuzione in modo S, il signal verrà processato immediatamente al ritorno al modo U.
>- Se il processo è in stato di pronto, il signal viene tenuto in sospeso finché il processo tornerà in esecuzione.
>- Se il processo è in stato di attesta in waitqueue, ci sono due casi:
>	- Se l'attesa è interrompibile il processo viene immediatamente risvegliato da signal senza che l'evento su cui era in attesa si sia verificato.
>	- Altrimenti il signal rimane pendente perché il processo in attesa non può essere risvegliato da un signal.
>

>[!tip] Preemption
>Dato che il nucleo è non-preemptive non viene eseguita immediatamente una commutazione di contesto quando il sistema scopre che un task in esecuzione dovrebbe essere sospeso e portato in stato di pronto, ma viene semplicemente settato il flag `TIF_NEED_RESCHED`, che al momento opportuno causerà la commutazione di contesto.

L'uso di una waitqueue è conveniente nelle situazioni in cui la funzione che scoprirà il verificarsi dell'evento atteso conosce la coda relativa all'evento. Esistono però situazioni, nelle quali l'evento atteso è scoperto da una funzione che non ha modo di conoscere la waitqueue. In questi casi viene invocata una variante di wakeup che riceve come argomento direttamente un puntatore al descrittore del processo da risvegliare `wakeup_process`.

>[!tip] Timeout e timer
>Un timeout definisce una scadenza temporale. Esistono servizi per definire i timeout in vario modo. Il principio di base è sempre quello di specificare un intervallo di tempo a partire da un momento prestabilito.
>
>Il tempo all'interno del sistema è rappresentato dalla variabile `jiffies` che registra il numero di tick del clock di sistema intercorsi dall'avviamento del sistema. La durata effettiva dei `jiffies` dipende quindi dal clock del sistema.
>
>Quindi i servizi di sistema permettono di specificare l'intervallo di tempo secondo diverse rappresentazioni esterne, che dipendono dalla scala temporale e dal livello di precisione desiderato.
>
>L'interrupt del clock aggiorna i `jiffies`.
>
>Siccome il controllo della scadenza dei timeout non può essere svolto ad ogni tick per ragioni di efficienza, si utilizzano delle soluzioni complesse. Noi ipotizzeremo semplicemente che esista una routine `controlla_timer` che controlla la lista dei timeout per verificare se qualche timeout è scaduto.

### Non-preemption del nucleo
>[!note]
>In assenza della regola di non-preemption del nucleo ci possono essere dei problemi di concorrenza, per l'esistenza di molte CPU che eseguono in parallelo, oppure per la sospensione di un'attività a causa di una commutazione di contesto, con partenza di una nuova attività.

