>[!note]
>Il processo di traduzione da linguaggio sorgente di alto livello a linguaggio macchina è diviso in quattro fasi:
>- Analisi sintattica e trattamento errori
>- Generazione del codice macchina (forma simbolica)
>- Ottimizzazione del codice macchina
>- Assemblaggio e collegamento
>
>È possibile avere delle rappresentazioni intermedie come l'albero sintattico del programma e il codice macchina non ottimizzato.
>
>Spesso si sceglie il modello del processore per cui tradurre per ottenere del codice macchina ad hoc.

### Assemblaggio e collegamento
>[!note]
>Il processo di assemblaggio risolve i simboli dell'assemblatore, tra cui indirizzi di memoria, etichetta d'istruzione e spiazzamento, e li traduce in una codifica numerica. In questo processo inoltre le pseudoistruzioni vengono convertite nelle loro istruzioni equivalenti.
>
>Infine il processo di collegamento (linking) unisce più moduli eseguibili (per esempio programma e librerie standard).

### Modello di compilazione
>[!note]
>Di seguito si illustra un modello di compilazione per tradurre da linguaggio c a linguaggio macchina RISC-V, ispirato a:
>- ISA RV64i (Aritmetica intera a $64\text{ bit}$)
>- ABI di GCC
>
>Questo modello consiste di insiemi di convenzioni e regole per:
>- Segmentare il programma
>- Dichiarare le variabili
>- Usare (leggere e scrivere) le variabili
>- Rendere le strutture di controllo
>
>Tuttavia non attua la traduzione più efficiente.

### Processo
>[!note]
>Per il sistema operativo il processo tipico ha tre segmenti essenziali:
>- Codice (o testo): funzione main e funzioni utente
>- Dati: variabili globali e dinamiche
>- Pila: aree di attivazione di funzione, contenenti indirizzi, parametri, eventuali registri salvati e variabili locali
>
>Il segmento pila è creato al lancio del processo.
>
>Consideriamo la seguente immagine come il modello di memoria usato da RISC-V, cioè il modello $38\text{ bit}$ corrente.
>
>![[Pasted image 20250620163204.png|center]]
>
>Siccome RISC-V fa uso di indirizzi da $64\text{ bit}$, si ha come massimo spazio di memoria: $$2^{64}\text{ byte}$$

Per indicare il testo in RISC-V usiamo la direttiva `.text`, mentre per indicare i dati si usa `.data`, non è necessario dichiarare esplicitamente gli indirizzi.

```
		.data
		...
		
		.text
MAIN:   ...
		...
```

### Dichiarazione di variabili
>[!note]
>Per tradurre da linguaggio sorgente a linguaggio macchina bisogna definire un modello di architettura run-time per memoria e processore. Le convenzioni del modello run-time comprendono:
>- Collocazione e ingombro delle diverse classi di variabile
>- Nomenclatura e destinazione di uso dei registri del processore
>  
>Questo modello ci consente l'interoperabilità tra porzioni di codice di provenienza differente, per esempio codice utente e librerie standard precompilate.

I linguaggi ad alto livello come il c permettono di definire variabili con associato un tipo. Ciascuno di questi tipi ha una certa dimensione controllabile con l'istruzione `sizeof(tipo)`. In C, usando Linux, i diversi tipi di variabile ingombrano in questo modo:

| nome          | dimensione                          |
| ------------- | ----------------------------------- |
| char          | $1\text{ byte}$ (b)                 |
| short int     | $2\text{ byte}$ (half)              |
| int           | $4\text{ byte}$ (word)              |
| long int      | $8\text{ byte}$ (dword)             |
| long long int | $8\text{ byte}$ (dword)             |
| array         | Somma degli ingombri degli elementi |
| struct        | Somma degli ingombri dei campi      |
In questo corso non si considera il tipo reale, si può vedere lo standard IEEE 754 per vedere le implementazioni di float e double.

Per operare su una variabile (tipicamente globale) in assembly è necessario:
1. Caricare la variabile in un registro
2. Eseguire le operazioni prendendo il dato dal registro
3. Riscrivere la variabile in memoria

A volte i passi 1 e 3 sono evitabili.

>[!tip] Allineamento in memoria delle variabili
>Le variabili in memoria sono allineate secondo il loro tipo nel seguente modo:
>- Byte: ad un indirizzo qualunque: $0,1,2,\cdots$
>- Half: ad un indirizzo pari: $0,2,4,\cdots$
>- Word: ad un indirizzo multiplo di $4$: $0,4,8,\cdots$
>- Dword: ad un indirizzo multiplo di $8$: $0,8,16,\cdots$
>
>È possibile cambiare l'allineamento usando la direttiva `.align`:
>- `.align 0` per nessun allineamento
>- `.align 1` per allineamento ad indirizzo pari
>- `.align 2` per allineamento ad indirizzo multiplo di $4$
>- `.align 3` per allineamento ad indirizzo multiplo di $8$

Distinguiamo diverse classi di variabili: globali, parametri, locali, e dinamiche.

>[!tip] Variabili globlali
>La variabile globale è collocata in memoria a indirizzo fisso stabilito dall'assemblatore. Per comodità l'indirizzo simbolico della variabile globale coincide con il nome della variabile, anche se solitamente è scritto in maiuscolo. Gli ordini di dichiarazione e disposizione in memoria delle variabili coincidono.
>
>Le variabili globali sono allocate nel segmento dei dati statici a partire dall'indirizzo $\mathtt{0x}0000\space0000\space1000\space0000$, e procede in ordine di dichiarazione come in c. Alle variabili si assegna spazio dagli indirizzi bassi verso quelli alti.
>
>Per allocare vettori si allocano $n\cdot k\text{ byte}$ dove $n$ sono i numeri di elementi, mentre $k$ è la dimensione del tipo.
>
>Per quanto riguarda agli struct, il compilatore riserva uno spazio di $n\text{ byte}$ sufficiente a contenere in sequenza i campi della struct. Si accede poi a un campo interno della struct mediante lo spiazzamento del campo rispetto all'indirizzo iniziale della struct.

>[!example] Vettore con accesso costante
>Convertiamo il seguente codice c in assembly:
>```c
>LONG v [5]
>v[0] = 6
>v[2] = v[1] + 7
>```
>
>```
>V:  .zero 40
>
>	li t0, 6
>	la t1, v
>	sd t0, 0(t1)
>	
>	la t0, V
>	ld t1, 8(t0)
>	addi t1, t1, 7
>	sd t1, 16(t0)
>```

>[!example] Vettore con scansione sequenziale con indice
>Convertiamo il seguente codice c in assembly:
>```c
>LONG v [5]
>LONG a
>
>for (a = 0; a <= 4; a++)
>	v[a] = v[a] + 7
>```
>
>```
>V:      .zero 40
>A:      .dword 0
>
>FOR:    // controllo for
>		la t0, A
>		ld t1, (t0)
>		li t2, 4
>		bgt t1, t2, END
>		
>		// interno ciclo for
>		la t0, V
>		la t1, A
>		ld t2, (t1)
>		slli t2, t2, 3
>		add t0, t0, t2
>		ld t3, (t0)
>		addi t3, t3, 7
>		sd t3, (t0)
>		
>		// aumento contatore
>		la t0, A
>		ld t1, (t0)
>		addi t1, t1, 1
>		sd t1, (t0)
>		
>		j FOR
>END:    ...
>```

>[!example] Vettore con scansione sequenziale con puntatore
>Convertiamo il seguente codice c in assembly:
>```c
>LONG v [5]
>LONG a
>LONG * p
>
>p = v
>for (a = 0; a <= 4; a++) {
>	*p = *p + 7
>	p++
>}
>```
>
>```
>V:      .zero 40
>A:      .dword 0
>P:      .dword 0
>
>		la t0, V
>		la t1, P
>		sd t0, t1
>
>FOR:    // controllo for
>		la t0, A
>		ld t1, (t0)
>		li t2, 4
>		bgt t1, t2, END
>		
>		// *p = *p + 7
>		la t0, P
>		ld t1,  (t0)
>		ld t2, (t1)
>		addi t2, t2, t3
>		sd t2, (t1)
>		
>		// p++
>		la t0, P
>		ld t1, (t0)
>		addi t1, t1, 8
>		sd t1, (t0)
>		
>		// a++
>		la t0, A
>		ld t1, (t0)
>		addi t1, t1, 1
>		sd t1, (t0)
>		
>		j FOR
>
>END:    ...
>```

>[!tip] Parametri in ingresso alla funzione
>I primi sei parametri vanno passati nei registri `a2`,  `a3`, `a4`, `a5`, `a6` e `a7`. Se sono di tipo scalare o puntatore si passa una dword. Nel caso di vettore si punta al primo elemento. Nel caso di struct bisogna vedere l'ABI del compilatore (in caso di GCC si passa il primo campo della struct).
>
>Eventuali parametri rimanenti vanno impilati a cura del chiamante: prima di chiamare la funzione il chiamante salva sulla pila i registri `t0` - `t6` e `a0` - `a7` che il chiamante necessita di riavere inalterati al rientro.
>
>Il valore di uscita va restituito nel registro `a0`. Se occorre si usa anche il registro `a1`.

>[!tip] Variabili locali
>La variabile locale può essere gestita in vari modi, secondo il tipo di variabile e il grado di ottimizzazione del codice, e anche in dipendenza di come la variabile viene utilizzata.
>
>Per le variabili scalari o puntatori si può:
>- Memorizzare in un registro del blocco `s0`-`s11`, se per altro motivo non deve avere un indirizzo di memoria
>- Memorizzare nell'area di attivazione della funzione
>
>Per le variabili di tipo scalare, a cui si accede solo tramite un puntatore, e per gli array e struct, si memorizzano nell'area di attivazione della funzione.

### Uso di sottoprogrammi
>[!note]
>In un linguaggio ad alto livello, la chiamata a sottoprogramma ha come effetto la creazione di un'area di attivazione (activation record o frame) sulla pila (stack). Quando il sottoprogramma termina l'area viene rilasciata (o deallocata) dalla pila.
>
>Con sottoprogrammi annidati le aree vengono impilate sulla pila e l'ultima area corrisponde al sottoprogramma correntemente in esecuzione.
>
>L'area di attivazione è associata alle seguenti informazioni:
>- Parametri formali e i loro valori
>- Indirizzo di rientro al chiamante
>- Le informazioni per gestire lo spazio allocato per l'area di attivazione
>- Le variabili locali
>- Il valore restituito
>
>![[Pasted image 20250620180206.png|center]]
>
>Il programma chiamante deve eseguire le operazioni di disposizione dei parametri di ingresso della procedura in posto accessibile, e trasferire il controllo alla procedura (usando `jal`). La procedura chiamata deve eseguire le operazioni di:
>- Allocazione dell'area di attivazione
>- Esecuzione del suo compito
>- Memorizzazione del risultato in un luogo accessibile al chiamante
>- Restituzione del controllo al chiamante con `ret`
>
>È necessario fare in modo che i registri importanti per la computazione siano salvati in un area di memoria, per poi essere ripristinati. 

La convenzione per il salvataggio/ripristino dei registri adottate da RISC-V è la seguente: il chiamante e il chiamato salvano sulla pila soltanto un particolare gruppo di registri ciascuno, e il chiamato può utilizzare la pila per le strutture dati locali e per le variabili locali.

I registri salvati sono `s0`-`s11` e `fp`, mentre quelli non salvati sono `t0`-`t6`, `a2`-`a7`, `a0`-`a1`, `ra`.

>[!tip] Stack
>La pila (stack) è una struttura dati costituita da una coda LIFO, dove i dati vengono inseriti nella pila con l'operazione `push`, mentre vengono estratti dalla pila con l'operazione `pop`. È necessario un puntatore alla cima della pila per salvare i registri che servono al sottoprogramma chiamato, questo puntatore è memorizzato nello stack pointer `sp` e viene aggiornato ogni volta che viene inserito o estratto il valore di un registro.
>
>L'inserimento di un dato nella pila (operazione di `push`) avviene prima decrementando `sp` per allocare lo spazio, aumentando quindi la dimensione della pila, e poi eseguendo l'istruzione `sd` per inserire il dato.
>
>```
>addi sp, sp, -8
>sd registro, 0(sp)
>```
>
>Viceversa, l'estrazione di un dato dalla pila (operazione di `pop`) avviene prima eseguendo l'istruzione `ld` per estrarre il dato e poi incrementando `sp`p er rilasciare il dato, riducendo quindi la dimensione della pila.
>
>```
>ld registro, 0(sp)
>addi sp, sp, 8
>```
>
>La pila permette di allocare uno spazio di memoria privato per ogni istanza, ed è dinamica.
>
>In RISC-V lo stack frame viene esplicitamente allocato dal programmatore in una sola volta all'inizio della procedura. Quando una procedura rientra, essa rilascia lo stack frame incrementando il registro `sp` di quanto lo aveva decrementato alla chiamata.

Si ha che il registro `fp` punta alla prima parola dell'area di attivazione, mentre il registro `sp` punta all'ultima parola dell'area. Ad ogni operazione di `push`/`pop` modifichiamo sempre dinamicamente `sp`, tuttavia può essere utile sfruttare anche il registro `fp`. Infatti l'uso del registro `fp` è del tutto opzionale.

Il registro `fp` permette di concatenare le aree di attivazione impilate come in una lista con puntatori.

![[Pasted image 20250620213643.png|center]]

I registri `t0`-`t6` e `a0`-`a7` vanno salvati e poi ripristinati da parte del chiamante, se esso deve o preferisce poterli riavere inalterati dopo la chiamata. Invece, i registri `s0`-`s11` vanno salvati e poi ripristinati da parte del chiamato, se esso li usa per allocare le variabili locali.

Anche il registro `ra` è facoltativo, va usato se una procedura ne chiama un altra, o anche se è ricorsiva.

>[!example]
>Traduciamo da c ad assembly la seguente funzione:
>```c
>LONG somma_algebrica (LONG g, LONG, h, LONG, i, LONG j) {
>	LONG f
>	f = (g + h) - (i + j)
>	return f
>}
>```
>Associamo i parametri `g`, `h`, `i` e `j` rispettivamente a `a2`, `a3`, `a4` e `a5`. Associamo la variabile locale `f` ad `s0`. Si ha che il calcolo di `f` richiede solo 3 registri, `s0`, `t0` e `t1`. È quindi necessario salvare solo il registro `s0` nella pila
>```
>// salvataggio nella pila
>addi sp, sp, -8
>sd s0, 0(sp)
>
>// procedura
>add t0, a2, s3,
>add t1, a4, a5
>sub s0, t0, t1
>add a0, s0, zero
>
>// ripristino dalla pila
>ld  s0, 0(sp)
>addi sp, sp, 8
>
>// rientro al chiamante
>jr ra
>```

>[!tip] Funzione foglia
>Una procedura è detta foglia quando non ha annidate al suo interno chiamate ad altre procedure o a se stessa, e quindi non è necessario il salvataggio del registro `ra`.

>[!example] Calcolo del fattoriale
>Ricordandoci la definizione di fattoriale: $$n!=\begin{cases}
 n(n-1)(n-2)\cdots1\qquad &n\geq1 \\
>1&n=0
>\end{cases}$$
>E l'implementazione in c:
>```c
>LONG fatt (LONG n) {
>	LONG tmp
>	tmp = n
>	
>	if (tmp == 0) {
>		return 1
>	} else {
>		tmp = tmp * fatt (tmp - 1)
>		return tmp
>	}
>}
>
>LONG ret = 0
>
>main ( ) {
>	LONG val
>	val = 4
>	ret = fatt (val)
>}
>```
>
>Traduciamo in assembly:
>```
>		.text
>FATT:   // salvataggio stato
>		addi sp, sp, -16
>		sd ra, 8(sp)
>		sd s0, 0(sp)
>		
>		// implementazione metodo
>		mv s0, a2
>IF:     bne s0, zero, ELSE
>THEN:   li a0, 1
>		j ENDIF
>ELSE:   sub a2, s0, 1
>		jal FATT
>		mul s0, s0, a0
>		mv a0, s0
>ENDIF:  // ripristino stato
>		ld s0, 0(sp)
>		ld ra, 8(sp)
>		addi sp, sp, 16
>		ret
>
>		.data
>RET:    .dword 0
>
>		.text
>MAIN:   addi sp, sp, -16
>		sd ra, 8(sp)
>		sd s0, 0(sp)
>		li s0, 4
>		mv a2, s0
>		
>		jal FATT
>		la t0, RET
>		sd a0, 0(t0)
>		
>		// ripristino stato
>		ld s0, 0(sp)
>		ld ra, 0(sp)
>		addi sp, sp, 16
>		
>		// terminazione
>		li a0, 93
>		li a7, 0
>		ecall
>```

### Valutazione di espressioni algebriche
>[!note]
>I compilatori traducono in linguaggio macchina la valutazione di un'espressione algebrica affrontando in modo ottimizzato i seguenti passi schematici:
>- Derivazione dell'albero sintattico dell'espressione
>- Ordinamento dei nodi dell'albero
>- Assegnamento dei registri
>  
>Noi manualmente completiamo l'espressione con le parentesi associando da sinistra, e ripetiamo il procedimento fino ad aver esaminato tutta l'espressione. Poi valutiamo il primo operatore valutabile da sinistra.
>
>Un operatore è valutabile se le sua parti sono una variabile, oppure una costante, oppure una sottoespressione già calcolata in un registro di tipo `t`.