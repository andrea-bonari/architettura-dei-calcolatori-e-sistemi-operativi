>[!note]
>Il sistema operativo (SO) è un insieme di programmi (moduli software) che svolgono funzioni di servizio nel calcolatore. Costituisce la parte essenziale del cosiddetto software di sistema in quanto non svolte funzioni applicative, ma ne costituisce un supporto.
>
>Lo scopo del sistema operativo è quello di:
>- Mettere a disposizione dell'utente una macchina virtuale in grado di eseguire comandi dati dall'utente, utilizzando una macchina reale di livello inferiore.
>- Mettere a disposizione del software applicativo un insieme di servizi di sistema, invocabili tramite chiamate di sistema.
>- Controllare le risorse fisiche del sistema e creare e gestire parallelismo.

Nel modello di esecuzione sequenziale, l'esecuzione di $n$ programmi avviene in sequenza, attivando l'$i$-esimo programma solo dopo che è stata terminata l'esecuzione dell'$(i-1)$-esimo programma. Questo garantisce che non possano esistere interferenze nell'esecuzione dei vari programmi.

In un modello di esecuzione parallela si garantisce il soddisfacimento di due obiettivi contrastanti: parallelismo ed esecuzione senza interferenze.

In Linux e in molti altri SO gli obiettivi vengono soddisfatti tramite la creazione dinamica di tanti esecutori quanti sono i programmi da eseguire in parallelo.

Gli esecutori creati dinamicamente vengono chiamati processi.

>[!tip] Processi
>Un processo è un esecutore completo.
>
>Il SO è in grado di creare processi indipendenti e mette a disposizione del programmatore delle system calls che permettono di creare ed eliminare i processi.
>
>I processi sono da considerarsi macchine virtuali, cioè macchine realizzate tramite il SO e l'HW. Per questo motivo un processo non è semplicemente un programma in esecuzione, ma è associato a risorse, e il programma eseguito può essere sostituito senza annullarlo.
>
>Ogni processo è identificato in modo univoco da un PID (Process IDentifier). Tutti i processi sono creati da altri processi e quindi hanno un processo padre. Dal punto di vista del programmatore di sistema, la memoria di lavoro associata ad un processo può essere vista come costituita da tre segmenti:
>- Segmento codice.
>- Segmenti dati.
>- Segmento di sistema: contiene i dati non gestiti esplicitamente dal programma in esecuzione, ma dal SO.
>
>Esistono meccanismi ad hoc per scambiare informazione tra processi (IPC)

Il SO gestisce eventuali conflitti di accesso contemporaneo alle risorse reali condivise, creando delle risorse virtuali e accodando i processi richiedenti, pertanto il codice eseguibile di un programma non può accedere direttamente alla periferica ma deve utilizzare delle system call.