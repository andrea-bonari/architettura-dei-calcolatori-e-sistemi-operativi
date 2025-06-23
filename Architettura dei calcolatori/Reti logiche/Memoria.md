>[!note]
>La memoria è un blocco funzionale sequenziale complesso. Mantiene a tempo indefinito le informazioni se alimentata, permettendo l'accesso in lettura o scrittura. Ha una struttura a vettore i cui elementi sono le parole di memoria di una certa lunghezza.
>
>Un componente integrato di memoria di memoria si caratterizza specificando:
>- La capacità
>- Le funzioni
>- Il numero di porte di accesso
>- Il tempo necessario per l'accesso
>  
>Il contenuto della memoria viene letto o scritto una parola per volta, in un ciclo di clock. Si accede a una parola di memoria tramite la porta di accesso alla memoria.
>
>La porta di accesso alla memoria può funzionare in lettura e scrittura o in sola lettura.
>
>![[Pasted image 20250622150914.png|center]]
>
>I segnali dell'interfaccia di memoria sono:
>- Ingressi di indirizzo: codificano in binario l'indirizzo della parola su cui operare.
>- Uscite/Ingressi di dato: servono a leggere/scrivere una parola.
>- $\text{Write enable}$: comando di scrittura, se è un impulso, deve avere una durata minima che consenta di soddisfare i tempi di setup e hold.
>- $\text{Output enable}$: comando di abilitazione delle uscite dati, se è attiva lo sono anche le uscite, viceversa sono isolate.
>- $\text{Chip select}$: comando di abilitazione del componente, se è attivo si può accedere al contenuto, viceversa non si può accedere.
>
>Per le linee di dato e indirizzo sono da rispettare i tempi di setup e hold.

La memoria è organizzata a matrice di bistabili.

![[Pasted image 20250622151453.png|center]]

### DRAM
>[!note]
>Nella RAM dinamica il singolo bit di informazione è memorizzato nella carica di un condensatore, il cui accesso avviene tramite un transistor che può leggere o scrivere il suo valore.
>
>La DRAM usa un singolo transistor per cella contro i $4$/$6$ transistor della cella SRAM, quindi le DRAM hanno un costo inferiore per bit e capacità maggiore, ma tempo di accesso minore.
>
>Ha una capacità grande-grandissima, tempo di accesso medio, funziona in lettura e scrittura ed è volatile.
>
>È usata principalmente come memoria centrale dei calcolatori.

È necessario un rinfresco delle parole di memoria per la degradazione della carica. Queste operazioni consumano meno del $2\%$ dei cicli di memoria.

### SRAM
>[!note]
>A differenza della DRAM, la SRAM è realizzata con dei bistabili. Ha una capacità medio-piccola, un tempo di accesso molto breve, funziona in lettura e scrittura ed è volatile.
>
>È usata principalmente come memoria cache di primo livello.

### ROM
>[!note]
>La ROM è una memoria realizzata come matrice di transistor. Ha una capacità grande, tempo di accesso medio, funziona in sola lettura ed è persistente.

### PROM, EPROM e EEPROM
>[!note]
>Ha capacità e tempo simili alla ROM, ed è di sola lettura e persistenti. Sono programmabili sul campo tramite un apposito dispositivo programmatore.
>
>La PROM è programmabile solo una volta, la EPROM è cancellabile con raggi UV, e la EEPROM è cancellabile elettricamente.
>
>Sono usate per piccoli volumi di produzione e prototipi.

### FLASH
>[!bite]
>La memoria flash ha capacità e tempo simili alla DRAM, funziona in lettura e scrittura ed è persistente.
>
>È usata principalmente per dati multimediali, programmi fissi ma periodicamente aggiornabili, e in generale come memoria.

### NVRAM
>[!note]
>La NVRAM è una nuova tecnologia di RAM con contenuto persistente. Ha capacità molto grande e tempo di accesso simile alla DRAM, e funziona in lettura e scrittura.
>
>Per quanto sia persistente è necessario un rinfresco, anche se molto poco frequente.
>
>Avrebbe uso sostitutivo alla DRAM come memoria centrale del calcolatore, ma con il vantaggio di essere persistente.
>
>Ci sono varie tecnologie in competizione: memoria resistiva, memoria a cambio fase ed altre.