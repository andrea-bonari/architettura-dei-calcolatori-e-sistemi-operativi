>[!note]
>I circuiti digitali sono formati da componenti digitali elementari, chiamati porte logiche. Le porte logiche sono i circuiti minimi per l'elaborazione di segnali digitali e corrispondono agli operatori elementari dell'algebra di commutazione.
>
>Le porte logiche sono classificate in base al metodo di funzionamento: `NOT`, `AND` e `OR`.

>[!tip] Transistor
>L'elemento funzionale fondamentale per la costruzione di porte logiche è il transistor, un dispositivo elettronico che funziona come interruttore. Ha due stati di funzionamento: interruttore aperto o interruttore chiuso.
>
>![[Pasted image 20250621165503.png|center]]
>
>Se la tensione di base $V_\text{ingresso}$ è inferiore ad una data soglia critica, il transistor si comporta come interruttore aperto, cioè tra emettitore e collettore non passa corrente, e quindi la tensione di uscita diventa uguale a quella di alimentazione: $$V_\text{uscita}=V_\text{alimentazione}=5\text{ V (in tecnologia TTL)}$$
>Viceversa, se la tensione di base $V_\text{ingresso}$ è superiore ad una data soglia critica, il transistor si comporta come interruttore chiuso, e quindi la tensione di uscita divenga uguale a quella di massa: $$V_\text{uscita}=0\text{ V (in tecnologia TTL)}$$
>
>Il singolo transistor della figura è l'equivalente di una porta `NOT`.

| Simbolo funzionale                   | Nome  | Funzionamento                                                                                                                                                                  |
| ------------------------------------ | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ![[Pasted image 20250621170020.png]] | NOT   | L'uscita vale $1$ se e solo se l'ingresso vale $0$.                                                                                                                            |
| ![[Pasted image 20250621170103.png]] | AND   | L'uscita vale $1$ se e solo se entrambi gli ingressi valgono $1$.                                                                                                              |
| ![[Pasted image 20250621170137.png]] | OR    | L'uscita vale $1$ se e solo se almeno un ingresso vale $1$.                                                                                                                    |
| ![[Pasted image 20250621170210.png]] | NAND  | L'uscita vale $0$ se e solo se entrambi gli ingressi valgono $1$.                                                                                                              |
| ![[Pasted image 20250621170245.png]] | NOR   | L'uscita vale $1$ se e solo se entrambi gli ingressi valgono $0$.                                                                                                              |
| ![[Pasted image 20250621170326.png]] | XOR   | L'uscita vale $1$ se e solo se una variabile vale $1$ (disuguaglianza). Se generalizzato a $n$ ingressi, l'uscita vale $1$ se e solo se il numero di $1$ è dispari.            |
| ![[Pasted image 20250621170508.png]] | N-XOR | L'uscita vale $1$ se e solo se entrambe le variabili valgono $0$ o $1$ (uguaglianza). Se generalizzato a $n$ ingressi, l'uscita vale $1$ se e solo se il numero di $1$ è pari. |
| ![[Pasted image 20250621170535.png]] | AND   | L'uscita vale $1$ se e solo se tutti e $3$ gli ingressi valgono $1$.                                                                                                           |
| ![[Pasted image 20250621170607.png]] | OR    | L'uscita vale $0$ se e solo se tutti e $3$ gli ingressi valgono $0$.                                                                                                           |
Il numero di transistor per realizzare una porta dipende dalla tecnologia, dalla funzione e dal numero di ingressi. In generale per una porta `NOT` servono $1$ o $2$ transistor, per le porte `AND` e `OR` servono $3$ o $4$ transistor, e per le altre ne servono più di $4$.

La velocità di commutazione di una porta dipende dalla tecnologia, dalla funzione e dal numero di ingressi. Le porte più efficienti sono tipicamente le porte `NAND` e `NOR` a due ingressi: possono commutare in meno di $1 \text{ ns}$.

Possiamo quindi calcolare un indicazione del costo della rete logica e il ritardo di propagazione associato ad essa.

>[!tip] Porte NAND
>È possibile realizzare i tre operatori fondamentali utilizzando le porte NAND:
>
>| NOT | AND | OR |
>| - | - | - |
>| ![[Pasted image 20250621171043.png]] | ![[Pasted image 20250621171057.png]] | ![[Pasted image 20250621171108.png]] |

### Reti combinatorie
>[!note]
>Ad ogni funzione combinatoria, data come espressione booleana, si può sempre associare un circuito digitale, formato da porte logiche, che viene chiamato rete combinatoria. Gli ingressi della rete combinatoria sono le variabili della funzione, mentre l'uscita della rete combinatoria emette il valore assunto dalla funzione.
>
>Una rete combinatoria è quindi un circuito digitale con almeno un ingresso ed un uscita, formato da porte logiche `AND`, `OR` e `NOT` e privo di retroazioni.

La tabella delle verità di una rete combinatoria può anche essere ricavata per simulazione del funzionamento circuitale della rete combinatoria stessa.

Una singola funzione combinatoria può emettere più reti combinatorie differenti che la sintetizzano, pertanto due reti combinatorie che realizzano la medesima funzione combinatoria si dicono equivalenti. Esse hanno tutte la stessa funzione, ma possono avere struttura e costo differente.

Data una funzione booleana, la soluzione iniziale al problema di determinare una sua espressione consiste nel ricorso alle forme canoniche.

Le forme canoniche sono, rispettivamente, la forma somma di prodotti (SoP, prima forma canonica) e quella di prodotto di somma (PoS, seconda forma canonica). Data una funzione booleana esiste una ed una sola forma canonica SoP, ed una e una sola forma PoS che la rappresenta.

>[!tip] Sintesi in prima forma canonica
>Data una tabella delle verità, a $n\geq1$ ingressi, della funzione da sintetizzare, la funzione $F$ che realizza la prima forma canonica può essere specificata come la somma logica di tutti e soli i termini prodotto delle variabili di ingresso corrispondenti agli $1$ della funzione.
>
>Ogni termine prodotto è costituito dal prodotto logico delle variabili di ingresso, prese in forma naturale se valgono $1$, in forma complementata se valgono $0$.

>[!tip] Sintesi in prima forma canonica
>Data una tabella delle verità, a $n\geq1$ ingressi, della funzione da sintetizzare, la funzione $F$ che realizza la seconda forma canonica può essere specificata come il prodotto logico di tutti i termini somma delle variabili di ingresso corrispondenti agli $0$ della funzione.
>
>Ogni termine somma è costituito dalla somma logica delle variabili di ingresso, prese in forma naturale se valgono $0$, in forma complementata se valgono $1$.

