>[!note]
>Il linguaggio assembly è un linguaggio specifico del particolare processore più primitivo rispetto ai linguaggi ad alto livello. È molto restrittivo con il suo set di operazioni.

>[!tip] Stored Program
>I programmi sono sequenze di istruzioni macchina rappresentate in binario, vengono caricati in memoria esattamente come i dati.
>
>Siccome sia i dati che le istruzioni sono codificati in binario la CPU riesce a interpretare solo alcuni codici numerici come istruzioni.

In RISC-V la memoria è indirizzata a byte.

### Processo di compilazione
Il passaggio dal codice sorgente all'esecuzione del programma si articola in tre fasi:
1. Compilazione
2. Linking
3. Caricamento ed esecuzione

### Registri
>[!note]
>RISC-V ha 32 registri di uso generale, sono numerati da $0$ a $31$. e sono indicati come `x0`-`x31`. Inoltre hanno anche dei nomi simbolici convenzionali. Hanno una lunghezza di $64$ bit e possono solo contenere valori interi (I valori decimali hanno dei registri specifici).

>[!tip] Registri referenziabili
>| Registro | nome ABI | Utilizzo |
>| - | - | - | 
>| `x0` | `zero` | Contiene solo una costante $0$ non modificabile |
>| `x1` | `ra` | Indirizzo di rientro da funzione |
>| `x2` | `sp` | Puntatore alla cima della pila |
>| `x3` | `gp` | Puntatore all'area dati globale |
>| `x4` | `tp` | Puntatore a thread |
>| `x5`-`x7` | `temp0`-`temp2` | Valori temporanei |
>| `x8` | `s0`/`fp` | Variabile locale di funzione o frame pointer |
>| `x9` | `s1` | Variabile locale di funzione |
>| `x10` - `x11` | `a0`-`a1` | Valore restituito da funzione |
>| `x12`-`x17`| `a2`-`a7` | Argomenti di ingresso a funzione |
>| `x18`-`x27` | `s2`-`s11` | Variabili locali di funzione |
>| `x28`-`x31` | `t3`-`t6` | Valori temporanei |

### Istruzioni
>[!note]
>Le istruzioni comprese nel linguaggio macchina di qualunque computer cono classificabili in quattro categorie:
>- Aritmetico/Logiche
>- Load/Store
>- Branch/Jump
>- I/O
>  
>  In RISC-V hanno tutte la dimensione di $32$ bit, e sono caratterizzate da i primi $25$ bit di un istruzione usati per indicare i parametri, e i rimanenti $7$ bit (quelli meno significativi) usati per indicare il codice operativo dell'istruzione.

Le istruzioni possono indicare un valore immediato e referenziare al massimo tre registri: due registri sorgente e un registro di destinazione. Gli indirizzi sono a $64$ bit, espressi con indirizzamento a byte.

In RISC-V le istruzioni sono classificabili in 6 categorie:

| Formato                  | Istruzioni                                                                       | Immediato                     | Immediato esteso |
| ------------------------ | -------------------------------------------------------------------------------- | ----------------------------- | ---------------- |
| `R` (register)           | aritmetico-logiche                                                               | n/a                           | n/a              |
| `I` (immediate)          | di load, aritmetico-logiche con immediati, salto incondizionato, link a registro | 12 bit in CPL2                | estensione segno |
| `S` (store)              | di store                                                                         | 12 bit in CPL2                | estensione segno |
| `B` (branch)             | branch condizionale                                                              | 12 bit in CPL2, multiplo di 2 | estensione segno |
| `U` (load upper integer) | istruzione `lui`                                                                 | 20 BIT IN cpl2                | N/A              |
| `J` (jump & link)        | salto incondizionato e link                                                      | 20 bit in CPL2, multiplo di 2 | estensione segno |
##### Istruzioni formato `R`:
>[!note]
>L'istruzione assembler contiene il codice operativo e tre campi relativi a tre operandi: $$\text{OPCODE}\quad\text{DEST},\quad\text{SORG1},\quad\text{SORG2}$$

>[!tip] Istruzione `add`:
>Questa istruzione addiziona il contenuto di due registri sorgente `rs1` e `rs2` e mette nel registro destinazione `rd` il risultato:
>```asm
>add rd, rs1, rs2                    # rd <- rs1 + rs2
>```
>

>[!tip] Istruzione `sub`
>Questa istruzione sottrae il contenuto di due registri sorgente `rs1` e `rs2`, e mette nel registro destinazione `rd` il risultato:
>```asm
>sub rd, rs1, rs2                    # rd <- rs1 - rs2
>```

##### Istruzioni di trasferimento con la memoria
>[!note]
>Sono istruzioni che si occupano di trasferire dati dalla memoria fisica ai registri e viceversa.

>[!tip] Istruzione `ld`
>Questa istruzione trasferisce una copia della parola contenuta in una specifica locazione di memoria `LOC` mettendola in un registro della CPU `rd`:
>```asm
>ld rd, LOC                   # rd <- memoria[LOC]
>```
>
>Possiamo anche specificare un offset in in questo modo:
>```asm
>ld rd, 100(LOC)              # rd <- memoria[LOC + 100]
>```

>[!tip] Istruzione `sd`
>Questa istruzione trasferisce una parola da un registro della CPU `rs2` mettendola in una specifica locazione di memoria `LOC`, sovrascrivendo il contenuto precedente di quella locazione:
>```asm
>sd rs2, LOC                  # memoria[LOC] <- rs2
>```
>Possiamo anche specificare un offset in questo modo:
>```asm
>sd rs2, 100(LOC)             # memoria[LOC + 100] <- rs2
>```

>[!tip] Istruzione `li`
>Questa istruzione carica la costante `const` in `t0`:
>```asm
>li t0, const                 # t0 <- const
>```

>[!tip] Istruzione `la`
>Carica l'indirizzo `LABEL` (non i dati all'indirizzo) in `a0`:
>```asm
>la a0, LABEL                 # a0 <- LABEL
>```

>[!note] Array con elementi interi da 64 bit
>L'elemento $i$-esimo di un array si troverà alla locazione $br+8i$. Dove $br$ è l'indirizzo del primo elemento dell'array, $i$ è l'indice e $8$ dipende dall'indirizzamento della memoria.

>[!example]
>Leggere il contenuto dell'elemento `A[3]` e memorizzarlo in `s1`. L'indirizzo dell'array è contenuto nel registro `s2`: 
>```asm
>ld s1, 24(s2)
>```

>[!tip] Ordine della memoria fisica
>Una parola contiene più byte. I byte di una parola possono essere numerati da sinistra a destra e viceversa:
>- big endian, con i byte numerati da sinistra a destra
>- little endian, con i byte numerati da desta a sinistra
>
>![[Pasted image 20240918094131.png]]
>
>RISC-V usa little endian.

##### Istruzioni logiche
>[!tip] Istruzione `and`:
>Questa istruzione esegue l'and bit a bit tra i registri sorgente `rs1` e `rs2` e mette nel registro destinazione `rd` il risultato:
>```asm
>and rd, rs1, rs2 
>```

>[!tip] Istruzione `or`:
>Questa istruzione esegue l'or bit a bit tra i registri sorgente `rs1` e `rs2` e mette nel registro destinazione `rd` il risultato:
>```asm
>or rd, rs1, rs2 
>```

>[!tip] Istruzione `xor`:
>Questa istruzione esegue lo xor bit a bit tra i registri sorgente `rs1` e `rs2` e mette nel registro destinazione `rd` il risultato:
>```asm
>xor rd, rs1, rs2 
>```

>[!tip] Istruzione `slli`:
>Questa istruzione esegue un shift da sinistra dall'indirizzo sorgente `rs1` emette nel registro destinazione `rd` il risultato con un padding di `const` zeri:
>```asm
>slli rd, rs1, const12
>```

>[!tip] Istruzione `srli`:
>Questa istruzione esegue uno shift da destra dall'indirizzo sorgente `rs1` emette nel registro destinazione `rd` il risultato con un padding di `const` zeri:
>```asm
>srli rd, rs1, const12
>```

>[!tip] Istruzione `srai`:
>Questa istruzione esegue uno shift di `const` da destra dall'indirizzo sorgente `rs1` emette nel registro destinazione `rd` il risultato con estendendo il segno a sinistra:
>```asm
>srai rd, rs1, const12
>```

##### Istruzioni di salto

>[!tip] Istruzione `beq`
>Questa istruzione confronta i registri `rs1` e `rs2` e salta all'istruzione (relativa al `PC`) `ind_salto` se i due registri sono uguali:
>```asm
>beq rs1, rd2, ind_salto
>```

>[!tip] Istruzione `bne`
>Questa istruzione confronta i registri `rs1` e `rs2` e salta all'istruzione (relativa al `PC`) `ind_salto` se i due registri non sono uguali:
>```asm
>bne rs1, rd2, ind_salto
>```

>[!tip] Istruzioni simili
>Esistono anche le istruzioni:
>- `blt`: branch less than
>- `bge`: branch greater equal
>
>E le corrispondenti unsigned.

>[!tip] Istruzione`jal`
>Salto incondizionato relativo al `PC`, con salvataggio dell'indirizzo di rientro nel registro `rd`.
>```asm
>jal rd, spi20
>```

>[!tip] Istruzione `jalr`
>Chiamata al sottoprogramma relativa a registro `rs1` con salvataggio dell'indirizzo di rientro nel registro `rd`.
>```asm
>jalr rd, spi12(rs1)
>```

### Etichette
>[!note]
>Nonostante nelle istruzioni in linguaggio macchina si usino gli indirizzi, per un programmatore assembler non è conveniente, o talvolta possibile calcolarli. Si usano quindi delle etichette (label) che si associano a specifici punti del programma. L'assemblatore poi traduce le etichette nei corrispondenti indirizzi.

>[!example] Esempio `if`, `then`, `else`
>```asm
>IF:    bne s3, s4 ELSE   # got to else if i != j
>THEN:  add s0, s1, s2    # f = g + h
>        j ENDIF           # go to ENDIF
>ELSE:  sub s0, s1, s2    # f = g - h
>ENDIF: ...
>```

