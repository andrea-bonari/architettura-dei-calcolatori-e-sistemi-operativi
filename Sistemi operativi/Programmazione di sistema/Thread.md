>[!note]
>Un thread è un attività parallela che consente "strutturalmente" un grado di cooperazione elevato. Talvolta il thread è chiamato processo leggero. La gestione del thread da parte del SO è meno onerosa rispetto alla gestione del processo.
>
>Possiamo dire che il thread ha un flusso di controllo che può essere svolto in parallelo con altri thread nell'ambito di uno stesso processo. Questo flusso di controllo è una funzione attivata alla creazione del thread. Poiché i thread sono attività parallele in uno stesso processo condividono lo spazio di indirizzamento.

Esistono svariati modelli implementativi di thread, qui si considera lo standard POSIX.

Il thread è attivato nell'ambito del processo, e quando il processo termina terminano forzatamente anche tutti i suoi thread, qualunque sia il loro punto di esecuzione. È necessario garantire la terminazione coerente dei thread.

Ogni thread ha un identificatore di thread, di tipo `pthread_t`, e come per i processi ogni thread può essere posto in attesa di un evento.

>[!tip] Funzione `pthread_create`
>La funzione `pthread_create` è simile a `fork`, ma il thread viene creato passandogli il nome della funzione che esso deve eseguire.
>```c
>pthread_create (&tID, NULL, tf, (void*) n)
>```
>


>[!tip] Funzione `pthread_join`
>La funzione `pthread_join` è simile a `waitpid`, ma si deve sempre specificare il thread la cui terminazione si vuole attendere, e il suo uso è obbligato per garantire la terminazione coerente dei thread quando termina il processo che li ha creati.
>
>Dato che il thread esegue una funzione, termina quando esegue `return` o alla fine del codice eseguibile della funzione.
>
>Il thread che termina può passare, tramite `return` un codice di terminazione al thread che lo ha creato. Il codice di terminazione è passato tramite il secondo parametro di `join`.
>
>```c
>pthread_join (tID, (void*) &thread_exit)
>```

Il thread viene creato nell'ambito di un processo che ha un suo flusso di controllo. Si prende `main` come thread principale o di default. Quando un codice eseguibile viene lanciato, nel processo viene creato un solo thread, chiamato "thread principale", che a sua volta può creare altri thread. Ciascun thread ha una sua pila, indipendente dalle pile degli altri thread e allocata in un (sotto)spazio di indirizzamento diverso.
