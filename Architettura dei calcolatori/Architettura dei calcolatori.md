>[!note]
>L'architettura di un calcolatore si basa su svariati livelli di astrazione. Il corso si occuperà di coprire la microarchitettura, l'architettura e il sistema di runtime.
>
>![[Pasted image 20250619190254.png|center]]

Un programma stesso può essere astratto come linguaggio assembly o linguaggio macchina (MIPS).

### Architettura Von Neumann
>[!note]
>Il modello di riferimento tuttora usato è il modello Von Neumann, e si basa sulla presenza di:
>- Un processore (CPU)
>- Una memoria di lavoro o memoria principale (MM)
>- Un bus di sistema (dati, indirizzi e controllo)
>- Interfacce per i dispositivi di Input/Output (I/O)
>
>![[Pasted image 20240917093724.png|center]]

>[!tip] Unità di controllo (CU)
>Legge le istruzioni dalla memoria e ne determina il tipo.

>[!tip] Unità Aritmetico Logica (ALU)
>Esegue le operazioni necessarie per eseguire le istruzioni.

>[!tip] Registri
>Sono delle memorie ad alte velocità usate per risultati temporanei e informazioni di controllo. Il valore massimo memorizzabile in un registro è determinato dalle dimensioni del registro. Esistono dei registri di uso genero e registri di uso specifico.

Le istruzioni eseguite dalla CPU si distinguono in:
- Registro-Memoria: gestiscono il trasferimento dati tra i registri della CPU e la memoria centrale
- Registro-registro: Utilizzano il contenuto dei registri per svolgere operazioni e memorizzano il risultato in un registro (ciclo del data path)

L'unità di informazione trasferita prende il nome di parola (word).

Il ciclo del data path è al centro del funzionamento di quasi tutte le CPU. Quanto è più piccolo il tempo di ciclo e più è veloce è il calcolatore.

### Ciclo di esecuzione istruzioni
>[!note]
>Per eseguire le istruzioni il calcolatore ripete il ciclo Fetch-Decode-Execute:
>
>1. Prendi l'istruzione corrente e inseriscila nel Registro Istruzioni (`IR`).
>2. Incrementa il Program Counter (`PC`) in modo che contenga l'indirizzo dell'istruzione successiva.
>3. Determina il tipo di istruzione corrente.
>4. Se l'istruzione usa una parola in memoria, determina dove si trova.
>5. Carica la parola, se necessario, in un registro della CPU.
>6. Esegui l'istruzione.
>
>

Dati e istruzioni vengono manipolati dal calcolatore dopo essere state opportunamente codificate.

Un istruzione è suddivisa in campi (field):
- Campo codice operativo, che indica il tipo di operazione.
- Gli altri campi indicano gli operandi e/o gli operatori stessi.

Le modalità di indirizzamento indicano le diverse modalità attraverso le quali far riferimento agli operandi delle istruzioni. 

Generalmente il numero dei registri ad uso generale non è sufficientemente grande da consentire di memorizzare tutte le variabili di un programma, quindi ad ogni variabile si assegna un indirizzo in memoria. La modalità più usata è l'architettura LOAD/STORE, dove gli operandi dell'ALU possono provenire soltanto dai registri, e di conseguenza sono necessarie apposite istruzioni di caricamento e memorizzazione.

>[!tip] Interruzioni
>Oltre a trasferire dati/istruzioni tra memoria e processore, il calcolatore acquisisce dati dai dispositivi di ingresso e invia dati ai dispositivi di uscita attraverso apposite istruzioni che gestiscono I/O.
>
>Un flusso di esecuzione di un programma può essere interrotto da un segnale di interruzione (`INTERRUPT`) per una richiesta di intervento che un dispositivo I/O manda al processore.
>
>Il processore fornisce il servizio richiesto mediante l'esecuzione di una procedura di servizio delle interruzioni (Interrupt Service Routine) che deve salvare in memoria lo stato del processore prima di servire l'interruzione: il contenuto del `PC`, dei registri ad uso generale e alcune informazioni di controllo vengono salvati in memoria.
>
>Quando la procedura di servizio dell'interruzione viene completata, lo stato del processore viene ripristinato in modo che il programma interrotto possa proseguire.

### Architettura RISC
>[!note]
>RISC (Reduced Instruction Set Computer) è un architettura di istruzioni di processori con istruzioni molto semplici, frequentemente impiegate e implementabili in modo efficiente in hardware.

>[!quote] Cenni storici
>I primi computer avevano ISA molto semplice. Negli anni '70, l'architettura dominante era quella dei cosiddetti processori microprogrammati (Complex Instruction Set Computer o CISC). L'idea dell'architettura CISC era di fornire istruzioni molto complesse che rispecchiassero i costrutti dei linguaggi ad alto livello.
>
>Tuttavia dopo delle misurazioni di performance effettuate a metà degli anni '70 si dimostrò che la maggior parte delle applicazioni utilizzavano solo poche semplici istruzioni. Di conseguenza negli anni '80 si ebbe l'avvento dell'architettura Reduced Instruction Set Computer (RISC).

Il progetto di un processore RISC è più semplice e ottimizzabile, tuttavia dato il ridotto set di istruzioni il compito del compilatore diventa più complesso. Di conseguenza:
- Il progetto del processore diventa più semplice e ottimizzabile
- È necessario ottimizzare la selezione delle istruzioni RISC da impiegare per la traduzione di un compilatore

### Memoria
>[!note]
>La memoria è un componente il cui compito è contenere il sistema operativo ed i processi in esecuzione (completi di istruzioni e dati). 
>
>Una caratteristica fondamentale è la dimensione complessiva. 
>È inoltre caratterizzata dalla dimensione di ogni parola trasferita. Ad ogni parola in memoria è associato un indirizzo composto da $k$-$\text{bit}$. I $2^{k} \text{ bit}$  costituiscono lo spazio di indirizzamento del calcolatore.

>[!tip] Gerarchia della memoria
>La gerarchia della memoria è una struttura secondo il quale il processore prova a recuperare i dati dalla memoria più vicina ad esso, e se non li trova prova a recuperarli sulla memoria successiva, fino a raggiungere tutte le memorie disponibili. Generalmente si parte dalle memorie più veloci e piccole e si raggiunge le memorie più lente e grandi.
>
>![[Pasted image 20240917102401.png|center]]

Solitamente la dimensione della parola in memoria coincide con la dimensione dei registri contenuti della CPU. In questo modo se anche il bus dati è largo come la parola di memoria, l'operazione di `load`/`store` avviene in un singolo ciclo.

Le memorie in cui ogni locazione può essere raggiunta in un breve e prefissato intervallo di tempo vengono chiamate memorie ad accesso casuale (RAM). Nelle RAM il tempo di accesso alla memoria è fisso e indipendente dalla posizione della parola a cui si vuole accedere.

### Bus di sistema
>[!note]
>Il bus di sistema è il componente che permette la comunicazione tra le diverse unità del calcolatore, ed è generalmente composto da tre parti:
>- Bus dati: trasferisce dati e istruzioni da/verso la memoria. La sua dimensione garantisce il trasferimento contemporaneo di una o più parole di memoria.
>- Bus indirizzi: la CPU trasmette l'indirizzo di memoria da cui prelevare il dato nel caso di lettura data dalla memoria, oppure l'indirizzo in cui depositare un dato in caso di scrittura.
>- Bus di controllo: transitano le informazioni ausiliarie per la corretta definizione delle operazioni da compiere e per la sincronizzazione tra CPU e memoria.

I principali vantaggi della struttura a bus singolo sono l'elevata flessibilità e i bassi costi. Tuttavia i dispositivi collegati al bus variano in termini di velocità dell'esecuzione delle operazioni, ed è quindi necessario un meccanismo di sincronizzazione per garantire il trasferimento efficiente delle informazioni sul bus. Tipicamente dentro le unità che usano il bus sono presenti dei buffer per non vincolarsi alla velocità del dispositivo più lento connesso al bus.