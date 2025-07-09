>[!note]
>La realizzazione di un SO multiprogrammato come Linux o Windows richiede da parte dell'hardware la disponibilità di alcuni meccanismi fondamentali. Usiamo come riferimento specifico `x64`. Alcune sue funzionalità sono inutilmente complesse per motivi di compatibilità, ne sarà fornite una versione semplificata.

Nell'`x64` la pila cresce da indirizzi alti verso indirizzi bassi. Il decremento e l'incremento dello SP sono svolti nella stessa iscrizione di scrittura e lettura in memoria e quindi `push` e `pop` richiedono una sola istruzione ciascuno.

Inoltre, in `x64` si salva il valore dell'indirizzo di ritorno sulla pila, non in un registro.

### Istruzioni privilegiate
>[!note]
>Il processore ha la possibilità di funzionare in due stati (o modi) diversi: modo utente e modo supervisore.
>
>Il modo S può eseguire tutte le proprie istruzioni e può accedere a tutta la propria memoria, mentre il modo U può eseguire solo una parte delle proprie istruzioni e può accedere solo a una parte della propria memoria.
>
>Le istruzioni eseguibili solo quando il processore è in modo S sono dette istruzioni privilegiate.
>
>Nell'`x64` esistono 4 modi, ma Linux ne usa solo i due estremi (CPL3 per modo U, e CPL0 per modo S).

Il modo di funzionamento è rappresentato da un bit del `psr`.

>[!tip] Istruzione `syscall`
>Esiste un'istruzione `syscall` non privilegiata, che realizza un salto al SO. Essa opera nel seguente modo:
>- Il valore del `pc` incrementato è salvato sulla pila.
>- Il valore del `psr` viene salvato sulla pila.
>- Nel `pc` e `psr` vengono caricati i valori presenti in una struttura dati a un indirizzo noto all'hardware detta vettore di Syscall.
>
>Linux inizializza il vettore di syscall durante la fase di avviamento del sistema, con la coppia:
>- Indirizzo della funzione `system_call`.
>- `psr` opportuno per l'esecuzione di `system_call`, modo S.
>
>Quindi `system_call` costituisce il punto di entrata unico per tutti i servizi di sistema di Linux.
>
>Esiste anche un operazione `sysret` privilegiata, che carica gli elementi salvati nella pila nel `pc` e `psr`.
>
>In Linux è eseguita alla fine della funzione `system_call`.

>[!tip] Commutazione di pila
>La pila utilizzata implicitamente dalla CPU nello svolgimento delle istruzioni è puntata dal registro `sp`. Per realizzare il SO è necessario fare in modo che la pila utilizzata durante il funzionamento in modo S sia diversa da quella utilizzata durante il funzionamento in modo U.
>
>Quindi la CPU utilizza una pila diversa quando opera in modi diversi (sPila e uPila). Linux alloca ad ogni processo una sPila costituita da $2$ pagine.
>
>Nella commutazione da modo U a modo S la commutazione di pila avviene prima del salvataggio di informazioni sulla stessa. L'indirizzo di ritorno a modo U deve essere salvato su sPila, e nel ritorno da modo S a modo U l'informazione per il ritorno verrà prelevata da sPila, cioè prima di commutare.
>
>Per poter commutare è necessaria un'opportuna struttura dati basata su 2 celle di memoria `usp` e `ssp`.
>
>La `ssp` contiene il valore da caricare in `sp` al modo S, mentre in `usp` viene salvato il valore del registro `sp` al momento del passaggio a modo S.
### Modello di memoria
>[!note]
>Quando il processore è in modo U non deve poter accedere alle zone di memoria riservate al SO. Viceversa, quando il processore è in modo S deve poter accedere sia alla memoria del SO, sia alla memoria dei processi.

In `x64` lo spazio di indirizzamento potenziale è $2^{64}\text{ byte}$, tuttavia l'architettura limita lo spazio virtuale utilizzabile a  $2^{48}\text{ byte}$. Tale spazio è suddiviso nei $2$ sottospazi di modo U e S, ambedue da $2^{47}\text{ byte}$.

La CPU in modo S può utilizzare tutti gli indirizzi canonici, mentre in modo U un indirizzo superiore a $\mathtt{0x}0000\space7FFF\space FFFF\space FFFF$ genera un errore.

>[!tip] Paginazione
>La memoria è suddivisa in unità dette pagine, di dimensione $4\text{ kB}$. Queste costituiscono l'unità di allocazione della memoria.
>
>Ogni indirizzo prodotto dalla CPU viene trasformato in un indirizzo fisico prima di accedere alla memoria fisica. Questa mappatura è descritta dalla tabella delle pagine.

Linux associa ad ogni processo una diversa tabella delle pagine. Nell'`x64` esiste un registro (`cr3`) che contiene l'indirizzo di inizio della tabella delle pagine, utilizzata per la mappatura degli indirizzi.
### Meccanismo di interruzione
>[!note]
>Esiste un insieme di eventi rilevati dall'hardware, a cui è associata una particolare funzione detta gestore dell'interrupt o routine di interrupt, che fanno parte del SO. Quando il processore rileva un evento, esso interrompe il programma correntemente in esecuzione ed esegue un salto all'esecuzione della funzione associata a tale evento (in modo S). Quando la routine di interrupt termina, il processore riprende l'esecuzione del programma che è stato interrotto.
>
>Per poter riprendere l'esecuzione il processore ha salvato sulla pila, al momento dell'interrupt, l'indirizzo della prossima istruzione del programma interrotto, e quindi dopo l'esecuzione della routine di interrupt tale indirizzo è disponibile per eseguire il ritorno.
>
>L'istruzione privilegiata che esegue questo ritorno è `iret`. Le routine di interrupt sono completamente asincrone rispetto al programma in esecuzione.

Il meccanismo di interrupt si combina con il doppio modo di funzionamento S ed U in maniera simile a quello della `syscall`. Dal punto di vista hardware non c'è differenza tra le due.

Se il modo del processore al momento dell'interrupt era già S, alcune operazioni non sono necessarie, ma il registro di stato è comunque salvato sulla sPila.

L'istruzione `iret` riporta la macchina al modo di funzionamento in cui era prima che l'interrupt si verificasse, prelevando il `psr` dalla sPila.

Il processore deve sapere quale sia l'indirizzo della routine di interrupt e il valore di `psr` da utilizzare, utilizza quindi la tabella degli interrupt, una struttura dati con accesso hardware che contiene un certo numero di vettori di interrupt costituiti da una coppia $(\text{PC},\text{PSR})$.

Esiste poi un meccanismo hardware che è in grado di convertire l'identificativo dell'interrupt nell'indirizzo del corrispondente vettore di interrupt.

L'inizializzazione della tabella degli interrupt con gli indirizzi è svolta dal SO in fase di avviamento.

>[!tip] Gestione degli errori
>Durante l'esecuzione delle istruzioni possono verificarsi degli errori che impediscono al processore di proseguire.
>
>Gran parte dei processori prevede di trattare l'errore come se fosse un particolare tipo di interrupt, quando si verifica un errore che impedisce al processore di procedere normalmente con l'esecuzione delle istruzioni, viene attivata, con un opportuno vettore di interrupt, una routine del SO che decide come gestire l'errore stesso.
>
>Spesso la gestione consiste nella terminazione forzata (`abort`) del programma che ha causato l'errore, eliminando il processo.

>[!tip] Priorità dell'interrupt
>Normalmente gli interrupt possono essere annidati, però non è sempre opportuno permettere a un interrupt di interrompere la routine che serve un altro interrupt. È quindi necessario prevedere che un evento molto importante e che richiede una risposta urgente possa interrompere la routine di interrupt che server ad un evento meno importante, ma non il contrario.
>
>Il processore possiede quindi un livello di priorità definito nel registro `psr`, che può essere modificato opportunamente tramite specifiche istruzioni. Ad ogni interrupt è associato un livello di priorità, e questo viene accettato se e solo se il suo livello di priorità è superiore al livello di priorità del processore in quel momento.
>
>In caso contrario l'interrupt è tenuto in sospeso fino al momento in cui la priorità del processore non sarà stato abbassato sufficientemente.
### Interfacce standard e ABI
>[!note]
>Le regole che governano il modo in cui un compilatore conforme a gnu deve tradurre i sorgenti sono definiti dalla Application Binary Interface (ABI). Queste regole servono per garantire che tutti i moduli siano tradotti in modo coerente con le convenzioni adottate per il passaggio dei parametri.

In `x64` per passare parametri alla funzione `system_call` è necessario passare il numero di servizio da invocare nel registro `rax`, e passare eventuali parametri ordinatamente nei registri `rdi`, `rsi`, `rdx`, `r10`, `r8`, `r9`.

Un programma applicativo non invoca la `syscall` direttamente, ma invoca una funzione della librarie `glibc` che a sua volta contiene la chiamata di sistema.

In questa libreria sono presenti funzioni che corrispondono ai servizi offerti dal SO.

Di seguito tutti i valori delle `syscall`.

| `%rax` | System call    | `%rdi`                 | `%rsi`                 | `%rdx`                | `%r10`                |
| ------ | -------------- | ---------------------- | ---------------------- | --------------------- | --------------------- |
| `0`    | `sys_read`     | `unsigned int fd`      | `char *buf`            | `size_t count`        |                       |
| `1`    | `sys_write`    | `unsigned int fd`      | `const char *buf`      | `size_t count`        |                       |
| `2`    | `sys_open`     | `const char *filename` | `int flags`            | `int mode`            |                       |
| `3`    | `sys_close`    | `unsigned int fd`      |                        |                       |                       |
| `4`    | `sys_stat`     | `const char *filename` | `struct stat *statbuf` |                       |                       |
| `5`    | `sys_fstat`    | `unsigned int fd`      | `struct stat *statbuf` |                       |                       |
| `6`    | `sys_lstat`    | `const char *filename` | `struct stat *statbuf` |                       |                       |
| `7`    | `sys_poll`     | `struct poll_fd *ufds` | `unsigned int nfds`    | `long timeout_msecs`  |                       |
| `8`    | `sys_lseek`    | `unsigned int fd`      | `off_t offset`         | `unsigned int origin` |                       |
| `9`    | `sys_mmap`     | `unsigned long addr`   | `unsigned long len`    | `unsigned long prot`  | `unsigned long flags` |
| `10`   | `sys_mprotect` | `unsigned long start`  | `size_t len`           | `unsigned long prot`  |                       |
