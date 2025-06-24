>[!note]
>Il BUS del calcolatore è un organo di collegamento che collega tutte le unità funzionali di un calcolatore.
>
>Definiamo una transazione di BUS come un insieme di operazioni sul BUS che premette di raggiungere un obiettivo. Distinguiamo quindi la transazione di trasferimento e la transazione di interrupt.
>
>I BUS del calcolatore si distinguono in:
>- Interni: confinati all'interno d'una singola unità funzionale.
>- Esterni: collegano le unità funzionali.

I calcolatori più semplici sono dotati di un solo BUS esterno che collega CPU, memoria e unità di I/O, tuttavia la maggior parte dei calcolatori odierni è dotata di numerosi BUS esterni, in particolare:
- BUS di memoria
- BUS di I/O

Le attività del calcolatore si sviluppano per cicli di BUS: in ogni ciclo avviene un operazione. Le operazioni sono governate dal master che detiene il controllo del BUS. Gli slave non possono dare inizio a un'operazione in modo autonomo.

>[!tip] Transazione di interrupt
>Una transazione di interrupt è un insieme di operazioni che conduce da una segnalazione di interrupt da parte di una periferica alla sua presa in carico da parte delle CPU.

>[!tip] Transazione di trasferimento
>Una transazione di trasferimento si può generalmente scomporre in due fasi principali: la fase di arbitraggio che seleziona l'unità master, e la fase di trasferimento, durante il quale il master seleziona lo slave, indica la direzione del trasferimento e trasferiscono delle unità di informazione.

Per realizzare la scansione dei cicli di BUS esistono due metodi fondamentali:
- BUS sincrono: governato da un segnale di clock.
- BUS asincrono: governato solo dall'interazione tra master e slave.

Nei BUS asincroni ci sono degli opportuni segnali di controllo che realizzano un protocollo di sincronizzazione a segnale. Questi sono i segnali `MSYN` e `SSYN`. In ordine:
- `MSYN` è attivato.
- `SSYN` viene attivato in risposta a  `MSYN`.
- `MSYN` viene disattivato in risposta a  `SSYN`.
- `SSYN` viene disattivato in risposta a `MSYN`.

Questa operazione ha nome full-handshake ed è indipendente dal clock ed è insensibile alla presenza di SLAVE lenti.