>[!note]
>Nel modello di programmazione concorrente ci sono numerosi flussi di controllo eseguiti in parallelo che interferiscono tra loro. Questo causa diversi problemi, come il non determinismo dell'esecuzione. 
>
>A differenza dell'esecuzione sequenziale, dov'è possibile testare per dimostrare la correttezza, nell'esecuzione parallela, proprio per il problema di non determinismo, si può dimostrare la correttezza solo usando la logica formale.
>
>Esistono diverse tecniche/costrutti di programmazione ad hoc per risolvere sequenze critiche, istruzioni atomiche, deadlock o stallo e sincronizzazione.

### Sequenze critiche
>[!note]
>Una sequenza è una successione sequenziale di operazioni/istruzioni eseguite dal thread. Usiamo la notazione `ti.j` con `j` istruzione eseguita dal thread `i`. Una sequenza critica è una successione di istruzioni eseguite da più thread in parallelo che non devono essere "mescolate" tra loro, affinché il risultato dell'esecuzione sia sempre corretto.
>
>Per garantire questa proprietà è necessario sequenzializzare l'esecuzione della sequenza critica tra i vari thread.
>
>La mutua esclusione è la proprietà che si vuole garantire per l'esecuzione concorrente delle sequenze critiche, affinché il risultato sia corretto.

Definiamo le istruzioni atomiche come istruzioni che, dato qualsiasi ordinamento relativo ai thread dà sempre luogo al risultato corretto.

>[!tip] Istruzioni concorrenti serializzabili
>Date due istruzioni in linguaggio di alto livello appartenenti a una sequenza ed eseguibili da due thread, si può avere `t1.i < t2.j` oppure `t2.j < t1.i`. Tuttavia si potrebbe avere anche `t1.i :: t2.j`, che indica un'esecuzione concorrente dove le istruzioni macchina che implementano le due istruzioni di alto livello sono alternate variamente. Due istruzioni C concorrenti sono serializzabili se l'implementazione concorrente in linguaggio macchina produce sempre risultati corretti.

>[!tip] Mutex
>Per garantire la correttezza dell'esecuzione concorrente di sequenze critiche o istruzioni non serializzabili, occorre implementare la mutua esclusione sulla sequenza o sull'istruzione, utilizzando dei costrutti specializzati. Nei `pthread` il costrutto è il `mutex`.
>
>Un `mutex` implementa un blocco che ingloba la sequenza/istruzione critica e il cui accesso è controllato da un segnale di occupato. Quando un thread attiva un blocco, eventuali altri thread che tentano di accedere al blocco vengono messi in attesa finché il primo non libera o rilascia il blocco.
>
>Nel linguaggio C va dichiarata una variabile di tipo `pthread_mutex_t` nel programma, e va inizializzata tramite la funzione `pthread_mutex_init`. Successivamente il `mutex` va bloccato e sbloccato tramite `pthread_mutex_lock` e `pthread_mutex_unlock`.

>[!tip] Deadlock
>In presenza di `mutex` in generale si può determinare un deadlock quando due thread devono bloccare due risorse A o B ma le bloccano in ordine inverso. Più in generale, la situazione di deadlock è rappresentata dall'esistenza di un ciclo in un grafo di attesa, dove i nodi del grafo rappresentano i thread e l'arco orientato dal nodo $i$ a $j$ rappresenta il fatto che `ti` richiede la risorsa `x` bloccata da `j`

### Sincronizzazione e semafori
>[!note]
>Per sincronizzazione si intende una relazione temporale deterministica che si vuole imporre tra due o più thread. Il costrutto specializzato per questa è il semaforo.
>
>Nella libreria `pthread` il semaforo è una variabile di tipo `sem_t`. Il semaforo viene inizializzato a un certo valore, tipicamente positivo, tramite la funzione `sem_init`. Il thread utilizza il semaforo tramite due sole funzioni `sem_wait` e `sem_post`.
>
>La funzione `sem_wait` decrementa il valore del semaforo. Se il semaforo ha valore positivo, il thread che ha invocato la `sem_wait` decrementa tale valore e prosegue nell'esecuzione. Se il semaforo ha valore nullo, allora il thread che ha invocato la `sem_wait` rimane bloccato finché il semaforo non diventa positivo, e quindi va al punto precedente.
>
>La funzione `sem_post` incrementa il valore del semaforo. Il thread che ha invocato la `sem_post` incrementa di un'unità il valore e prosegue nell'esecuzione.

Il valore del semaforo è interpretabile com il numero di risorse disponibili, ossia associate al semaforo. Se il valore è positivo, rappresenta il numero di risorse disponibili, cioè quanti thread lo possono decrementare senza rimanere in attesa.

Se il valore è nullo non ci sono risorse disponibili e ci possono essere thread in attesa.

Si ha che un semaforo inizializzato a $1$ è utilizzabile come mutex.