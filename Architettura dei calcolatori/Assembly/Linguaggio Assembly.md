>[!note]
>Il linguaggio assembly è un linguaggio specifico del particolare processore più primitivo rispetto ai linguaggi ad alto livello. È molto restrittivo con il suo set di operazioni.

>[!tip] Stored Program
>I programmi sono sequenze di istruzioni macchina rappresentate in binario, vengono caricati in memoria esattamente come i dati.
>
>Un programma è una sequenza di codici numerici binari, di cui alcuni vengono interpretati dalla CPU come istruzioni, mentre altri sono dati.

In RISC-V la memoria è indirizzata a byte.

### Processo di compilazione
>[!note]
>Il passaggio dal codice sorgente all'esecuzione del programma si articola in tre fasi:
>1. Compilazione (tramite un assembler)
>2. Linking (tramite un linker)
>3. Caricamento ed esecuzione

### Caratteristiche di un'architettura di un set di istruzioni
>[!note]
>Un architettura di un set di istruzioni è composto da:
>- Registri
>- Istruzioni macchina e tipi di formati (codifica, operazioni, indirizzamento)
>- Modello di memoria

### Registri
>[!note]
>In RISC-V esistono 32 registri di uso generale, sono numerati da $0$ a $31$ (bastano quindi $5\text{ bit}$ per codificarli). e sono indicati come `x0`-`x31`. Inoltre hanno anche dei nomi simbolici convenzionali. Hanno una lunghezza di $64\text{ bit}$ e possono solo contenere valori interi (I valori decimali hanno dei registri specifici).

>[!tip] Registri referenziabili
>| Registro | nome ABI | Utilizzo |
>| - | - | - | 
>| `x0` | `zero` | Contiene solo una costante $0$ non modificabile |
>| `x1` | `ra` | Indirizzo di rientro da funzione |
>| `x2` | `sp` | Puntatore alla cima della pila |
>| `x3` | `gp` | Puntatore all'area dati globale |
>| `x4` | `tp` | Puntatore a thread |
>| `x5`-`x7` | `t0`-`t2` | Valori temporanei |
>| `x8` | `s0`/`fp` | Variabile locale di funzione o frame pointer (se in uso) |
>| `x9` | `s1` | Variabile locale di funzione |
>| `x10` - `x11` | `a0`-`a1` | Valore restituito da funzione |
>| `x12`-`x17`| `a2`-`a7` | Argomenti di ingresso a funzione (max $6$ argomenti, gli eccedenti in pila) |
>| `x18`-`x27` | `s2`-`s11` | Variabili locali di funzione (max $10$ o $11$ variabili, le eccedenti in pila) |
>| `x28`-`x31` | `t3`-`t6` | Valori temporanei |

>[!tip] Registri non referenziabili
>
>|Nome ABI| Utilizzo |
>| - | - |
>| `pc` | Program Counter |
>| `hi`, `lo` | Registri per moltiplicazioni e divisioni |

### Istruzioni
>[!note]
>Le istruzioni comprese nel linguaggio macchina di qualunque computer cono classificabili in quattro categorie:
>- Aritmetico/Logiche
>- Load/Store
>- Branch/Jump
>- I/O

### Linguaggio macchina
>[!note]
>Le istruzioni in linguaggio assembly vanno tradotte in linguaggio macchina (cioè sequenze di $0$ e $1$) per essere eseguite.