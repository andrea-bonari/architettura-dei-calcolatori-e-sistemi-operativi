>[!note]
>Linux è il kernel di un sistema operativo open source basato su Unix, sviluppato da Linus Torvalds. Siccome Linux è soltanto il nome del kernel, e non del sistema operativo nel complesso, viene spesso accoppiato con il sistema GNU e quindi il termine appropriato per riferirsi al SO è GNU/Linux.

Linux è generalmente applicato ai campi dei:
- Personal Computer
- Server
- Sistemi embedded
- Sistemi real-time
- Sistemi mobili

La funzione principale di Linux è la realizzazione di un ambiente di esecuzione dei programmi applicativi costituito da un insieme di processi.

### Parallelismo tra processi
>[!note]
>Linux è un SO multiprogrammato time-sharing, e quindi la virtualizzazione del parallelismo è ottenuta facendo eseguire in alternanza i diversi processi dall'unico processore.
>
>A ogni processo è assegnato un quanto di tempo, e alla sua scadenza il processo è sospeso dall'esecuzione (preemption) e un nuovo processo può utilizzare il processore.
>
>In Linux definiamo un thread come processo leggero, un processo normale come processo non leggero, e processo (o task) come processo generico. Come nei thread i processi leggeri appartenenti allo stesso processo normale condividono tutti la stessa memoria, eccetto la pila.
>
>Il kernel Linux è non-preemtable, e quindi non può essere sospeso dall'esecuzione.

Siccome lo standard POSIX richiede che tutti i thread di uno stesso processo condividano lo stesso PID è necessario adattare i due modelli.

In Linux un thread group è l'insieme di processi leggeri che appartengono allo stesso processo normale. Ogni processo possiede quindi un TGID oltre al PID, che è uguale al PID del primo processo del gruppo. Quindi ad ogni processo è associata la coppia di identificatori $(\text{PID},\text{TGID})$ dove `gettid` restituisce il PID del processo, e `getpid` restituisce il TGID.

La sostituzione di un processo in esecuzione con un altro è chiamata commutazione di contesto, e con il termine contesto di un processo si intende l'insieme di informazioni relative ad ogni processo che il SO gestisce. Durante questa operazione il processo che lascia l'esecuzione deve salvare la parte di suo contesto in memoria, mentre il processo che entra in esecuzione deve avere ripristinato il suo contesto.

>[!tip] Scheduler
>Il componente del SO che decide quale processo mandare in esecuzione è detto scheduler.
>
>La politica di scheduling da lui attuata prevede che i processi più importanti siano eseguiti con priorità, e che i processi di pari importanza siano eseguiti in maniera equa.
>
>Lo scheduler si occupa anche di attuare la commutazione di contesto.

### SMP
>[!note]
>L'architettura multiprocessore meglio supportata da Linux attualmente è Symmetric Multiprocessing (SMP), in cui ci sono $n\geq2$ processori identici collegati ad una singola memoria centrale, che hanno tutti accesso a tutti i dispositivi periferici, sono controllati da un singolo sistema operativo e vengono considerati identici.
>
>Nel caso di processori multicore il concetto di SMP si applica ai singoli core, considerandoli come processori diversi.

In Linux, SMP utilizza una gestione statica e una riallocazione dei task tra le CPU per bilanciamento del carico di lavoro.

Per quanto riguarda la gestione statica delle task, consiste nell'allocare ogni task a una singola CPU. Lo spostamento di una task da una CPU a un'altra richiede di svuotare la cache, e quindi introduce un ritardo nell'accesso a memoria finché i dati non sono stati caricati nella cache della nuova CPU.

Per evitare che un processo possa svolgere azioni dannose per il sistema stesso o per altri processi il SO svolge un aspetto di limitazione delle azioni che un processo può svolgere, usando alcuni meccanismi Hardware.

>[!tip] Adattamento di Linux alle periferiche
>L'accesso alle periferiche avviene assimilandole a dei file, in modo che un programma non abbia necessità di conoscere i dettagli realizzativi della periferica stessa. Questa implementazione è diversa dal driver della periferica che si occupa di gestire le caratteristiche della periferica stessa.
>
>Per rendere semplice l'evoluzione del sistema per supportare una nuova periferica sono utili 2 caratteristiche:
>- Poter aggiungere con facilità al SO il device driver.
>- Permettere di configurare un sistema solo con i gestori delle periferiche effettivamente utilizzati.
>  
>Per permettere tutto Linux fornisce la possibilità di inserire nel sistema dei kernel_modules, senza necessità di ricompilare il sistema.

>[!tip] Creazione di un modulo kernel
>Di seguito una semplice implementazione un kernel_module.
>
>```c
>#include <linux/init.h>
>#include <linux/module.h>
>
>static int __init acso_init(void) {
>	printk("Inserito modulo ACSO Hello\n");
>	return 0;
>}
>
>module_init(acso_init);
>
>static void __exit acso_exit(void) {
>	printk("Rimosso modulo ACSO Hello\n");
>}
>
>module_exit(acso_exit);
>```
>
>Per compilare il modulo si usa il comando `make`. Per caricarlo si usa `insmod ./acso_hello.ko`, e per rimuoverlo si usa `rmmod acso_hello`.

### Dipendenza dall'architettura hardware
>[!note]
>Linux è scritto in linguaggio C e compilato con il compilatore `gcc`. Alcune funzioni dipendono dalle peculiarità dell'hardware e sono implementate in maniera diversa per le diverse architetture. Nella fase di link del SO per una cerca architettura viene scelta l'implementazione opportuna tramite un `makefile`.

Faremo riferimento all'architettura `x86-64, long 64 bit`, e chiameremo `x64` un processore dotato di questa architettura.

### Strutture dati per la gestione dei processi
>[!note]
>L'informazione relativa ad ogni processo è rappresentata in strutture dati mantenute dal SO.
>Le strutture dati utilizzate per rappresentare/salvare il contesto di un processo sono una struttura dati chiamata descrittore del processo, il cui indirizzo costituisce un identificatore univoco, e una pila del sistema operativo del processo,

Il descrittore del processo è implementato nel kernel linux nel file `linux/sched.h`, ed è implementato come segue:

```c
struct task_struct {
	pid_t pid;
	pid_t tgid;
	
	volatile long state;    // -1 unrunnable, 0 runnable, >0 stopped
	
	void* stack;    // puntatore alla pila del SO del task
	
	struct thread_struct thread; // contiene il contesto HW
	
	// ... variabili utilizzate per lo scheduling
	
	struct mm_struct* mm;    // puntatori alle strutture usate nella gestione della memoria
	
	// variabili per la restituzione dello stato di terminazione
	int exit_state;
	int exit_code, exit_signal;
	
	struct fs_struct* fs;    // informazioni filesystem
	struct files_struct* files    // informazioni file aperti
}
```

La struttura per il contesto hardware ha definizione diversa per ogni architettura (`linux/arch/x86/include/processor.h`) e contiene le informazioni del contesto HW della CPU del processo quando non è in esecuzione.

```c
struct thread_struct {
	// ...
	
	unsigned long sp0;    // puntatore alla base dela pila del SO del processo
	
	unsigned long sp;    // puntatore alla posizione corrente della pila di SO
	unsigned long usersp;    // puntatore alla pila di modo U
}
```

Un kernel_module può accedere al proprio `task_struct` usando il metodo `get_current`, che restituirà un puntatore a quella struct.