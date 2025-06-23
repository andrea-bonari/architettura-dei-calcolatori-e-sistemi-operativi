>[!note]
>Il pipelining è un processo che aumenta il throughput dell'intero lavoro parallelizzando più istruzioni. Si basa sulla sovrapposizione dell'esecuzione di più istruzioni appartenenti a un flusso di esecuzione sequenziale.
>
>Di base il lavoro svolto in una CPU pipeline per eseguire un'istruzione è diviso in passi, che richiedono una frazione del tempo necessario al completamento dell'intera istruzione. Gli stadi sono sovrapposti per formare la pipeline.
>
>È vantaggiosa perché è completamente trasparente al programmatore.

Il tempo necessario per far avanzare un'istruzione di uno stadio lungo la pipeline corrisponde idealmente ad un ciclo di clock della pipeline. Siccome gli stadi sono tutti collegati in successione, la durata di un ciclo di clock è determinata dal tempo richiesto per lo stadio più lento della pipeline.

Se gli stadi sono perfettamente bilanciati, l'accelerazione ideale dovuta al pipelining è pari al numero di stadi di pipeline: $$\text{tempo con pipeline}= \frac{\text{tempo senza pipeline}}{\text{numero di stadi}}$$
In generale, gli stadi non sono perfettamente bilanciati e l'introduzione del pipelining comporta costi aggiuntivi.

>[!tip]
>Consideriamo che nell'architettura RISC-V le istruzioni possono essere suddivise al massimo in cinque passi. Ciascuno stadio di pipeline ha una durata prefissata che deve essere sufficientemente lunga da consentire l'esecuzione dell'operazione più lenta.
>
>Al crescere del numero di istruzioni il rapporto tra i tempi totali di esecuzione dei programmi su macchine senza e con pipeline è vicino al limite ideale.
>
>Formalizziamo i passi svolti durante l'esecuzione delle istruzioni in modalità pipeline:
>- IF (Instruction Fetch)
>- ID (Instruction Decode)
>- EX (Execution)
>- MEM (Memory Access)
>- WB (Write Back)
>  
>Quindi suddividiamo la nostra architettura in questi stadi.
>
>![[Pasted image 20250623132213.png|center]]

### Struttura pipeline della CPU RISC-V
>[!note]
>Suddividere l'esecuzione di un'istruzione in cinque passi implica che in ogni ciclo di clock siano eseguite cinque istruzioni, introduciamo quindi dei registri di pipeline che separano i diversi stadi.
>
>![[Pasted image 20250623132344.png|center]]

Le informazioni memorizzate nei registri interstadio sono relative ad istruzioni diverse. L'istruzione memorizzata nel registro di pipeline IF/ID fornisce il numero del registro di scrittura, mentre i dati scritti sono quelli relativi all'istruzione che si trova nel registro MEM/WB. Modifichiamo quindi la CPU in modo da trasmettere attraverso i registri di pipeline l'indirizzo del registro da scrivere durante lo stadio WB.

![[Pasted image 20250623132621.png|center]]

>[!tip] Implementazione dell'unità di controllo
>
>È necessario impostare correttamente il valore dei vari segnali di controllo per ciascuno stadio della pipeline per ciascuna istruzione, estendiamo quindi i registri di pipeline per salvare e propagare questi segnali.
>
>![[Pasted image 20250623132910.png|center]]

Ci possono essere dei conflitti di tipo strutturale, dati e di controllo.

### Conflitti
>[!note]
>Il processo di pipelining, per quanto efficiente può portare a dei conflitti:
>- Conflitti strutturali: tentativo di usare la stessa risorsa da parte di due o più istruzioni nello stesso intervallo di tempo.
>- Conflitto sui dati: tentativo di utilizzare un risultato prima che sia pronto.
>- Conflitto sul controllo: tentativo di prendere una decisione sulla prossima istruzione da eseguire prima che la condizione sia valutata.

Nell'architettura RISC-V non abbiamo conflitti strutturali, questo perché la memoria delle istruzioni è separata dalla memoria dati, e il banco di registri, per quanto possa essere usato nello stesso ciclo di clock in lettura e scrittura, usa in modo coerente la tecnica di temporizzazione.

> [!tip] Conflitto di dati  
> Se le istruzioni eseguite nella pipeline sono dipendenti tra loro, possono nascere problemi dovuti a conflitti di dati.
> 
> Per risolvere questi problemi si possono applicare tecniche software o hardware.
> 
> Nel caso delle tecniche software si possono inserire istruzioni `nop` (no operation), oppure si può applicare scheduling o riordino delle istruzioni in modo da impedire che istruzioni correlate siano troppo vicine.
> 
> Per quanto riguarda alle tecniche hardware si possono inserire "bolle" o stalli nella pipeline, o si possono propagare i dati in avanti (forwarding).
> 
> Inserire delle bolle significa bloccare il flusso di ingresso di istruzioni nella pipeline finché il conflitto non è risolto. Lo stallo di un'istruzione viene inserito nello stadio ID che rileva il conflitto e che quindi non può completare la scrittura del registro ID/EX. Il valore del `pc` viene congelato fino al termine del conflitto. In ciascun ciclo di stallo, lo stadio ID scriverà nella parte "segnali di controllo" del registro ID/EX solo valori pari a $0$, in questo modo nei cicli di clock seguenti, gli stadi successivi a ID di fatto non lavorano, poiché i segnali di controllo a $0$ si propagano finché non termina l'effetto dello stallo.
> 
> Un'altra possibile soluzione è il forwarding, che consiste nella propagazione in avanti dei dati. Questo è possibile perché in certe operazioni (tipicamente R-type) il risultato è disponibile alla fine dello stadio EX. Per farlo si aggiungono multiplexer agli ingressi dell'ALU, così da prelevare i dati da qualsiasi registro di pipeline (EX/MEM o MEM/WB) e non solo dal registro ID/EX.
> 
> La logica di rilevamento per attivare il forwarding o uno stallo è implementata confrontando i registri `Rd`, `Rs1`, `Rs2` nei registri interstadio, e verificando il flag `RegWrite`:
> 
> |Condizione|Tipo di Forwarding|Azione da intraprendere|
> |---|---|---|
> |EX/MEM.RegWrite = 1 **e** EX/MEM.Rd ≠ 0|EX → EX (ForwardA)|ForwardA = 10 se EX/MEM.Rd = ID/EX.Rs1|
> |EX/MEM.RegWrite = 1 **e** EX/MEM.Rd ≠ 0|EX → EX (ForwardB)|ForwardB = 10 se EX/MEM.Rd = ID/EX.Rs2|
> |MEM/WB.RegWrite = 1 **e** MEM/WB.Rd ≠ 0|MEM → EX (ForwardA)|ForwardA = 01 se MEM/WB.Rd = ID/EX.Rs1|
> |MEM/WB.RegWrite = 1 **e** MEM/WB.Rd ≠ 0|MEM → EX (ForwardB)|ForwardB = 01 se MEM/WB.Rd = ID/EX.Rs2|
> |ID/EX.MemRead = 1 **e** (ID/EX.Rd = IF/ID.Rs1 **o** IF/ID.Rs2)|Load → Use (Hazard)|Inserisci uno **stallo** di 1 ciclo|
> 
> Nel caso di hazard Load → Use, il dato letto non è ancora disponibile allo stadio EX e nemmeno il forwarding basta. In questo caso si rileva la situazione durante lo stadio ID:
> 
> ```c
> if (ID/EX.MemRead && ((ID/EX.Rd == IF/ID.Rs1) || (ID/EX.Rd == IF/ID.Rs2))) {
>    Stall = 1;
>    PCWrite = 0;
>    IF/IDWrite = 0;
>    ControlSignals = 0; // inserimento NOP
> }
> ```
> 
> Questo stallo introduce una bolla nella pipeline: l'istruzione in ID resta congelata, e le successive vengono temporaneamente bloccate, finché il dato non è disponibile.
> 
>![[Pasted image 20250623205325.png|center]]
> 

È necessaria un'unità di rilevamento dei conflitti che, durante lo stadio ID di "use" possa inserire uno stallo tra la lettura del dato e il suo indirizzo

>[!tip] Conflitto di controllo
>Per alimentare la pipeline occorre prelevare un'istruzione a ogni ciclo di clock, ma la decisione relativa al salto condizionato non viene presa fino allo stadio MEM.
>
>Questo ritardo nel determinare l'istruzione corretta da prelevare viene chiamata conflitto di controllo.
>
>Come soluzione si può:
>
>Utilizzare la pipeline standard e non utilizzare la predizione: è possibile via software inserire tre `nop` dopo ogni istruzione di salto condizionato, oppure via hardware bloccare la scrittura del `pc` e di IF/ID per tre cicli dopo un'istruzione di salto condizionato.
>
>Utilizzando la predizione, assumiamo che il salto venga non eseguito. Quando si verifica che il salto deve essere eseguito si scartano le tre istruzioni già nella pipeline.
>
>Si può anche utilizzare una pipeline ottimizzata che, sposta la decisione del salto dallo stadio MEM allo stadio ID e richiede di anticipare il calcolo dell'indirizzo e la valutazione del confronto, in questo modo si può avere al più un'istruzione da scartare.
>Per anticipare il calcolo di destinazione `pc` e campo immediato sono disponibili nel registro IF/ID della pipeline una volta letta l'istruzione di salto. Per farlo basta spostare il sommatore che calcola l'indirizzo del salto dallo stadio EX allo stadio ID.
>Per anticipare la decisione sul salto è necessario confrontare il contenuto dei due registri letti nello stadio ID, si implementa quindi uno XOR bit a bit dei due valori: se il risultato è $0$ allora i due valori sono uguali, e quindi basta mettere un NOR sulle uscite, se è uguale a $1$ allora la condizione è verificata.
>
>Per eliminare l'istruzione in esecuzione nello stadio IF viene aggiunto un segnale di controllo `IF.Flush` che azzera la parte del registro di pipeline IF/ID.
>
>In caso di branch taken per poter eseguire il fetch dell'istruzione di destinazione di salto e scartare l'istruzione presente nello stadio IF è necessario generare sempre nello stadio ID.

Il nostro modello di processore completo dopo queste considerazioni è il seguente.

![[Pasted image 20250623211155.png|center]]
