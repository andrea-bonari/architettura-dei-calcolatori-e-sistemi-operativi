>[!note]
>Le principali system call per la gestione dei processi consentono di:
>- Generare un processo figlio, cioè una copia del processo padre in esecuzione.
>- Terminare un processo figlio.
>- Attendere la terminazione di un processo figlio.
>- Sostituire il codice di un processo in esecuzione, cioè caricare in memoria ed eseguire un programma.

Un processo figlio $F$ può a sua volta generare un ulteriore processo figlio $F$. Si stabilisce così in creazione una gerarchia di processi.

>[!tip] Funzione `fork`
>La funzione `fork` crea un processo figlio identico al processo padre, che è una copia identica del padre all'istante del fork. Viene duplicato il segmento codice, il segmento dati e il segmento di sistema. L'unica differenza tra figlio e padre è il valore restituito dalla fork:
>- Nel padre, la funzione restituisce il PID del processo figlio appena generato (o $-1$ in caso di errore).
>- Nel figlio restituisce $0$.
>
>```c
>pid_t fork(void)
>```

>[!tip] Funzione `exit`
>La funzione `exit` termina il processo corrente. Si può restituire al padre il codice di terminazione. Se il processo che termina non ha più processo padre il valore viene restituito all'interprete comandi del SO.
>
>```c
>void exit(int)
>```

>[!tip] Funzione `getpid`
>La funzione `getpid` consente ad un processo di conoscere il valore del proprio PID.
>
>```c
>pid_t getpid(void)
>```

>[!tip] Funzione `wait`
>La funzione `wait` sospende l'esecuzione del processo padre che la esegue ed attente la terminazione di un qualsiasi processo figlio. Se il figlio termina prima che il padre esegua la `wait` l'esecuzione della `wait` nel padre termina istantaneamente, senza sospenderlo.
>
>Il valore restituito dalla funzione è il valore del PID del figlio terminato, mentre il parametro passato per indirizzo assume il valore del codice di terminazione del figlio.
>
>```c
>pid_t wait(int*)
>```

>[!tip] Funzione `waitpid`
>La funzione `waitpid` sospende l'esecuzione del processo padre ed attende la terminazione del processo figlio di cui viene fornito il PID. Se il figlio termina prima che il padre esegua la `waitpid` l'esecuzione della `waitpid` nel padre termina istantaneamente.
>
>Nel padre il valore restituito assume il valore del pid del figlio terminato, `status` assume il valore del codice di terminazione del figlio, e `options` specifica ulteriori opzioni.
>
>```c
>pid_t waitpid(pid_t pid, int* status, int options)
>```

>[!tip] Funzione `execl`
>La funzione `execl` sostituisce il segmento codice e il segmento dati del processo corrente con il codice e i dati di un programma contenuto in un file eseguibile specificato. Il segmento di sistema non viene sostituito.
>
>Il processo rimane lo stesso e quindi mantiene lo stesso PID.
>
>La funzione `execl` passa dei parametri al programma che viene eseguito, tramite il meccanismo di passaggio dei parametri al main `argc` e `argv`. Il valore restituito è $0$ se l'operazione è stata eseguita correttamente, e $-1$ se c'è stato un errore e l'operazione di sostituzione del codice è fallita. L'argomento `argn` deve essere sempre `NULL`.
>
>```c
>int execl(char* path_programma, char* arg0, char* arg1, ..., char* argn)
>```
>
>La sostituzione di codice non implica necessariamente la generazione di un figli, in questo caso quando il programma che è stato lanciato tramite la `execl` termina, termina anche il processo che lo ha lanciato.
>
>È quindi necessario creare un nuovo processo che effettua la sostituzione di codice quando è necessario mantenere in vita il processo di partenza dopo l'esecuzione del codice sostituito.

L'esecuzione parallela di processi può essere simulata o reale. Esiste questa distinzione poiché, anche nei sistemi multiprocessore, spesso il numero di processi eccede il numero di processori, pertanto quasi sempre il parallelismo è in parte simulato.