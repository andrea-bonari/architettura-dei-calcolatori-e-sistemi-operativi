>[!note]
>La realizzazione di un SO multiprogrammato come Linux o Windows richiede da parte dell'hardware la disponibilità di alcuni meccanismi fondamentali. Usiamo come riferimento specifico `x64`. Alcune sue funzionalità sono inutilmente complesse per motivi di compatibilità, ne sarà fornite una versione semplificata.

Nell'`x64` la pila cresce da indirizzi alti verso indirizzi bassi. Il decremento e l'incremento dello SP sono svolti nella stessa iscrizione di scrittura e lettura in memoria e quindi `push` e `pop` richiedono una sola istruzione ciascuno.

Inoltre, in `x64` si salva il valore dell'indirizzo di ritorno sulla pila, non in un registro.

### Istruzioni privilegiate

### Modello di memoria

### Meccanismo di interruzione

### Interfacce standard e ABI