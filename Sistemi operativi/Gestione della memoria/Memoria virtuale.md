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

La creazione di una VMA consiste esclusivamente nella definizione dello spazio virtuale associato senza alcuna allocazione di pagine fisiche. Durante questo procedimento si predispone la porzione di tabella delle pagine necessaria a rappresentare le pagine virtuale della VMA.