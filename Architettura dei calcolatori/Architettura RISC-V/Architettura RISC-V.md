>[!note]
>Analizziamo un'implementazione semplificata del processore RISC-V, a cui è associato il corrispondente set di istruzioni.
>
>L'architettura RISC-V utilizza l'architettura pipeline, una tecnica per migliorare le prestazioni basata sulla sovrapposizione dell'esecuzione di più istruzioni appartenenti ad un flusso di esecuzione sequenziale.

L'insieme ridotto di istruzioni contiene le categorie `R`, `I`, `S` e `B`, non considera le istruzioni nei formati `U` e `J`.

### Esecuzione delle istruzioni
>[!note]
>Il formato di istruzioni completo si compone di:
>- opcode ($7\text{ bit}$): è identico per gruppi di istruzioni
>- numero di registro ($5\text{ bit}$)
>- `funct3` e `funct7`: estensioni del codice operativo per gestire le operazione dell'ALU
>- immediato ($12\text{ bit}$ o $20\text{ bit}$): valore numerico in complemento a $2$.

>[!tip] Esecuzione istruzioni aritmetico logiche
>Un'istruzione aritmetico logica viene eseguita in 4 passi:
>- Prelievo istruzione dalla memoria istruzioni e incremento del `pc`.
>- Lettura dei $2$ registri sorgente dal banco dei registri, utilizzando i bit $19$-$15$ e $24$-$20$ per selezionarli.
>- Operazione della ALU sui dati letti dal banco registri, utilizzando il campo `funct3` e `funct7` per realizzare la funzione aritmetico-logica.
>- Scrittura dell'ALU nel banco dei registri utilizzando i bit `11`-`7` dell'istruzione per selezionare il registro di destinazione.

>[!tip] Esecuzione istruzioni `load`
>Un'istruzione di `load` viene eseguita in 5 passi:
>- Prelievo istruzione dalla memoria istruzioni e incremento del `pc`.
>- Lettura del registro base dal banco dei registri, usando i bit $19$-$15$.
>- Operazione dell'ALU per calcolare la somma del valore letto dal registro base e dell'immediato a $64\text{ bit}$, costruito opportunamente dai $12\text{ bit}$ di offset.
>- Prelievo del dato nella memoria dati utilizzando come indirizzo di lettura il risultato dell'ALU.
>- Scrittura del dato proveniente dalla memoria nel banco dei registri, il registro di destinazione è indicati dai bit $11$-$17$.

>[!tip] Esecuzione istruzioni `store`
>Un'istruzione di `store` viene eseguita in 4 passi:
>- Prelievo istruzione dalla memoria istruzioni e incremento del `pc`.
>- Lettura del registro base (bit $19$-$15$) e del registro sorgente (bit $24$-$20$) del banco dei registri.
>- Operazione dell'ALU per calcolare la somma del valore letto dal registro base e dell'immediato a $64\text{ bit}$, costruito opportunamente dai $12\text{ bit}$ di offset.
>- Scrittura del dato proveniente dal registro sorgente nella memoria dati utilizzando come indirizzo di scrittura il risultato dell'ALU.

>[!tip] Esecuzione istruzioni di salto condizionato
>Un'istruzione di salto condizionato viene eseguita in 4 passi:
>- Prelievo istruzione dalla memoria istruzioni e incremento del `pc`.
>- Lettura dei due registri sorgente dal banco registri, utilizzando i bit $19$-$15$ e $24$-$20$ per selezionarli.
>- Operazione dell'ALU per effettuare la sottrazione tra i valori letti dal banco dei registri: il valore del `pc` viene sommato all'immediato a $64\text{ bit}$, costruito opportunamente dai $12\text{ bit}$ di offset. Siccome l'offset a $12\text{ bit}$ indica una distanza di salto misurata in half-word, si esegue uno scorrimento a sinistra di una posizione per riportarlo al byte. Il risultato è l'indirizzo di destinazione del salto.
>- L'uscita Zero dell'ALU viene utilizzata per decidere quale valore debba essere memorizzato nel `pc`.

Per ogni tipo di istruzione i primi due passi da eseguire sono identici:
- Inviare il contenuto del `pc` ad una memoria che contiene il codice per prelevare l'istruzione.
- Leggere uno o due registri dal banco dei registri utilizzando i campi dell'istruzione per selezionare i registri ai quali accedere.

Dopo questi passi, le azioni necessarie per concludere l'esecuzione dell'istruzione dipendono dal tipo di istruzione.

Per quanto riguarda il tempo di esecuzione delle istruzioni si ha che:
- `ld` dura $800\text{ ps}$.
- `sd` dura $700\text{ ps}$.
- Le istruzioni in formato R durano $600\text{ ps}$.
- `beq` dura $500\text{ ps}$.

### Struttura del processore
>[!note]
>Di seguito la struttura base del processore RISC-V.
>
>![[Pasted image 20250623123340.png|center]]
>
>Si ha che la memoria istruzioni è separata dalla memoria dati. Poi i $32$ registri sono organizzati in register file con due porte di lettura e una porta in scrittura, con $3$ ingressi collegati ai campi dell'istruzione che specificano i registri sorgente e il registro destinazione, $1$ ingresso per i dati che possono essere scritti nel registro destinazione e $2$ uscite che riportano il contenuto dei $2$ registri letti.
>
>Il risultato dell'ALU è utilizzato per:
>- Scrivere in un registro destinazione.
>- Essere usato come indirizzo di lettura/scrittura della memoria dati.
>- Calcolare l'indirizzo della prossima istruzione.

Per temporizzare gli elementi di stato si utilizza un clock con fronte attivo di salita, e quindi le operazioni sono efficaci solo in corrispondenza del segnale di controllo dell'operazione asserito e del fronte attivo del segnale di clock.

Per convenzione non si riporta esplicitamente il segnale di controllo dell'operazione se questa avviene ad ogni fronte attivo del segnale di clock.

>[!tip] Immediati
>Le istruzioni dell'architettura di riferimento hanno tutte immediati a $12\text{ bit}$ che rappresentano un offset da sommare al contenuto di un registro per ottenere un indirizzo di memoria.
>
>Per `ld` e `sd` l'offset è un valore a byte, mentre per `beq` l'offset è un valore a mezza parola e per essere sommato al `pc` per calcolare l'indirizzo destinazione deve essere riportato a offset a byte.
>
>La costruzione dell'immediato a $64\text{ bit}$ viene realizzata da uno specifico modulo dell'architettura, che riceve in ingresso i $32\text{ bit}$ dell'istruzione. Tramite l'opcode distingue `ld`, `sd` e `beq`, e l'estensione viene effettuata in parallelo nei 3 formati. I due bit di codice operativo $5$ e $6$ possono controllare un muxer per selezionare l'estensione appropriata a seconda dell'istruzione in esecuzione.
>
>Nel caso di `ld` e `sd` l'estensione ottenuta è completa e può essere utilizzata, mentre nel caso di `beq`, bisogna riportare l'offset a byte, e quindi si esegue uno shift a sinistra di $1$.

>[!tip] Salti condizionati
>![[Pasted image 20250623125439.png|center]]
>
>La base per il calcolo dell'indirizzo di destinazione di un salto è il `pc`. L'operazione di salto condizionato è costituita da due operazioni:
>- Calcolo dell'indirizzo di destinazione del stato tramite un sommatore che effettua la somma tra il `pc` e il valore dell'offset esteso e fatto scorrere a sinistra di $1\text{ bit}$.
>- Confronto nell'ALU del contenuto dei registri operandi letti da `rf`. Se il segnale di uscita `zero` è asserito la condizione di salto è verificata e l'indirizzo di destinazione del salto diventa il nuovo `pc`.
>  
>Per poter caricare nel `pc` l'indirizzo corretto della prossima istruzione da eseguire è necessario aggiungere un multiplexer a due vie in ingresso al `pc` stesso.

Combiniamo questi elementi in un'unica unità di elaborazione. Assumiamo che tutte le istruzioni siano eseguite in un solo ciclo di clock:
- Nessuna risorsa può essere utilizzata più di una volta per istruzione.
- Qualsiasi risorsa di cui si ha bisogno più di una volta deve essere duplicata.

Alcune unità funzionali possono essere condivise da differenti flussi di istruzioni, per condividere un elemento tra due diversi tipi di istruzione si deve introdurre un muxer.

Si ha quindi che la struttura della CPU completa è la seguente.

![[Pasted image 20250623130107.png]]

Aggiungendo un unità di controllo per generare i segnali di controllo alla ALU in funzione agli ingressi.

![[Pasted image 20250623130245.png|center]]