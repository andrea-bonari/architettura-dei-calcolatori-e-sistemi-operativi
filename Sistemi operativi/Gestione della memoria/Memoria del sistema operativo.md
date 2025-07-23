>[!note]
>La memoria virtuale del SO è gestita tramite paginazione, e come nei processi normali i suoi indirizzi virtuali sono tradotti in indirizzi fisici da MMU.
>
>Tuttavia il SO è anche il gestore della memoria fisica, e le sue tabelle delle pagine hanno dimensioni molto grandi. Pertanto esso è dipendente dalla MMU.

L'architettura `x64` ha uno spazio di indirizzamento potenziale di $2^{64}\text{ Byte}$ con indirizzi virtuali da $64\text{ bit}$, tuttavia lo spazio di indirizzamento virtuale utilizzabile è limitato a $2^{48}\text{ Byte}$. La dimensione di una pagina è $4\text{ kB}=2^{12}\text{ Byte}$, e di conseguenza ci sono $12\text{ bit}$ per lo spiazzamento. Quindi il numero di pagine virtuali è $2^{36}= \frac{2^{48}}{2^{12}}$, e quindi servono $36\text{ bit}$ per indicare il NPV.

Lo spazio di indirizzamento è diviso in due sottospazio di modo U e S ambedue da $2^{47}\text{ Byte}$. Lo spazio virtuale del kernel va da $\mathtt{FFFF\space 8000\space 0000\space0000}$ a $\mathtt{FFFF\space FFFF\space FFFF\space FFFF}$ e per linux si suddivide in 5 aree principali:

| Area                                     | Costanti simboliche | Indirizzo iniziale                              | Dimensione      |
| ---------------------------------------- | ------------------- | ----------------------------------------------- | --------------- |
| Unused space                             |                     | $\mathtt{FFFF\space 8000\space0000\space0000}$  |                 |
| Mappatura memoria fisica                 | `PAGE_OFFSET`       | $\mathtt{FFFF\space 8800\space0000\space0000}$  | $64\text{ TB}$  |
| Unused space                             |                     |                                                 | $1\text{ TB}$   |
| Memoria dinamica del kernel              | `VMALLOC_START`     | $\mathtt{FFFF\space C900\space0000\space0000}$  | $32\text{ TB}$  |
| Unused space                             |                     |                                                 |                 |
| Mappatura memoria virtuale               | `VMEMMAP_START`     | $\mathtt{FFFF\space EA00\space0000\space0000}$  | $1\text{ TB}$   |
| Unused space                             |                     |                                                 |                 |
| Codice e dati del SO                     | `_START_KERNEL_MAP` | $\mathtt{FFFF\space FFFF\space8000\space0000}$  | $0.5\text{ GB}$ |
| Area per caricare dinamicamente i moduli | `MODULES_VADDR`     | $\mathtt{FFFF\space FFFF\space A000\space0000}$ | $1.5\text{ GB}$ |
Per utilizzare gli indirizzi fisici si dedica una parte dello spazio virtuale del SO alla mappatura uno ad uno della memoria fisica. L'indirizzo iniziale di tale area virtuale è definito dalla costante `PAGE_OFFSET`.

Nel codice del SO esistono funzioni che eseguono la conversione tra indirizzi fisici e virtuali dell'area di rimappatura con gli opportuni controllo. 

Per evitare inoltre di allocare una tabella così grande per ogni processo la tabella è organizzata ad albero su $4$ livelli. I $36\text{ bit}$ del NPV sono suddivisi in $4$ gruppi, e ogni gruppo rappresenta lo spiazzamento all'interno di una tabella contenente $512$ righe. Dato che ogni PTE occupa $64\text{ bit}$, la dimensione di ogni directory è $4\text{ KB}$ e quindi occupa esattamente una pagina. L'indirizzo della directory principale è contenuto nel registro `CR3` della CPU e viene caricato ogni volta che viene mandato in esecuzione un processo.

>[!tip] Attraversamento delle pagine
>Le PTE delle tabelle attraversate non utilizzano i $12\text{ bit}$ meno significativi per indirizzare la tabella del livello successivo. I $12\text{ bit}$ meno significativi rappresentano proprietà di pagina.
>
>Quando il processore deve accedere ad un indirizzo fisico in base a un indirizzo virtuale, deve attraversare tutta la gerarchia della tabella delle pagine per trovare la PTE. Questo processo è detto Page Walk e richiede 5 accessi a memoria per accedere ad una sola cella utile di memoria.
>
>Per evitare questo inaccettabile numero di accessi `x64` possiede il TLB, che contiene le corrispondenze NPV-NPF più utilizzate recentemente. Quando viene modificato il contenuto di `cr3` la MMU invalida la TLB, escluse le righe marcate come globali, cioè condivise da più processi.
>
>La MMU cerca autonomamente nella TP le PTE di pagine non presenti. Quando viene chiesto un indirizzo virtuale la MMU verifica se la pagina P richiesta è presente nel TLB. Se è presente la accede immediatamente. In caso contrario la MMU esegue il Page Walk per recuperare la PTE relativa alla pagina P, e se questa è valida il suo contenuto viene copiato nel TLB e viene acceduta. Se anche quest'ultima non è valida si verifica un page fault.

### Paginazione in Linux
>[!note]
>La gestione della paginazione è una funzione che dipende in grande misura dall'architettura hardware. Linux utilizza un modello parametrizzato per adattarsi alle diverse architetture. La tabella delle pagine è sempre residente in memoria fisica e mappa tutto lo spazio di indirizzamento del processo, user e kernel.

In `x64` tutta la memoria è paginata, indipendentemente dal modo di funzionamento della CPU, quindi anche il SO e la stessa tabella delle pagine sono paginati. All'avviamento del sistema la tabella delle pagine non è ancora inizializzata, e quindi deve esistere un meccanismo di avviamento che permetta di arrivare a caricare tale tabella per far partire il sistema.

Non esiste una TP del SO, ma solamente quella dei processi e quindi il SO viene mappato dalla TP in ogni processo. Lo spazio virtuale è suddiviso esattamente a metà tra modo U e modo S. Questo fatto non genera ridondanza, perché tutte le metà superiori delle TP di ogni processo puntano alla stessa struttura di sottodirectory relativi al SO.

>[!tip] Occupazione della tabella delle pagine
>Nell'area virtuale del kernel una parte è utilizzata per la mappatura della memoria fisica, che consente al SO di accedere direttamente a indirizzi fisici. La dimensione effettivamente mappata è determinata dalla dimensione effettiva della memoria fisica. Il rapporto tra la dimensione della porzione di tabella delle pagine interessata e la dimensione della memoria fisica tende a $\frac{1}{512}$.