>[!note]
>La memoria virtuale di un processo Linux è suddivisa in un certo numero di aree di memoria virtuale (VMA) per differenziare le diverse parti del programma eseguibile. Questo è fatto rispettivamente per:
>- Differenziare diritti di accesso.
>- Permettere ad alcune aree di crescere.
>- Condividere alcune pagine tra processi.
>
>Ogni VMA è caratterizzata da una coppia di indirizzi virtuali di pagina che ne definiscono l'inizio e la fine. Ogni area è quindi costituita da un numero intero di pagine consecutive con caratteristiche di accesso omogenee.

La VMA di un processo è composta da:
- Codice (C): istruzioni del programma da eseguire.
- Costanti per la rilocazione dinamica (K): area per parametri determinati in fase di link per il collegamento con le librerie dinamiche.
- Dati statici (S): dati inizializzati allocati per tutta la durata di un programma.
- Dati dinamici (D): dati allocati dinamicamente su richiesta del programma. Il limite è definito dalla variabile `brk`.
- Pile dei thread (T): aree utilizzate per le pile dei thread.
- Area per Memory-Mapped files (M): permette di mappare un file in una parte della memoria virtuale del processo per poter essere letto o scritto come un array di byte in memoria.
- Memoria condivisa: area dati di un processo che può essere acceduta anche da altri processi. Lo stesso file mappato in memoria da diversi processi consente di realizzare un meccanismo di condivisione della memoria, in lettura o scrittura.
- Librerie dinamiche: librerie non incorporate staticamente nel programma eseguibile dal linker ma vengono caricate in memoria durante l'esecuzione del programma in base alle esigenze.
- Pila: area di pila di modo U del processo, che contiene tutte le variabili ad allocazione automatica delle funzioni di un programma C.

```c
struct vm_area_struct {
	struct mm_struct* vm_mm;
	
	unsigned long vm_start;
	unsigned long vm_end;
	
	struct vm_area_struct *vm_next, *vm_prev;
	
	unsigned long vm_flags;
	
	...
	
	unsigned long vm_pgoff;
	
	struct file *vm_file;
}
```

>[!tip] Backing Store Segment
>Un'area virtuale di memoria può essere mappata su file detto backing store. In caso contrario l'area è detta anonima.
>
>La struttura `vm_area_struct` contiene l'informazione `struct file * vm_file` per il puntatore al file utilizzato come backing store e la posizione all'interno del file `unsigned long vm_pgoff`.
>
>Per le aree C, K ed S il backing store è il file eseguibile, con offset il punto del file eseguibile dove inizia il corrispondente segmento.

Esistono due tipi di aree di memoria virtuale:
- Mapping basato su file (Codice, librerie, file, dispositivi di I/O)
- Mapping anonimo (Pila, Heap, Pagine Copy on write)

La creazione di una VMA consiste esclusivamente nella definizione dello spazio virtuale associato senza alcuna allocazione di pagine fisiche. Durante questo procedimento si predispone la porzione di tabella delle pagine necessaria a rappresentare le pagine virtuale della VMA. Per ogni pagina virtuale la corrispondente PTE indicherà che la pagina non è fisicamente allocata.

L'allocazione delle pagine fisiche avviene solamente quando un processo legge o scrive una pagina virtuale NPV. In questo caso nella PTE associata a tale NPV viene inserito in tale caso il numero di NPF di pagina fisica.

È quindi importante distinguere tra le operazioni che modificano esclusivamente lo spazio virtuale del processo e e le operazioni che allocano memoria fisica.

>[!tip] Page Fault
>Quando la MMU genera un interrupt di page fault per una pagina virtuale NPV viene attivata la routine di risposta Page Fault Handler.
>
>Assumiamo che la tabella delle pagine contenga solo le pagine virtuali appartenenti alle VMA del processo. Distinguiamo se una pagina virtuale è residente in memoria tramite il flag `P=1`.
>
>Linux crea sempre $34$ pagine di pile, e all'exec di un programma Linux invalida tutta la tabella delle pagine del processo.
>
>La tecnica utilizzata per il caricamento delle pagine in memoria fisica è il demand paging, cioè il caricamento delle pagine necessarie in base alle richieste mediante page fault.
>
>Al momento dell'esecuzione di un programma la VMA viene creata con $34$ pagine virtuali iniziali non allocate fisicamente, tranne quelle utilizzate immediatamente che sono allocate fisicamente.
>
>Il tentativo di accedere ad una pagina virtuale posta subito dopo l'area esistente (pagina di growsdown) produce una crescita automatica dell'area, mentre il tentativo di accedere pagine diverse da quelle esistenti o quella di growsdown produce una segmentation fault.

>[!tip] Heap
>La memoria dati dinamica (heap) cresce su richiesta del programma. Normalmente in C si usa la funzione `malloc`, che alloca un nuovo spazio nella VMA D tramite il servizio di sistema `brk` o `sbrk`.
>
>```c
>void* sbrk(intptr_t incremento)
>```
>Incrementa l'area dati dinamici del valore `incremento`.
>
>```c
>int brk(void* end_data_sergment)
>```
>Assegna il valore `end_data_segment` al campo `end` della VMA dei dati dinamici, restituisce $0$ se ha successo, $-1$ se errore.

### Creazione di nuovi processi
>[!note]
>Si ha che il nuovo processo creato è un'immagine del padre. Linux aggiorna la tabella dei processi ma non alloca nuova memoria fisica.
>
>Il nuovo processo condivide la memoria del padre tranne quella di pila, che contiene la variabile pid.
>
>Quando uno dei due processi cerca di scrivere in memoria si duplica la pagina. Questa tecnica è detta copy on write. Per identificare un'operazione di scrittura tutte le pagine del padre condivise con il figlio al momento della fork vengono modificate nei diritti di accesso solo lettura.

Ad ogni pagine fisica è associato un descrittore di pagina che contiene un contatore `ref_count` che indica il numero di utilizzatori della pagina fisica.

Dopo una fork le pagine duplicate fisicamente sono solo quelle scritte durante l'esecuzione della fork.

In Linux nella Copy on Write la pagina fisica originale è attribuita al processo figlio, mentre per il processo padre viene allocata una nuova pagina fisica.

>[!tip] Exec
>L'esecuzione della `exec` pone a "non valido" tutte le pagine relative a codice e dati del processo. La struttura delle VMA viene riscostruita in base al contenuto del file eseguibile e vengono predisposte le PTE necessarie nella TP.
>
>Viene quindi caricata in memoria fisica la pagina di codice contenente la prima istruzione eseguibile, la prima pagina di pila nella quale vengono posti i parametri passati al main e il programma inizia l'esecuzione.

>[!tip] Context switch ed exit
>Il context switch causa lo svuotamento del TLB, e copia il valore del dirty bit associato alle pagine del TLB nel descrittore della pagina fisica `page_descriptor`.
>
>La procedura di exit elimina la struttura delle VMA del processo, elimina la tabella delle pagine del processo e dealloca le pagine virtuali del processo con `ref_count=1` liberando le pagine fisiche. Infine si esegue il context switch con relativo flush del TLB.

>[!tip] Thread
>I thread condividono la stessa struttura di aree virtuali e tabella delle pagine del processo padre.
>
>In Linux la creazione di ogni thread alloca un'area di tipo T di $2048$ pagine, cioè $8\text{ MB}$ per ogni pila di thread. Inserisce poi una pagina in sola lettura come area di interposizione tra ogni pila come protezione.
>
>La pila del primo thread inizia logicamente al NPV $\mathtt{7FFF\space F77F\space F}$ e si sviluppa verso indirizzi più bassi. Quindi la sua VMA è compresa tra $\mathtt{7FFF\space F6FF\space F}$ e $\mathtt{7FFF\space F77F\space F}$, e quindi la pila del secondo thread inizia logicamente al NPV $\mathtt{7FFF\space F6FF\space E}$ e si sviluppa verso indirizzi più bassi, e quindi la sua VMA è compresa tra $\mathtt{7FFF\space F67F\space E}$ e $\mathtt{7FFF\space F6FF\space E}$.

### Creazione di una VMA
>[!note]
>Le VMA possono essere create dal sistema operativo tramite la system call `mmap`. Come detto in precedenza le VMA possono essere classificate in base a due criteri: Mappate su file o anonime, e shared o private.
>
>Consideriamo i tipi di aree shared e mappate su file, private e mappate su file e anonime e private.

```c
#include <sys/mman.h>

void* mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
```

Le page cache sono l'insieme di pagine fisiche utilizzate per rappresentare i file in memoria e le strutture dati e funzioni necessarie per la gestione.

Quando un processo richiede di accedere ad una pagina virtuale mappata su file esso
- Determina il file indicato nella VMA e l'offset in pagine
- Verifica se esiste nella Page Cache. Se esiste mappa una pagina virtuale su pagina fisica, se non esiste alloca una pagina fisica nella Page Cache e carica la pagina del file richiesta.

>[!tip] VMA shared mappata su file
>I dati vengono scritti sulla pagina della page cache condivisa. Quindi la pagina fisica viene modificata, tutti i processi che mappano tale pagina fisica vedono immediatamente tutte le modifiche, e la pagina modificata verrà scritta su disco una volta sola al termine dell'utilizzo.
>
>Per quanto riguarda il meccanismo di copy on write le pagine delle VMA di tipo scrivibile vengono abilitate solo in lettura. Quando un processo P scrive su una di queste pagine si verifica un page fault per violazione di protezione. Quindi il page fault handler alloca una nuova pagina fisica, e ci copia il contenuto della pagina che ha generato la violazione. Pone quindi nella tabella delle pagine la corrispondenza NPV - NPF e modifica il diritto di accesso in scrittura.

Tutte la pagine nelle page cache non vengono liberate neppure quando tutti i processi che le utilizzavano non le usano più. Linux applica il principio di mantenere in memoria le pagine lette da disco il più a lungo possibile, per evitare successivi accessi a disco se richiesti nuovamente.

Le pagine in page cache vengono liberate in base alla politica di gestione della memoria quando si arriva ad una situazione di memoria quasi piena. In questo caso vengono salvate sul file del disco.

>[!tip] VMA anonime
>Le aree anonime non hanno un file associato, e quindi la definizione di un'area anonima non alloca memoria fisica. Le pagine virtuali sono tutte mappate sulla ZeroPage (pagina fisica di tutti $0$ mantenuta dal SO). Durante le operazioni di lettura trovano una pagina inizializzata a $0$ e non allocano una pagina fisica, mentre durante le operazioni di scrittura attivano il sistema di copy on write.
>
>Le pagine anonime se devono essere scaricate dalla memoria vengono salvate nello swap file.

Le VMA di codice sono mappate su file e sono private. Se due processi eseguono contemporaneamente lo stesso codice le pagine di codice sono condivise e il secondo processo trova le pagine in Page Cache.

Il linker dinamico utilizza le VMA per la condivisione delle pagine fisiche delle librerie condivise. Il linker crea le VMA per le librerie utilizzando `MAP_PRIVATE`.