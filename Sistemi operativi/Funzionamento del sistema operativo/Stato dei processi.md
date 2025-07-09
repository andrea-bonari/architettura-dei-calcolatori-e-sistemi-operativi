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

>[!tip] Gestione del context switch

>[!tip] Gestione degli interrupt

### Gestione dello stato di attesa

### Segnali

