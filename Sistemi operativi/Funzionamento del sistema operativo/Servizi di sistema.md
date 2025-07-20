>[!note]
>I servizi di sistema sono un insieme di routine messe a disposizione dal SO, che sono invocabili tramite `syscall`.

La funzione `syscall` nella libreria `glibc` è dichiarata come segue:

```c
long syscall(long num_del_servizion, param_del_servizio, ...)
```

Per convenzione ogni chiamata è associata ad un numero del servizio richiesto. In corrispondenza al numero del servizio si associa una costante simbolica dal prefisso `sys` seguito dal nome del servizio.

>[!tip] Convenzione sui nomi
>È difficile conoscere il nome della system call service routine che esegue un determinato servizio di sistema. Talvolta ha lo stesso nome del servizio, ma in molti casi ha un nome diverso, legato anche all'evoluzione del sistema.
>
>Per semplificare noi assumiamo che il nome della system call service routine che esegue un servizio sia uguale al nome della costante simbolica utilizzata per individuare il servizio nella chiamata della funzione `syscall` di `glibc`. Pertanto per noi le system call service routine hanno un nome costituito dal prefisso `sys_` seguito dal nome del servizio.
### Avviamento
>[!note]
>Al momento dell'avviamento (bootstrap) devono avvenire le seguenti operazioni:
>- Inizializzazione di alcune strutture dati del SO.
>- Creazione di un processo iniziale chiamato `idle` (PID 1).
>  
>Deve infatti esistere almeno un processo iniziale che viene creato direttamente dal sistema operativo. Le operazioni di avviamento successive alla creazione del processo 1 sono svolte dal programma `init`, cioè un normale programma non privilegiato che differisce dagli altri processi solamente per il fatto di non avere un processo padre.
>
>Il processo 1, eseguendo init crea un processo per ogni terminale sul quale potrebbe essere eseguito un login. Questa operazione è eseguita leggendo il file di configurazione (`inittab`). Quando un utente si presenta al terminale ed esegue il login, se l'identificazione va a buon fine, allora il processo che eseguiva il programma di login lancia in eseuzione il programma `shell`, e da questo momento la situazione è quella di normale funzionamento.

Il processo `idle`, dopo aver concluso le operazioni di avviamento assume le seguenti caratteristiche:
- I suoi diritti di esecuzione sono sempre inferiori a quelli di tutti gli altri processi.
- Non ha mai bisogno di sospendersi tramite `wait_xxx`.

### System Call Interface
>[!note]
>Prima di invocare la `SYSCALL`, per effettuare il passaggio di parametri dalla funzione `syscall` alla funzione del kernel `system_call` occorre passare il numero del servizio da invocare nel registro `rax`, e passare eventuali parametri ordinatamente nei registri `rdi`, `rsi`, `rdx`, `r10`, `r8` e `r9`.
>
>La funzione `syscall` invoca il kernel tramite l'istruzione `SYSCALL` che invoca la funzione del kernel `system_call`. L'indirizzo al quale la CPU data all'interno del kernel è quello della funzione `system_call`. A sua volta la `system_call` invoca la System Call Service Routine opportuna `sys_xxx` per eseguire il servizio `xxx` in base al numero presente nel registro `rax`.

### Creazione di processi
>[!note]
>Attualmente la libreria più utilizzata per i thread in Linux è la Native POSIX Thread Library.
>
>I processi normali sono creati quando si esegue una `fork`, quelli leggeri quando si esegue una `pthread_create`, e in entrambi i casi la creazione è effettuata dal padre.
>
>Il processo figlio potrà condividere con il padre una serie più o meno estesa di proprietà e componenti, mentre il processo leggero creato da una `pthread_create` condivide con il padre chiamante una serie di componenti, di cui noi consideriamo solamente la memoria e la tabella dei file aperti.
>
>Entrambe le funzioni `fork` e `pthread_create` sono realizzate dal nucleo Linux tramite una sola e complessa system call service routine, la `sys_clone`.

>[!tip] Funzione `clone`
>La funzione di libreria `glibc` `clone` crea un processo con caratteristiche di condivisione definite analiticamente tramite una serie di flag.
>
>```c
>int clone (int (*fn)(void *), void* child_stack, int flags, void* arg, ...)
>```
>
>`int (*fn)(void *)` indica che si tratta di un puntatore a una funzione che riceve un puntatore a void come argomento e restituisce un intero, `void* arg` è il puntatore ai parametri da passare alla funzione `fn` e  `void* child_stack` è l'indirizzo della pula utente che verrà utilizzata dal processo figlio.
>
>I flag sono piuttosto numerosi, ci limitiamo a trattare:
>- `CLONE_VM`: indica che i due processi utilizzano lo stesso spazio di memoria.
>- `CLONE_FILES`: indica che i due processi devono condividere la tabella dei file aperti.
>- `CLONE_THREAD`: indica che il processo viene creato per implementare un thread.
>
>La funzione `clone` eseguita dal processo padre crea un processo figlio che esegue la funzione `fn(*arg)`, ha una sua pila utente dislocata all'indirizzo `child_stack`, condivide con il padre gli elementi indicati dai flag presenti nella chiamata. Inoltre se è specificato il flag `CLONE_THREAD` avrà lo stesso TGID del chiamante.
>
>La funzione `pthread_create` è implementata in maniera molto diretta invocando la funzione `clone` nel modo seguente.
>
>```c
>char* pila = malloc(...);
>
>clone(fn, pila, CLONE_VM, CLONE_FILES, CLONE_THREAD, ...);
>```
>
>Il processo figlio condivide memoria e file con il padre, e lo spazio per la pila utente del thread viene allocato all'interno della memoria dinamica del processo stesso.

La system call service routine `sys_clone` è pensata per creare un processo figlio normale e quindi assomiglia alla funzione `fork`.

```c
long sys_clone (unsigned long flags, void* child_stack, void* ptid, void* ctid, struct pt_regs* regs);
```

Se `child_stack` è $0$, allora il figlio lavora su una pila che è una copia fisica del padre posta allo stesso indirizzo virtuale. In questo caso `CLONE_VM` non dev'essere specificato, altrimenti non è garantita la correttezza del funzionamento. In caso contrario, il figlio lavora su una pila posta all'indirizzo `child_stack` e tipicamente la memoria non viene condivisa.


>[!tip] Eliminazione dei processi
>Esistono due system call service routine relative alla cancellazione dei processi: `sys_exit` e `sys_exit_group`.
>
>Il servizio `sys_exit_group` è implementato nel modo seguente: invia a tutti i membri del gruppo il signal di terminazione ed esegue una normale `sys_exit`.
>
>Il servizio `sys_exit` deve rilasciare le risorse utilizzate dal processo, restituire un valore di ritorno al processo padre, e invocare la funzione `schedule` per lanciare in esecuzione un nuovo processo.
>
>La terminazione di un singolo thread può avvenire per esecuzione dell'istruzione `return` e per invocazione di `pthread_exit`. Viene comunque realizzata utilizzando il servizio `sys_exit`.
>
>La funzione C di libreria `glibc` `exit` è usata per terminare un processo con tutti i suoi thread, ed è implementata invocando il servizio `sys_exit_group`.

### Accesso allo spazio U dal SO
>[!note]
>I singoli servizi di SO devono talvolta leggere o scrivere dati nella memoria utente del processo che li ha invocati. Linux fornisce una serie di macro assemble utilizzabili per questo scopo (`Linux/arch/x86/include/asm/uaccess.h`).

