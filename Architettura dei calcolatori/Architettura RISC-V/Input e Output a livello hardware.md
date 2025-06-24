>[!note]
>Le unità periferiche interagiscono con il calcolatore, e cioè con il processore e la memoria centrale, attraverso interfacce di ingresso/uscita collegate al bus di sistema.
>
>La struttura delle interfacce I/O è costituita da alcuni registri leggibili/scrivibili da parte del processore, ma comunicanti anche con il mondo esterno tramite circuiti ausiliari.
>
>Considerando uno schema molto semplificato, un'interfaccia è costituita dai seguenti registri:
>- Registro dati periferica: contiene i dati che la periferica deve leggere o ha scritto.
>- Registro comandi periferica: contiene indicazioni sulle operazioni che la periferica deve compiere e sul suo funzionamento.
>- Registro di stato periferica: contiene informazioni sullo stato di funzionamento della periferica. Dispone di un bit `ready` relativo alla possibilità, per il processore, di svolgere un'operazione di lettura o scrittura di un dato.

La CPU può accedere ai registri delle periferiche con due tecniche:
- Memory mapped I/O: assegna ai registri delle periferiche degli indirizzi di memoria (da $0$ a $256$, usata da RISC-V).
- Port mapped I/O: il processore interagisce con le periferiche usando istruzioni macchina specializzate.

>[!tip] Controllo di programma
>Si può gestire I/O tramite il controllo di un programma, cioè sincronizzazione e trasferimento vengono eseguiti dal programma di gestione della periferica (device driver).

>[!tip] Interrupt
>Il meccanismo di interruzione si basa su un'insieme di eventi rilevati a livello hardware dal processore, e un insieme di funzioni, ognuna associata in generale ad un evento. Ogni funzione è detta gestore dell'interrupt o routine di interrupt.
>
>Quanto il processore rileva un evento, interrompe il programma in esecuzione, salta automaticamente ad eseguire la routine di interrupt, e al termine dell'esecuzione della routine torna ad eseguire il programma che era stato interrotto.
>
>Il meccanismo di interruzione è simile a quello di invocazione di una funzione, con la differenza che le routine di interrupt sono causate da eventi rilevati dal processore e asincroni rispetto all'esecuzione del programma che viene interrotto.
>
>Siccome viene salvato l'indirizzo di ritorno è possibile gestire interruzioni annidate.

>[!tip] Direct Memory Access (DMA)
>Il meccanismo di DMA prevede che il driver della periferica trasferisca in modo autonomo, cioè senza intervento del processore, un certo numero di dati in memoria centrale, o dalla memoria centrale.
>
>Il DMA controller effettua il trasferimento dati a blocchi tra periferiche e memoria autonomamente, cioè senza l'intervento da parte del processore. È quindi in grado di generare i segnali per accedere a memoria.
>
>Consideriamo ad esempio le interfacce dei dispositivi di memorizzazione non volatile, detti volumi (SSD o HDD), loro utilizzano DMA per il trasferimento fisico dei dati.
>
>La tecnica di DMA prevede le seguenti fasi:
>1. Predisposizione: una procedura del SO scrive nei registri dell'interfaccia della periferica l'indirizzo della memoria e della periferica sul quale fare il trasferimento, il numero di parole di memoria da trasferire, e la direzione del trasferimento.
>2. Attivazione: una procedura del SO scrive nel registro di controllo dell'interfaccia il comando di avviamento dell'operazione, e quindi il processore passa ad eseguire altri programmi.
>3. Trasferimento del blocco di dati.
>4. Conclusione: se l'ultimo dato è stato trasferito, l'interfaccia segnala al processore tramite interrupt che l'operazione DMA si è conclusa
>   
>   
>Ai fini della realizzazione del SO, le periferiche gestite in DMA richiedono che il SO gestisca opportunamente delle aree di memoria (dette buffer) per il trasferimento dati in modalità DNA da/per periferiche.

