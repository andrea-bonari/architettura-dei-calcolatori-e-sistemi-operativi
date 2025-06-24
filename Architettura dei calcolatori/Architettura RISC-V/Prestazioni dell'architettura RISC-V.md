>[!note]
>Nel confronto delle prestazioni di due calcolatori realizzati come implementazioni diverse della stessa architettura ISA, si possono fare considerazioni che non sono applicabili con diversa ISA perché in questo caso:
>- I due programmi eseguibili da confrontare possono essere identici.
>- Le istruzioni eseguite per una certa configurazione di ingresso sono esattamente le stesse.
>
>Definiamo quindi dei parametri generici di prestazione. Sia $P$ una prova, cioè un programma da eseguire che in generale contiene cicli: $$T(P)=\text{cicli}(P)\cdot\text{periodo di clock}= \frac{\text{cicli}(P)}{\text{frequenza di clock}}$$
>Definiamo inoltre $\text{NI}(P)$ come il numero di istruzioni eseguite durante la prova, e $\text{CPI}(P)$ come i cicli di clock per istruzione: $$T(P)=\frac{\text{cicli}(P)}{\text{NI}(P)}= \text{NI}(P)\cdot \frac{\text{CPI}(P)}{\text{frequenza di clock}}$$

### Prestazioni di un processore pipeline con memoria cache
>[!note]
>In una pipeline senza stalli di alcun tipo si ha sempre $\text{CPI}(P)=1$. Se la pipeline presenta stalli introdotti per risolvere conflitti si ha: $$\begin{align*}
>\text{stalli}(P)&= \text{numero di cicli di stallo nella prova }P\\
>\text{cicli}(P)&= \text{NI}(P)+\text{stalli}(P)+4
>\end{align*}$$
>
>Se la memoria cache è ideale, allora essa influisce sulle prestazioni del processore. Se non lo è allora definiamo $\text{stalliM}(P)$ come il numero di stalli causati dalla memoria cache durante la prova $P$, e quindi si ha che: $$T(P)= \frac{\text{cicli}(P)+\text{stalliM}(P)}{\text{frequenza di clock}}$$
>Potremmo definire con più precisione: $$\text{stalliM}(P)=\text{numero di accessi alla memoria}(P)\cdot m\cdot M$$
>Con $m$ miss rate e $M=\text{dimensione del blocco (in parole)}\cdot\text{tempo di accesso a una parola in memoria centrale}$.



