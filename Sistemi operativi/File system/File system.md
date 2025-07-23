>[!note]
>Il File System è quel componente del SO che realizza i servizi di gestione dei file. Rappresenta la struttura logica della memoria di massa ed è costituito da un insieme di funzioni di SO e relative strutture dati di support. Linux è in grado di gestire file system molto diversi (standard, di rete, etc).
>
>Definiamo un virtual file system come un interfaccia unica per gestire i diversi file system che mette a disposizione le API che consentono utilizzo indipendente delle chiamate di sistema.

Le astrazioni fornite dal file system sono:
- File, link e operazioni sui file
- Direttori e file speciali
- Attribuiti e politiche di accesso
- Mapping del file system logico sui dispositivi fisici

>[!tip] Volume
>I dispositivi di memorizzazione non volatile possono essere suddivisi in porzioni dette partizioni. Ogni partizione può essere funzionalmente considerata come un dispositivo logico indipendente.
>
>L'indirizzamento dei dati si basa sul concetto di Logical Block Address (LBA), uno schema di indirizzamento nel quale l'intero dispositivo è rappresentato come un vettore lineare di blocchi, ogniuno costituito da un certo numero byte.
>
>Il termine volume indica una partizione di qualsiasi dispositivo di memorizzazione non volatile di massa dotato di uno schema di indirizzamento LBA.
>
>Il blocco costituisce l'unità fondamentale di informazione che viene trasferita con una sola operazione tra il disco e la memoria centrale.

>[!tip] File
>Un file è un insieme costituito da una sequenza di byte, un nome simbolico e univoco ed un insieme di attributi. Un file in Linux può contenere dati o programmi, contenere riferimenti ad altri file, oppure rappresentare l'astrazione di un dispositivo periferico.
>
>Tutte le operazioni sul contenuto del file richiedono il trasferimento in/da memoria di byte e partono dalla posizione corrente nel file, aggiornandola. Tra le operazioni principali troviamo creazione, apertura, chiusura, cancellazione, lettura, scrittura e riposizionamento.
>
>Potremmo suddividere il modello d'utente in 2 aspetti:
>- Accesso al singolo file: è costituito dalle funzioni utilizzate per scrivere e leggere informazioni sui singoli file.
>- Organizzazione della struttura complessiva dei file: è costituito dalle funzioni utilizzate per organizzare i file nei direttori e volumi.
>
>L'accesso a un file può avvenire tramite mappatura di una VMA sul file, oppure tramite system call. Il VFS deve implementare ambedue le modalità
>
>L'immagine del file system è memorizzata in memoria di massa. Per rendere più efficienti gli accessi a file, il SO mantiene in memoria centrale una parte delle strutture dati per gestire i file, tra cui la tabella dei file aperti per ogni processo, e la tabella globale dei file aperti nel sistema.
>
>Il descrittore di file è un numero intero non negativo che rappresenta un file aperto da un processo e sul quale il processo può effettuare operazioni di I/O tramite system call.

I file sono organizzati in una struttura ad alberi i cui nodi sono detti cataloghi. Per semplificare la gestione dei file, i nomi dei file sono inseriti nei cataloghi.

Un catalogo non è altro che un file dedicato a contenere i nomi di altri file. I file dedicati a servire come catalogo sono detti di tipo catalogo, mentre i file che contengono normali informazioni sono detti di tipo normale. La struttura dei cataloghi si basa sull'esistenza di un unico catalogo principale detto radice, che il sistema è in grado di identificare ed accedere autonomamente.

Il nome completo di un file di qualsiasi tipo è costituito dal concatenamento dei nomi di tutti i cataloghi sul percorso che porta dalla radice al file stesso, separati dal simbolo "/".

Di seguito la struttura del File System Linux:

![[Pasted image 20250723180018.png]]

