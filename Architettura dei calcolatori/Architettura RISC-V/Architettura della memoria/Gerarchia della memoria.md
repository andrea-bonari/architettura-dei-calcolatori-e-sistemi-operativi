>[!note]
>![[Pasted image 20250624113821.png|center]]
>
>Le tecnologie attualmente impiegate nella costruzione della memoria sono:
>
>| Nome | Velocità | Dimensioni tipiche |
>| - | - | - |
>| SRAM | $0.2\text{ ns}$ - $2.5\text{ ns}$ | Decine di $\text{kB}$ |
>| DRAM | $50\text{ ns}$ - $70\text{ ns}$ | Alcuni $\text{GB}$ |
>| Flash | $5\space\mathtt{\mu}\text{s}$ - $50\space\mathtt{\mu}\text{s}$ | Centinaia di $\text{GB}$ |
>| Dischi magnetici | $5\text{ ms}$ - $20\text{ ms}$ | Molti $\text{TB}$ |
>
>La memoria centrale e la memoria cache sono organizzate in blocchi di parole o di byte, di ugual dimensione.
>

### Memoria cache
>[!note]
>La memoria cache è un tipo di memoria molto veloce utilizzata nei computer per ridurre i tempi di accesso ai dati più frequentemente usate. Si trova tipicamente tra la CPU e la memoria centrale e ha lo scopo di memorizzare temporaneamente i dati e le istruzioni che la CPU utilizza più spesso, in modo da poterli recuperare rapidamente senza dover accedere alla RAM ogni volta.
>
>La memoria cache contiene copie di blocchi della memoria centrale, oppure blocchi liberi. Inoltre ad ogni blocchi cache è associato un bit che indica se il blocco significativo è libero.
>
>Il sistema di gestione della cache è in grado di copiare dalla memoria centrale oppure copiare alla memoria centrale.
>
>Definiamo:
>- Cache hit: accesso a dati presenti in un blocco di cache.
>- Cache miss: accesso fallito in cache, i dati devono essere recuperati nella memoria centrale.
>- Hit Rate: numero di accessi a memoria che trovano il dato in cache rispetto al numero totale di accessi.
>- Hit Time: tempo per accedere al dato in cache.
>- Miss Rate: $1-\text{Hit Rate}$.
>- Miss Penalty: tempo necessario a sostituire un blocco in cache e tempo di accesso al blocco.

Per quanto riguarda le istruzioni, se il processore deve leggere un'istruzione, se il blocco che contiene l'istruzione si trova in cache l'istruzione viene letta in un ciclo di clock e prosegue nell'esecuzione. Se l'istruzione non si trova in cache, il processore sospende l'esecuzione, il blocco contenente l'istruzione viene caricato dalla memoria centrale in un blocco libero di memoria cache e il processore riesegue la richiesta, preleva l'istruzione e prosegue l'esecuzione.

Per quanto riguarda il recupero dei dati, si procede in egual modo, tuttavia esiste il problema della coerenza tra memoria cache e memoria centrale.

>[!tip] Località dei programmi
>L'uso di memoria cache sfrutta il principio di località dei programmi:
>- Località spaziale: vengono trasferiti in cache più parole di quante non ne siano direttamente richieste.
>- Località temporale: viene sfruttata nella scelta del blocco da sostituire nella gestione di un cache miss.

Le cache si progettano in base al metodo di indirizzamento, metodo di identificazione, metodo di scrittura e metodo di sostituzione.

Definiamo il tempo medio di accesso alla memoria come: $$\text{AMAT}=\text{Hit Time}+(\text{Miss Rate}\cdot\text{Miss Penalty})$$

### Metodi di indirizzamento
>[!note]
>I metodi di indirizzamento della memoria cache descrivono come scegliere il blocco della cache in cui copiare un blocco di memoria centrale. Le tecniche principali sono:
>- Cache a indirizzamento diretto.
>- Cache completamente associative.
>- Cache set - associative.

>[!tip] Cache a indirizzamento diretto
>Nelle cache a indirizzamento diretto ogni blocco della memoria centrale è caricabile in un solo blocco della cache. Più blocchi di memoria centrale possono essere caricati nello stesso blocco di memoria cache. Si adotta quindi il metodo di mapping: il blocco di indirizzo $j$ della memoria centrale è caricabile solo nel blocco di indice: $$j\text{ mod }\text{numero di blocchi della cache}$$
>
>La struttura degli indirizzi, in questo caso è:
>- Se la memoria centrale è di $2^{n}$ parole, allora si avranno $2^{n}$ bit di indirizzo.
>- Se i blocchi di memoria centrale sono di $2^{k}$ parole/byte, allora si usano $k\text{ bit}$ dell'indirizzo per identificare una parola/byte nel blocco.
>- Se la memoria cache è di $2^{m}$ blocchi, allora $m\text{ bit}$ dell'indirizzo sono designati a identificare un blocco nella cache.
>
>I rimanenti $2^{n-m-k}$ sono i bit di etichetta, utilizzati per identificare il blocco effettivamente caricato in cache.
>
>Ogni posizione della cache quindi include:
>- `valid` bit, che indica se questa posizione contiene o meno dati validi.
>- Campo etichetta, che contiene il valore che identifica univocamente l'indirizzo di memoria corrispondente ai dati memorizzati
>- Campo dati, che contiene una copia dei dati.
>
>Dato un indirizzo di memoria centrale, l'accesso a cache avviene nel seguente modo:
>1. La parte di indirizzo che contiene l'indice del blocco in cache viene usata per selezionare il blocco.
>2. Se il blocco è valido, la parte di indirizzo che contiene l'etichetta viene confrontata con l'etichetta del blocco selezionato.
>3. Se il confronto è positivo, allora il dato è in cache e la parte di indirizzo che specifica il byte nel blocco viene utilizzata per accedere a byte corretto.
>
>
>Per gestire i cache miss l'unità di controllo del processore deve mettere in stallo la pipeline per un numero di cicli di clock non definito a priori.
>
>Si ha che il numero totale di bit nella cache a indirizzamento diretto è: $$2^{n}\cdot (\text{dimensione blocco}+\text
>{dimensione etichetta}+1)$$
>
>![[Pasted image 20250624131647.png|center]]

>[!tip] Cache associative
>Nelle cache associative un blocco può essere memorizzato in un qualunque blocco della cache, e non esiste una relazione tra indirizzo di memoria del blocco e posizione in cache.
>
>Consideriamo un indirizzo di memoria di $n\text{ bit}$ con blocchi di cache di $2^{m}\text{ B}$: Gli $m\text{ bit}$ meno significativi dell'indirizzo individuano il byte nel blocco della cache, mentre gli $n-m\text{ bit}$ più significativi individuano l'etichetta.
>
>La ricerca di un dato richiede il confronto di tutte le etichette presenti in cache, per aumentare le prestazioni la ricerca avviene in parallelo tramite una memoria associativa.

>[!tip] Cache set - associative
>Nelle cache set -  associative i blocchi della cache sono divisi in gruppi, dove ogni gruppo contiene almeno $2$ blocchi di cache. La memoria si indirizza per gruppi, e ogni blocco della memoria centrale può essere caricato in un solo gruppo, in uno qualsiasi degli $n$ blocchi. Una cache set - associativa in cui un blocco può andare in $n$ posizioni viene definita set - associativa a $n$ vie.
>
>Ogni blocco della memoria centrale corrisponde ad un unico gruppo della cache, ed il blocco può essere messo in uno qualsiasi degli elementi di questo gruppo.
>
>Un indirizzo di memoria di $n\text{ bit}$ è suddiviso in $4$ campi:
>- $b\text{ bit}$ meno significativi per individuare il byte all'interno della parola.
>- $k\text{ bit}$ per individuare la parola all'interno del blocco.
>- $m\text{ bit}$ per individuare il gruppo, indice del gruppo.
>- $n-(m+k+b)\text{ bit}$ come etichetta, quale blocco nel gruppo.
>
>Di seguito un esempio di cache set - associative a due vie.
>
>![[Pasted image 20250624133037.png|center]]

### Strategie di scrittura di un blocco
>[!note]
>Esistono due strategie di scrittura di un blocco:
>
>- Write Through: l'informazione viene scritta sia nel blocco del livello superiore, sia nel blocco di livello inferiore della memoria.
>- Write Back: l'informazione viene scritta solo nel blocco di livello superiore. Il livello inferiore viene aggiornato solo quando avviene la sostituzione del blocco.

