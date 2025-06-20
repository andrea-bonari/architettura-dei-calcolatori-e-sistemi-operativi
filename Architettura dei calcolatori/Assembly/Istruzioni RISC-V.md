>[!note]
>In RISC-V hanno tutte la dimensione di $32\text{ bit}$, e sono caratterizzate da i primi $25\text{ bit}$ di un istruzione usati per indicare i parametri, e i rimanenti $7\text{ bit}$ (quelli meno significativi) usati per indicare il codice operativo dell'istruzione.
>  
>![[Pasted image 20250620144239.png|center]]

Le istruzioni possono referenziare al massimo 3 registri (a $64\text{ bit}$) oppure indicare direttamente un valore numerico.

Riguardo al modello di memoria gli indirizzi sono a $64\text{ bit}$ espressi con indirizzamento a byte, e le istruzioni hanno indirizzi allineati distanti quattro unità.

Le istruzioni in RISC-V sono di $6$ tipi:

| Formato                      | Istruzioni                                                                                                  | Parametro Immediato                       | Immediato esteso ($64\text{ bit}$) |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------- | ---------------------------------- |
| `R` (register)               | aritmetico-logiche                                                                                          | n/a                                       | n/a                                |
| `I` (immediate)              | di caricamento da memoria (`load`), aritmetico-logiche con immediati, salto incondizionato, link a registro | $12\text{ bit}$ in CPL2 (Complemento a 2) | estensione segno                   |
| `S` (store)                  | di memorizzazione in memoria (`store`)                                                                      | $12\text{ bit}$ in CPL2                   | estensione segno                   |
| `B` (branch)                 | branch condizionale                                                                                         | $12\text{ bit}$ in CPL2, multiplo di $2$  | estensione segno                   |
| `U` (load upper integer)     | istruzione `lui`                                                                                            | $12\text{ bit}$ in CPL2                   | N/A                                |
| `J` (jump & link) (UJ in PH) | salto incondizionato e link                                                                                 | $12\text{ bit}$ in CPL2, multiplo di $2$  | estensione segno                   |
Nelle istruzioni di tipo immediato il parametro:
- in `load` e `store` è offset rispetto a un registro base per individuare l'indirizzo di memoria riferito al dato
- in `branch` e `jump` è rispetto al `PC`, o un altro registro, per individuare l'indirizzo di memoria dell'istruzione destinazione del salto

>[!tip]
>![[Pasted image 20250620144332.png|center]]
>
>Il campo `funct3` e `funct7` agisce come estensione del codice operativo per specificare esattamente cosa fa la funzione, in aggiunta a quanto già indicato dal codice operativo

### Istruzioni aritmetico-logiche
>[!note]
>In RISC-V, un'istruzione aritmetico-logica possiede tre operandi: i due registri contenenti i valori da elaborare (registri sorgente) e il registro contenente il risultato (registro destinazione). L'ordine degli operandi è fisso: prima il registro contenente il risultato dell'operazione e poi due operandi:
>```
>OPCODE DEST, SORG1 SORG2
>```

| Sintassi            | Traduzione        | Significato                                                                                                                                                                                                                                     |
| ------------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `add rd, rs1, rs2`  | `rd <- rs1 + rs2` | `add` addiziona il contenuto di due registri sorgente `rs1` e `rs2` e mette nel registro destinazione `rd` il risultato dell'addizione dei contenuti di `rs1` e `rs2`.                                                                          |
| `sub rd, rs1, rs2`  | `rd <- rs1 - rs2` | `sub` sottrae il contenuto dei due registri sorgente `rs1` (minuendo) e `rs2` (sottraendo) e mette nel registro destinazione `rd` il risultato della sottrazione dei contenuti di `rs1`.                                                        |
| `addi rd, rs1, NUM` | `rd <- rs1 + NUM` | `addi` addiziona una costante `NUM` a un registro `rs1` e mette nel registro di destinazione `rd`. La costante deve essere un numero rappresentabile su $12\text{ bit}$ e viene addizionata dopo averla estesa in segno fino a $64\text{ bit}$. |
RISC-V non ha un istruzione di sottrazione di costante perché basta usare `addi` con costante di segno cambiata.

>[!example]
>Data la formula:
>$$a=(b+c)-(c-f)+g$$
>Per eseguire le operazioni di addizione e sottrazione è necessario che le variabili (tipicamente locali) siano nei registri:
>- `s1` è associato a $b$, `s2` a $c$, `s3` a $d$, `s4` a $f$, `s5` a `g` e il risultato finale viene messo in `s3` (riutilizzato)
>- `t0` e `t1` contengono valori temporanei
>
>```asm
>add t0, s1, s2
>sub t1, s3, s4
>sub t0, t0, t1
>add s3, t0, s5
>```

### Istruzioni di trasferimento con la memoria
>[!note]
>Questo tipo di istruzioni si occupano di trasferire dati dalla memoria fisica ai registri (`load`) e viceversa (`store):
>```
>OPCODE REG, MEMADDR
>```

| Sintassi           | Traduzione                  | Significato                                                                                                                                                                                                                                                                                                                                                             |
| ------------------ | --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ld rd, NUM(LOC)`  | `rd <- memoria[LOC + NUM]`  | L'istruzione `ld` di load trasferisce una copia della parola doppia contenuta in una specifica locazione di memoria (`LOC + NUM`)  (a $64\text{ bit}$) mettendola in un registro della CPU, e lasciando inalterata la parola di memoria. Esistono anche le varianti `lw`, `lh` e `lb` che caricano rispettivamente $32 \text{ bit}$, $16 \text{ bit}$ e $8\text{ bit}$. |
| `sd rs2, NUM(LOC)` | `memoria[LOC + NUM] <- rs2` | L'istruzione `sd` di store trasferisce una parola doppia da un registro della CPU (a $64\text{ bit}$) mettendola in una specifica locazione di memoria (`LOC + NUM`) e sovrascrivendo il contenuto precedente di quella locazione.                                                                                                                                      |
| `li t0 VAL`        | `t0 <- VAL`                 | L'istruzione `li` carica una costante `VAL` nel registro `t0`.                                                                                                                                                                                                                                                                                                          |
| `la a0, LABEL`     | `a0 <- LABEL`               | L'istruzione `la` carica un indirizzo `LABEL` in `a0` dove `LABEL` è un etichetta simbolica per indicare il valore dell'indirizzo di memoria di un dato. Vengono caricati i $32\text{ bit}$ meno significativi, mentre i $32$ più significativi sono un a estensione di segno.                                                                                          |
| `.eqv NAME, VAL`   | `#define NAME VAL`          | La direttiva `.eqv` associa un valore decimale al simbolo `NAME`. Non è un istruzione macchina, è solo una dichiarazione di costante.                                                                                                                                                                                                                                   |

Durante l'istruzione `ld` la CPU invia l'indirizzo della locazione desiderata alla memoria e richiede un'operazione di lettura del contenuto della memoria. Poi la memoria effettua la lettura della parola memorizzata all'indirizzo specificato e la invia alla CPU.

Durante l'istruzione `sd` la CPU invia l'indirizzo della locazione desiderata alla memoria, insieme alla parola che vi deve essere scritta, e richiede un'operazione di scrittura. Poi la memoria effettua la scrittura della parola all'indirizzo specificato.

>[!tip] Funzionamento della memoria
>Una parola doppia contiene più byte, per esempio una parola da $32 \text{ bit}$ contiene $4\text{ byte}$. I byte di una parola possono essere enumerati da sinistra a destra o viceversa:
>- Big Endian, con i byte numerati da sinistra a destra
>- Little Endian, con i byte numerati da destra a sinistra
>
>RISC-V usa little endian.
>
>Con parole da $32 \text{ bit}$ e indirizzamento della memoria per byte, le parole consecutive sono collocate a indirizzi multipli di $4$.
>
>![[Pasted image 20250620115007.png|center]]

### Istruzione logiche

| Istruzione             | Significato                                                                                                                     |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `and rd, rs1, rs2`     | `and` esegue l'and bit a bit tra il contenuto dei due registri `rs1` e `rs2` e ne memorizza il risultato in `rd`.               |
| `or rd, rs1, rs2`      | `or` esegue l'or bit a bit tra il contenuto dei due registri `rs1` e `rs2` e ne memorizza il risultato in `rd`.                 |
| `xor rd, rs1, rs2`     | `xor` esegue lo xor bit a bit tra il contenuto dei due registri `rs1` e `rs2` e ne memorizza il risultato in `rd`.              |
| `not rd, rs1`          | `not` esegue il not bit a bit sul contenuto di `rs1` e ne memorizza il risultato in `rd`. È realizzata come `xori rf, rs1, -1`. |
| `slli rd, rs1, cost12` | `slli` (shift left logical) sposta a sinistra il contenuto di `rs1` del numero specificato e introduce bit $0$ a destra.        |
| `srli rd, rs1, cost12` | `srli` (shift right logical) sposta a destra il contenuto di `rs1` del numero specificato e introduce bit $0$ a sinistra.       |
| `srai rd, rs1, cost12` | `srai` (shift right arith) sposta a destra il contenuto di `rs1` del numero di bit specificato, ed estende il segno a sinistra. |
Esistono anche `andi`, `ori` e `xori` che eseguono rispettivamente and, or e xor bit a bit tra un operando e una costante (la costante viene prolungata da $12\text{ bit}$ a $64\text{ bit}$ tramite l'estensione di segno).

Nelle operazioni di shift si ha che `cost12` è una costante intera positiva compresa tra $1$ e $63$. Inoltre esistono anche le versioni `sll`, `srl` e `sra` in cui il terzo argomento è un registro `rs2` che specifica il numero di posizioni di scorrimento.

>[!tip] Gestione degli array di int a $64\text{ bit}$
>Si ha che la locazione dell'elemento $i$-esimo di un array si troverà in posizione: $$\text{br}+8\cdot i$$
>Dove $\text{br}$ è il registro base e $i$ è l'indice dell'elemento dell'array.
>
>Consideriamo `a = A[i]` e assumiamo che $i$ sia in `s1`, l'indirizzo del primo elemento dell'array sia contenuto nel registro `s3` e `a` sia in `s2`:
>
>```
>slli t1, s1, 3        # t1 contiene i * 8
>add t2, t1, s3    # t2 contiene indirizzo di A[i]
>ld s2, 0(t2)       # carico in s2 il valore di A[i]
>```
>
>Se poi volessi caricare `A[i+1]` basterebbe modificare lo spiazzamento:
>```
>ld s2, 8(t2)
>```

### Istruzione di salto
>[!note]
>Le istruzioni di salto si suddividono in salti condizionati e non condizionati. Nei salti condizionati si ha la forma:
>```
>OPCODE SORG1, SORG2, IND_SALTO
>```
>Avviene cioè on confronto tra `SORG1` e `SORG2` e se la condizione viene soddisfatta, allora salta. L'indirizzo di destinazione di salto `IND_SALTO` è espresso relativamente rispetto al `PC`.
>Nelle istruzioni di salto incondizionato il salto viene sempre eseguito.

| Istruzione                | Traduzione                           | Significato                                                                                                 |
| ------------------------- | ------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| `beq rs1, rs2, ind_salto` | /                                    | Se confrontando `rs1` e `rs2` essi sono uguali allora si effettua il salto.                                 |
| `bne rs1, rs2, ind_salto` | /                                    | Se confrontando `rs1` e `rs2` essi sono diversi allora si effettua il salto.                                |
| `blt rs1, rs2, ind_salto` | /                                    | Se confrontando `rs1` e `rs2` si ha che $\text{rs1}<\text{rs2}$ allora si effettua il salto.                |
| `bge rs1, rs2, ind_salto` | /                                    | Se confrontando `rs1` e `rs2` si ha che $\text{rs1}\geq\text{rs2}$ allora si effettua il salto.             |
| `j spi20`                 | `pc <- pc + spi20`                   | Si effettua un salto relativo a `pc` di `spi20`.                                                            |
| `jr rs1`                  | `pc <- rs1`                          | Si effettua un salto impostando `pc` al valore di `rs1`.                                                    |
| `jal spi20`               | `ra <- pc + 4`<br>`pc <- pc + spi20` | Si effettua un salto relativo a `pc` di `spi20`, salvando l'indirizzo di rientro nel registro `ra`.         |
| `jalr rs1`                | `ra <- pc + 4`<br>`pc <- rs1`        | Si effettua un salto impostando `pc` al valore di `rs1`, salvando l'indirizzo di rientro nel registro `ra`. |
| `ret`                     | `pc <- ra`                           | Salto impostando `pc` al valore di `ra`.                                                                    |

L'offset di un salto può variare tra $[-2^{11},+2^{11})$.
### Etichette
>[!note]
>Nelle istruzioni in un linguaggio macchina si usano gli indirizzi, tuttavia per un programmatore non è conveniente, o possible calcolare l'indirizzo numerico di dati e istruzioni. Si usano quindi etichette mnemoniche dette label che si associano a specifici punti del programma, l'assemblatore poi traduce le etichette nei corrispondenti indirizzi.
>
>Per le istruzioni:
>```
>LABEL_NAME:    addi x4, x5, 10
>                sub x4, x4, x6
>                ...
>```
>
>Mentre per i dati:
>```
>.data
>A:    .zero 8
>B:    .zero 8
>C:    .zero 8
>```

>[!example]
>Consideriamo il codice c:
>```c
>if (i == j)
>	f = g + h
>else
>	f = g - h
>```
>E supponiamo che le variabili `f`, `g`, `h`, `i` e `j` siano associate rispettivamente ai registri `s0`, `s1`, `s2`, `s3` e `s4`. Allora una traduzione in RISC-V:
>
>```
>IF:         bne s3, s4, ELSE
>THEN:       add s0, s1, s2
>			j ENDIF
>ELSE:       sub s0, s1, s2
>ENDIF:      ...
>```
>
>Notiamo che le etichette `IF` e `THEN` non servono per saltare, ma aiutano a distinguere le varie parti.

### Istruzioni di confronto

| Istruzione         | Significato                                                                                                                              |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `slt s1, s2, s3`   | L'istruzione `slt` assegna il valore $1$ a `s1` se $\text{s2}<\text{s3}$, altrimenti assegna il valore $0$.                              |
| `slti s1, s2, NUM` | L'istruzione `slti` assegna il valore $1$ a `s2` se $\text{s2}<\text{NUM}$, dove `NUM` è una costante. Altrimenti assegna il valore $0$. |
`slt`/`slti` esegue i confronti in aritmetica cno segno, cioè interprentando i valori come CPL2. Se i valori numeri sono degli indirizzi bisogna usare `sltu`/`sltiu`.

>[!example]
>Consideriamo il codice c:
>```c
>if (i < j)
>	k = i + j
>else
>	k = i - j
>```
>E supponiamo che le variabili `i`, `j` e `k` siano rispettivamente associate a `s0`, `s1` e `s2`.
>
>```
>IF:         slt t0, s0, s1
>			beq t0, zero, ELSE
>THEN:       add s2, s0, s1
>			j ENDIF
>ELSE:       sub s2, s0, s1
>ENDIF:      ...
>```

### Gestione delle costanti
>[!note]
>Le istruzioni di tipo I permettono di rappresentare costanti esprimibili su $12\text{ bit}$ in CPL2. Per avere una costante a $32\text{ bit}$ è necessario caricarla nel registro in due passi. Suddividiamo il valore della costante in una da $20\text{ bit}$ e l'altra da $12\text{ bit}$, che sono trattate come due immediati:
>- Si usa l'istruzione `lui` per caricare il primo immediato nei $20\text{ bit}$ più significativi della metà inferiore del registro. E poi azzera i $12\text{ bit}$ meno significativi, ed estende il segno della metà inferiore fino al $64$-esimo bit.
>- Una successiva istruzione `addi` consente di aggiungere tramite il secondo immediati, i $12\text{ bit}$ meno significativi della costante. 

| Istruzione        | Traduzione         | Significato                                                                                                                                                                                                                                                                |
| ----------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `lui rd, cost20`  | `rd <- cost20`     | Carica parzialmente la costante `cost20` nei $20\text{ bit}$ significativi della metà inferiore da $32\text{ bit}$ del registro `rd`, poi azzera i $12\text{ bit}$ meno significativi di `rd`, e infine estende in segno la metà inferiore di `rd` fino al $64$-esimo bit. |
| `auipc rd, spi20` | `rd <- pc + spi20` | Carica parzialmente un indirizzo relativo al `pc`. Prima somma lo spiazzamento `spi20` all'indirizzo dell'istruzione `auipc`, e poi lo tratta come la costante nell'istruzione `lui` e lo carica nel registro `rd`.                                                        |
Normalmente la costante `cost20` è specificata come valore numerico o come costante simbolica dichiarata tramite la direttiva `.eqv`.

In realtà l'istruzione `lui` carica nei $20\text{ bit}$ più significativi: $$\text{immediato\_20bit}[31:12]+\text{immediato\_{12bit}}[11]$$
Questo perché le espressioni immediate usano CPL2. Inoltre queste espressioni sono usate per semplificare le pseudoistruzioni `li` e `la`.

### Invocazione del sistema operativo
>[!note]
>Ci è fornita un istruzione di chiamata (invocazione) di sistema operativo, che fornisce un insieme di funzioni di servizio. Per richiedere una funzione di servizio bisogna impostare il codice di chiamata nel registro `a7`, inserire gli argomenti nei registri richiesti, e utilizzare l'istruzione macchina `ecall` senza argomenti.

>[!example]
>Traduciamo il seguente codice c in assembly:
>```c
>#include <stdio.h>
>
>void main() {
>	printf("Hello, World!");
>	return;
>}
>```
>
>```armasm
>			.data
>OUT_STRING: .asciz "\nHello, World!\n"
>			.text
>MAIN:       li a7, 4
>			la a0, OUT_STRING
>			ecall
>EXIT:       li a7, 93
>			li a0, -1
>			ecall
>```

### Direttive

| Direttiva       | Significato                                                                                                  |
| --------------- | ------------------------------------------------------------------------------------------------------------ |
| `.align num`    | Allinea il dato dichiarato secondo `num`: $0$ = byte, $1$ = half, $2$ = word, $3$ = dword.                   |
| `.ascii str`    | Riserva spazio per la stringa `str` nel segmento dati, **senza** terminatore NULL.                           |
| `.asciiz str`   | Riserva spazio per la stringa `str` nel segmento dati, **con** terminatore NULL.                             |
| `.byte b1,...`  | Riserva spazio per i byte elencati e li inizializza con i valori a $8\text{ bit}$.                           |
| `.data ind`     | Dichiara il segmento dati con indirizzo iniziale `ind` (default: `0x0000000010000000`).                      |
| `.dword d1,...` | Riserva spazio per parole doppie ($64\text{ bit}$), inizializzate e allineate su multipli di $8$.            |
| `.eqv sym, val` | Definisce un simbolo `sym` con valore `val` (come `#define` in C, senza allocare memoria). Locale al modulo. |
| `.globl sym`    | Rende il simbolo `sym` visibile agli altri moduli oggetto.                                                   |
| `.half h1,...`  | Riserva spazio per mezze parole ($16\text{ bit}$), inizializzate e allineate su indirizzi pari.              |
| `.space num`    | Riserva spazio per `num` byte non inizializzati. Usata per array, struct o quando mancano valori iniziali.   |
| `.text ind`     | Dichiara il segmento testo (codice) con indirizzo iniziale `ind` (default: `0x0000000000400000`).            |
| `.word w1,...`  | Riserva spazio per parole ($32\text{ bit}$), inizializzate e allineate su multipli di $4$.                   |
| `.zero num`     | Come `.space`, ma inizializza a zero (segmento BSS). Utilizzata da GCC.                                      |
### Indirizzamento
>[!note]
>RISC-V ha quattro modalità di indirizzamento:
>- A registro
>- Immediato
>- Con base e spiazzamento
>- Relativo al Program Counter `pc`.