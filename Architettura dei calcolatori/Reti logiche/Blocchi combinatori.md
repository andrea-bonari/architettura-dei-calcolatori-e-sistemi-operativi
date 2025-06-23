>[!note]
>Esiste una ben nota e ormai libreria di blocchi funzionali predefiniti di tipo combinatori, che contiene i blocchi per tutte le funzioni combinatorie base.
>
>I tipici blocchi funzionali combinatori sono:
>- Multiplexer
>- Decoder
>- Comparator
>- Half adder e full adder
>- Adder a $n\text{ bit}$
>- ALU

### Multiplexer
>[!note]
>Il blocco funzionale multiplexer ha $n\geq1$ ingressi di selezione, $2^{n}\geq2$ ingressi dati e $1$ uscita, dove gli ingressi dati sono numerati a partire da $0$.
>
>Se sugli ingressi di selezione è presenta la configurazione binaria corrispondente a $k$, allora il $k$-esimo ingresso dati viene presentato in uscita.
>
>Il simbolo funzionale è il seguente (estendibile a $n$ ingressi).
>
>![[Pasted image 20250621173123.png|center]]
>Considerando un ingresso per $k=3$, questo è lo schema circuitale.
>
>![[Pasted image 20250621173219.png|center]]

### Decoder
>[!note]
>Il blocco funzionale decoder ha $n\geq 1$ ingressi e $2^{n}\geq2$ uscite, con le uscite numerate a partire da $0$.
>
>Se sugli ingressi è presente un numero binario $k$, la $k$-esima uscita assume valore $1$ e le restanti uscite assumono valore $0$.
>
>Il simbolo funzionale è il seguente (estendibile a $n$ ingressi).
>
>![[Pasted image 20250621173410.png|center]]

### Comparator
>[!note]
>Il blocco funzionale comparatore ha due gruppi $A$ e $B$ di ingressi da $n\geq1\text{ bit}$ ciascuno e $3$ uscite, che indicano rispettivamente minoranza ($A<B$), uguaglianza ($A=B$), e maggioranza $A> B$.
>
>Il blocco confronta i due numeri binari $A$ e $B$ da $n\text{ bit}$ presenti sui due gruppi di ingressi, e attiva l'uscita corrispondente all'esito del confronto. 
>
>Il simbolo funzionale è il seguente (estendibile a $n$ ingressi).
>
>![[Pasted image 20250621173636.png|center]]

### Shifter combinatorio
>[!note]
>Il blocco funzionale shifter ha $n\geq1$ ingressi, $1$ ingresso per il bit aggiunto a destra, $1$ ingresso per il bit aggiunto a sinistra, un ingresso di controllo che comanda lo scorrimento a destra o a sinistra, e $n\geq1$ uscite.
>
>Nel caso di scorrimento a destra (divisione per $2$), viene aggiunto un bit a sinistra, e gli ingressi sono shiftati di una posizione a destra.
>Viceversa, nel caso di scorrimento a sinistra (moltiplicazione per $2$), viene aggiunto un bit a destra e gli ingressi sono shiftati di una posizione a sinistra.
>
>Lo schema circuitale per $5$ ingressi è il seguente.
>
>![[Pasted image 20250621174036.png|center]]

### Half adder
>[!note]
>Il blocco funzionale half adder ha due ingressi $A$ e $B$ e due uscite $\text{Carry}$ e $\text{Sum}$. Ci permette di sommare binariamente $A$ e $B$ tenendo conto di un resto.
>
>Lo schema circuitale è il seguente.
>
>![[Pasted image 20250621174407.png|center]]

### Full adder
>[!note]
>Il blocco funzionale full adder funziona esattamente come un half adder, con l'aggiunta di un ingresso $\text{CarryIn}$ che permette di fare operazioni col riporto di ingresso: $$\begin{align*}
>\text{Sum}&= A\text{ xor }B\text{ xor }C\\
>\text{Carry}&= A\cdot B+ C\cdot(A\text{ xor }B) 
>\end{align*}$$
>
>Lo schema circuitale è il seguente.
>
>![[Pasted image 20250621174638.png|center]]

### Adder a $n\text{ bit}$ in binario naturale intero
>[!note]
>Tramite i blocchi half adder e full adder possiamo creare un blocco che ci permette di eseguire la somma tra due numeri interi. Il simbolo funzionale (estendibile a $n\text{ bit}$) è il seguente.
>
>![[Pasted image 20250621175114.png|cente]]
>
>Lo schema circuitale è il seguente.
>
>![[Pasted image 20250621174810.png|center]]

Per quanto riguarda l'algebra in complemento a due su $k$ bit, per la somma $A+B$ si opera in binario naturale intero, e si verifica overflow se $A$ e $B$ hanno segno concorde ed il risultato ha segno discorde.

Per la sottrazione $A - B=A+(-B)$ si complementa a due l'operando $-B$, e poi si opera in binario naturale intero. Si verifica overflow se $A$ e $-B$ hanno segno concorde ed il risultato ha segno discorde.

Quindi un sommatore/sottrattore in complemento a due è un sommatore binario naturale intero a $k$ bit con un ingresso $\text{CarryIn}$, un ingresso di segnale `add/sub`, una batteria di `xor` sull'ingresso $B$ per effettuare il complemento ad uno, e una rete combinatoria per generare il segnale di overflow.

### ALU
>[!note]
>Il blocco funzionale ALU (Arithmetic Logic Unit) è un blocco funzionale che permette di eseguire le operazioni `AND`, `OR`, `add`, `subtract`, `set less than` e `NOR`. Funziona inserendo nelle righe di controllo dell'ALU un codice associato alla funzione da svolgere. Di seguito il simbolo funzionale accompagnato dai valori notevoli.
>
>![[Pasted image 20250621180410.png|center]]

Per quanto riguarda i numeri relativi, si usa il complemento a due. Le ALU sono normalmente in grado di operare sia con numeri interi naturali, che con numeri interi relativi rappresentati in complemento a due.

I numeri reali sono descritti dallo standard IEEE 754 per la rappresentazione in virgola mobile. Esistono delle ALU in grado di effettuare i calcoli aritmetici con i numeri reali, oltre che con i numeri interi.