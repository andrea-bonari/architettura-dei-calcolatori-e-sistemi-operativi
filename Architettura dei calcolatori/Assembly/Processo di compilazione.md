>[!note]
>Generalmente il flusso di compilazione e assemblaggio è il seguente:
>
>![[Pasted image 20250621124211.png|center]]
>

### Processo di assemblaggio
>[!note]
>Il processo di assemblaggio si applica al programma modulo per modulo, dove ciascuno modulo sorgente costruisce il corrispondente modulo oggetto. 
>
>Esamina riga per riga il codice sorgente assembly di un modulo e ne traduce le istruzioni simboliche nel formato di linguaggio macchina:
>- I codici mnemonici sono tradotti nei corrispondenti codici binari
>- I riferimenti ai registri sono tradotti nei corrispondenti numeri di registro
>- I riferimenti simbolici del modulo sono tradotti, se possibile, negli indirizzi binari corrispondenti usando una tabella dei simboli.
>  
>Il segmento di ciascun modulo è assemblato in termini di indirizzi virtuali rilocabili.

Durante il processo di assemblaggio:
1. Si traducono le pseudoistruzioni.
2. Si genera la tabella dei simboli (possono rimanere riferimenti non risolti definiti in un altro file).
3. Si genera un file oggetto.

>[!tip]
>Nel primo passo l'assemblatore non traduce nessuna istruzione ma costruisce la tabella dei simboli del modulo. I riferimenti simbolici associati alla tabella possono essere:
>- Simboli associati a direttive dell'assemblatore (`.eqv`).
>- Etichette che definiscono variabili del segmento dati.
>- Etichette che contrassegnano istruzioni destinazioni di salto.
>
>I valori degli indirizzi inseriti sono quelli rilocabili rispetto al segmento considerato.
>
>Una volta generata la tabella dei simboli del modulo, si genera la tabella di rilocazione.
>
>Un'istruzione è tradotta in modo incompleto, e quindi va elaborata anche da parte del modulo se:
>- Il riferimento simbolico presente in essa è relativo a variabili del segmento `.data`.
>- Il riferimento è relativo a simboli non (ancora) presenti nella tabella dei simboli del modulo: il simbolo è posto a $0$ per convenzione e andrà poi calcolato.
>- Il riferimento simbolico presente in essa è relativo a simboli su cui agisce un modificatore.
>
>In corrispondenza di ogni traduzione incompleta viene creato un elemento della tabella di rilocazione, nella forma: $$\langle\text{ indirizzo rilocabile istruzione},\text{ opcode istruzione},\text{ simbolo da risolvere con modificatore}\rangle$$

### Formato oggetto
>[!note]
>Un oggetto è composto da:
>- Intestazione: descrive le dimensioni del testo e dei dati del modulo.
>- Segmento testo: contiene il codice in linguaggio macchina delle procedure e funzioni del file sorgente.
>- Segmento dati: contiene una rappresentazione binaria dei dati definiti nel file sorgente.
>- Informazioni di rilocazione: identificano istruzioni e le parole di dati che dipendono da indirizzi assoluti.
>- Tabella dei simboli: associa un indirizzo alle etichette esterne contenute nel file sorgente e contiene l'elenco dei riferimenti non risolti
>- Informazioni di debug

### Collegatore (linker)
>[!note]
>Il linker è il componente che ha il compito di collegare diversi moduli oggetto in un unico file eseguibile in formato binario.
>
>Questo tuttavia ci da la limitazione di avere un solo spazio di indirizzamento per tutto il programma.
>
>Il collegatore deve:
>- Determinare la posizione in memoria, cioè l'indirizzo iniziale delle sezioni codice e dati dei diversi moduli.
>- Determinare il nuovo valore di tutti gli indirizzi simbolici che risultano modificati dallo spostamento della base.
>- Correggere in tutti i moduli i riferimenti a indirizzi simbolici che sono stati modificati.

>[!tip] Riguardo `la`
>Siccome il segmento dati inizia dall'indirizzo: $$\mathtt{0x}0000\space0000\space1000\space0000$$
>Le istruzioni di accesso alla memoria non possono fare riferimento direttamente agli oggetti in esso contenuti, poiché esse hanno immediati da $12\text{ bit}$.
>
>La suddivisione dell'indirizzo nelle sue componenti per accedere a un dato in memoria, è ottenuta in modo automatico tramite assemblatore e collegatore facendo uso della pseudoistruzione `la`. La traduzione di `la` è come segue:
>```
>auipc rd, %pcrel_hi (VAR)
>addi rd, rd, %pcrel_lo (VAR)
>```
>E il calcolo svolto dal collegatore è il seguente:
>```
>delta = indirizzo effettivo di VAR - PC di auipc
>%hi (delta) e %lo (delta)
>```

In fase di collegamento gli indirizzi definiti all'interno in un modulo possono cambiare se la base dello spazio di indirizzamento virtuale del modulo viene modificata. Pertanto, tutte le etichette che corrispondono a indirizzi assoluti all'interno del modulo costituiscono simboli il cui valore può cambiare al momento del collegamento.

Siccome l'assemblatore alloca la sezione testo e la sezione dati a partire dall'indirizzo base $0$, il collegatore li carica sequenzialmente.

Inoltre si crea la tabella dei simboli globale, che è costituita dall'unione delle tabelle dei simboli di tutti i moduli che vanno collegati. I simboli vengono modificati secondo l'indirizzo di base del modulo al quale appartengono.

### Caricamento ed esecuzione
>[!note]
>Nei sistemi UNIX/LINUX, il kernel del sistema operativo ha il compito di caricare il programma nella memoria principale, predisponendone l'ambiente di lavoro, poi di lanciarne l'esecuzione e infine terminarlo.

In ordine il kernel svolte le seguenti operazioni:
1. Legge l'intestazione del file eseguibile del programma e determina le dimensioni dei segmenti di testo e dati.
2. Crea un nuovo spazio di indirizzamento per il programma, dimensionato per contenere i segmenti di testo e dati, nonché un segmento per la pila.
3. Copia il testo e i dati dal file eseguibile in memoria, dentro il nuovo spazio di indirizzamento.
4. Se la procedura `main` del programma fa uso di argomenti, il nucleo li prepara nell'area reserved prevista nel modello di memoria.
5. Inizializza i registri dell'architettura.
6. Effettua una procedura di avvio che copia gli argomenti del programma dall'area reserved ai registri, e poi chiama la procedura `main` del programma.
7. Quando la procedura `main` termina, la procedura di avvio riprende e conclude il programma tramite la chiamata di sistema `exit`.