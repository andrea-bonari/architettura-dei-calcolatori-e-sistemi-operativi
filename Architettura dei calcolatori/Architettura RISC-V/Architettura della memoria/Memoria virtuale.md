>[!note]
>Gli indirizzi a cui fa riferimento il programma in esecuzione sono indirizzi virtuali. Lo spazio di indirizzamento virtuale è quello definito dalla lunghezza degli indirizzi virtuali.
>
>In un programma definiamo la dimensione virtuale come la somma delle aree di programma effettivamente presenti. La dimensione iniziale è calcolata dal linker dopo la generazione del codice eseguibile, tuttavia può variare durante l'esecuzione a causa della modifica della pila per le chiamate a sottoprogrammi, a causa della modifica dell'heap per la gestione delle strutture date dinamiche, oppure per la creazione di aree per la mappatura di librerie dinamiche o aree condivise con altri processi.
>
>Siccome gli indirizzi di memoria centrale sono indirizzi fisici, la memoria virtuale e la memoria fisica non coincidono. È quindi presente un meccanismo di traduzione automatica degli indirizzi virtuali in indirizzi fisici, questo processo è detto memory mapping.

Avere degli indirizzi virtuali implica che ogni programma eseguibile generato dal linker sia in formato rilocabile. La traduzione automatica degli indirizzi è attuata durante l'esecuzione da un componente hardware detto MMU, ed è completamente trasparente al programmatore, al compilatore e al linker. 

Questa traduzione consente il caricamento del programma in una qualsiasi locazione della memoria fisica.

>[!tip] Processi
>Un processo è un modello di esecuzione parallela, ed è un esecutore completo creato dinamicamente dal sistema operativo al lancio di un programma in esecuzione. È da considerarsi una macchina virtuale.
>
>Un processo non è semplicemente un programma in esecuzione, è associato a risorse e può essere sostituito senza annullare il processo stesso.

### Paginazione
>[!note]
>La paginazione è un meccanismo che suddivide la memoria virtuale del programma in porzioni di lunghezza fissa dette pagine virtuali, aventi una lunghezza che è potenza di due.
>
>Anche la memoria fisica viene suddivisa in pagine fisiche della stessa dimensione. Le pagine virtuali di un programma da eseguire vengono caricate in altrettante pagine fisiche, prese arbitrariamente e non necessariamente contigue.

Lo spazio di indirizzamento virtuale di ogni programma è lineare, ed è suddiviso in un numero intero di pagine di dimensione fissa. L'indirizzo virtuale può essere visto quindi come il numero di pagina virtuale (NPV) e lo spiazzamento nella pagina.

Lo spazio di indirizzamento fisico viene suddiviso in un numero intero di pagine di uguale dimensione di quelle utilizzate per lo spazio di indirizzamento virtuale. Ogni pagina della memoria può quindi contenere esattamente una pagina dello spazio di indirizzamento virtuale.

L'indirizzo fisico può essere visto quindi come il numero di pagina fisica (NPF) e lo spiazzamento nella pagina.

La corrispondenza tra pagine virtuali e pagine fisiche è generata da una tabella delle pagine associata al processo.

>[!tip] Condivisione delle pagine
>I diversi processi possono condividere delle pagine. Per ogni pagina condivisa esiste una riga nella corrispondente tabella delle pagine del processo.
>
>I valori di NPF relativi alle pagine condivise sono identici nelle righe di ogni processo che le condivide.

Il meccanismo di paginazione consente di rilevare, durante l'esecuzione, un accesso a zone di memoria che non appartengono allo spazio di indirizzamento virtuale del processo in esecuzione.

È possibile associare ad ogni pagina virtuale di un processo alcuni bit di protezione, che definiscono le modalità di accesso consentite per quella pagina. I diritti di accesso risultano particolarmente significativi nel caso di pagine condivise.

### Memory Management Unit (MMU)
>[!note]
>La traduzione del numero di pagina virtuale nel corrispondente numero di pagina fisica avviene mediante un dispositivo specializzato, la Memory Management Unit (MMU), che può essere posizionata nel chip della CPU, o in un chip separato.
>
>La MMU ha al suo interno una memoria molto veloce che contiene, in formato opportuno, le tabelle delle pagine dei processi. In generale contiene solo una parte della tabella delle pagine di ogni processo, e in aggiunta un campo della tabella contiene il PID del processo stesso.

La memoria della MMU è realizzata come memoria associativa: la selezione di riga avviene come associazione sul contenuto di opportuni campi della riga stessa, e il descrittore utilizzato è la coppia (PID, NPV) per ottenere il numero della pagina fisica corrispondente.

La dimensione della tabella associativa è pari a: $$\text{Processi in esecuzione}\cdot R$$
Dove $R$ è il numero di pagine residenti in memoria centrale per ogni processo.

>[!tip] Gestione delle pagine virtuali non residenti in memoria
>Durante l'esecuzione di un processo, un certo numero di pagine virtuali è caricato in memoria di lavoro, in caso di page fault, tramite un interrupt di mancanza di pagina il controllo deve essere passato al sistema operativo con il meccanismo dell'eccezione.
>
>Il processo di esecuzione è quindi interrotto e il sistema operativo deve:
>1. Rintracciare su disco la pagina virtuale richiesta.
>2. Trovare spazio in memoria per caricare la pagina richiesta. Questa operazione può implicare di dover scaricare da memoria un'altra pagina.
>3. Caricare la pagina da disco.
>4. Rieseguire l'istruzione che aveva generato il page fault.

>[!tip] Working set
>La maggior parte dei programmi non accede al suo spazio di indirizzamento in modo uniforma ma gli accessi tendono a raggrupparsi in un numero limitato di pagine.
>
>Si definisce quindi il working set come un insieme di pagine che in ogni istante rappresentano le pagine a cui hanno fatto accesso le ultime $k$ richieste in memoria. Se $k$ è sufficientemente grande, il working set di un programma varia molto lentamente per il principio di località.
>
>Il numero $R$ i pagine residenti in ogni processo è ottenuto da una stima del working set.

La memoria virtuale di un processo non è contenuta completamente nella memoria fisica durante l'esecuzione, e di conseguenza la parte non contenuta in memoria fisica deve esistere sul disco. Si usa quindi un dirty bit, che è posto a $1$ quando si accede alla pagina fisica corrispondente in scrittura, per gestire la ricopiatura della pagina in memoria su disco.

### Translation Lookaside Buffer (TLB)
>[!note]
>La Translation Lookaside Buffer è una cache che tiene traccia delle traduzioni (NPV, NPF). In caso di hit legge NPF e costruisce l'indirizzo fisico con offset, in caso contrario si usa la tabella della pagine.

