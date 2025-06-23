>[!note]
>Per elaborare informazioni occorre codificarle, nel nostro caso usando segnali. I segnali devono essere elaborati in modo opportuno tramite dispositivi di elaborazione.
>
>In un sistema digitale le informazioni vengono rappresentate, elaborate e trasmesse mediante grandezze fisiche che si considerano assumere solo valori discreti. Ogni valore è associato a una cifra della rappresentazione.
>
>In particolare, il segnale binario è una grandezza che può assumere due valori distinti, indicati convenzionalmente con $0$ e $1$.

Per elaborare un segnale binario si usano due classi di dispositivi di elaborazione:
- Reti combinatorie: senza retroazioni, quindi ogni segnale nella rete non può influenzare se stesso.
- Reti sequenziali: reti con retroazioni, quindi lo stato dipende dalla sequenza di ingressi applicata nel passato.

### Algebra di Commutazione
>[!note]
>L'algebra di commutazione è un derivato dell'algebra di Boole, e consente di descrivere matematicamente i circuiti ditali. Definisce le espressioni logiche che descrivono il comportamento del circuito da realizzare nella forma: $$U=f(I)$$
>A partire dalle equazioni logiche è possibile derivare la realizzazione circuitale.
>
>I componenti dell'algebra di Boole sono le variabili di commutazione, gli operatori fondamentali e le proprietà degli operatori logici.
>
>Una variabile di commutazione corrisponde al singolo bit di informazione rappresentata e elaborata, mentre gli operatori fondamentali sono negazione ($!A$ o $\smallsetminus A$), somma logica ($A + B$) e prodotto logico ($A\cdot B$).

>[!tip] Proprietà degli operatori logici
>
>| Legge | AND | OR |
>| - | - | - |
>| Identità | $1\cdot A = A$ | $0+A=A$ |
>| Elemento nullo | $0\cdot A=A$ | $1 + A=A$ |
>| Idempotenza | $A\cdot A=A$ | $A + A=A$ |
>| Inverso | $A\cdot !A=0$ | $A+!A=1$ |
>| Commutativa | $A\cdot B= B\cdot A$ | $A+B=B+A$ |
>| Associativa | $(A\cdot B)\cdot C= A\cdot( B\cdot C)$ | $(A+B)+C=A+(B+C)$ |
>| Distributiva | $A\cdot(B+C)=AB+AC$ | $A+B\cdot C=(A+B)\cdot(A+C)$ |
>| Assorbimento | $A\cdot (A+B)=A$ | $A+A\cdot B=A$ |
>| De Morgan | $!(A\cdot B)=!A+!B$ | $!(A+B)=!A\cdot !B$ |

### Funzioni combinatorie
>[!note]
>Una funzione combinatoria corrisponde ad un'espressione booleana, contenente una o più variabili booleane e gli operatori booleani `AND`, `OR` e `NOT`.

Per specificare il comportamento di una funzione combinatoria è possibile specificare, per ogni possibile configurazione degli ingressi, il valore dell'uscita, facendo una tabella delle verità.

Una tabella delle verità di una funzione a $n$ ingressi ha $2^{n}$ righe, che corrispondono a tutte le possibili configurazioni di ingresso.
