>[!note]
>L'allocazione della memoria può essere suddivisa a $2$ livelli:
>- Allocazione a grana grossa: fornisce ai processi e al sistema le grosse porzioni di memoria sulle quali operare.
>- Allocazione a grana fine: alloca strutture piccole nelle porzioni gestite dalla grana grossa.
>  
>È importante tenere distinte l'allocazione di memoria virtuale da quella fisica: quando un processo esegue una `brk` il suo spazio virtual viene incrementato ma non viene allocata memoria fisica. Solo quando il processo va a scrivere nel nuovo spazio virtuale la necessaria memoria fisica viene allocata.

Nei sistemi Linux una certa quantità di memoria viene allocata inizialmente al SO e non viene mai deallocata. Le eventuali richieste di memoria dinamica da parte del SO stesso vengono soddisfatte con la massima priorità. Quando invece un processo richiede memoria, questa gli viene allocata senza particolari limitazioni.

Tutti i dati letti dal disco vengono conservati indefinitamente in memoria per poter essere eventualmente riutilizzati, tuttavia, quando la memoria disponibile scende sotto una soglia predefinita viene eseguita un'operazione di riduzione delle pagine occupate (Page Reclaiming).

>[!tip] Scaricamento di una pagina
>Se una pagina è stata letta da disco e non è mai stata modificata la pagina viene semplicemente resa disponibile. Se invece è stata modifica deve essere scritta su disco prima di essere resa disponibile.
>
>Per quanto riguarda alla deallocazione, si scaricano molte pagine di cache non utilizzate dai processi, se non è sufficiente. Alcune pagine dei processi vengono scaricate, se non è sufficiente. E in certi casi un processo può venire eliminato completamente.

Il file `/proc/meminfo` riporta maggiori informazioni sull'utilizzo della memoria. I buffers contengono informazioni associate a dispositivi di tipo "block device" e contengono informazioni relative al file system.

L'unità di base per l'allocazione di memoria è la pagina, ma Linux cerca di mantenere la memoria meno frammentata possibile. Nonostante questa scelta possa apparire inutile non lo è, questo perché la memoria è anche acceduta da DMA, è preferibile usare pagine contigue fisicamente per motivi legati sia alle cache che alla latenza di accesso alle RAM, e la rappresentazione della RAM libera risulta più compatta.

### Buddy system
>[!note]
>L'allocazione a blocchi di pagine contigue si basa sul seguente meccanismo.
>
>La memoria è suddivisa in grandi blocchi di pagine contigue, la cui dimensione è una potenza di $2$ detta ordine del blocco. Per ogni ordine esiste una lista che collega tutti i blocchi di quell'ordine. Le richieste di allocazione indicano l'ordine del blocco che desiderano.
>
>Se un blocco richiesto è disponibile questo viene allocato, altrimenti è diviso in $2$. I due nuovi blocchi sono detti buddies l'uno dell'altro, e questa relazione è conservata dalle strutture dati utilizzate per l'allocazione.
>
>Quando un blocco viene liberato il suo buddy viene analizzato e, se è libero, i due vengono riunificati, ricreando un blocco più grande.

>[!tip] Deallocazione dalla memoria fisica
>Fino a quando la quantità di RAM libera è sufficientemente grande Linux alloca la memoria senza svolgere particolari controlli. La memoria destinata ai processi viene rilasciata quando un processo termina, e la memoria destinata alle disk cache cresce sempre perché non esiste un criterio per stabilire se un dato su un disco non verrà più utilizzato.
>
>Quando la quantità di memoria scende sotto un livello critico interviene PFRA (Page Frame Reclaiming Algorithm), il cui obiettivo è riportare il numero di pagine libere ad un livello di dimensione al minimo `maxFree`.
>
>Il meccanismo utilizzato da PFRA per scegliere quali pagine liberare si basa su LRU, cioè sulla scelta delle pagine non utilizzate da più tempo.
>
>Definiamo `freePages` come il numero di pagine di memoria fisiche libere in un certo istante, `requiredPages` come il numero di pagine che vengono richieste per una certa attività da parte di un processo, `minFree` come il numero minimo di pagine libere sotto il quale non si vorrebbe scendere, e `maxFree` come il numero di pagine libere al quale PFRA tenta di riportare `freePages`.
>
>Il PFRA può essere invocato direttamente da parte di un processo che richiede `requiredPages` di memoria se $(\text{freePages}-\text{requiredPages})<\text{minFree}$.
>
>È inoltre invocato periodicamente tramite `kswapd`, una funzione attivata periodicamente dal SO che invoca PFRA se $\text{freePages}<\text{maxFree}$.
>
>Il PFRA determina il numero di pagine da liberare con l'algoritmo: $$\text{toFree}=\text{maxFree}-\text{freePages}+\text{requiredPages}$$

>[!tip] Liste LRU
>Esistono due liste globali, dette Least Recently Used List (LRU List) che collegano tutte le pagine appartenenti ai processi:
>- Active list: contiene tutte le pagine che sono state accedute recentemente e non possono essere scaricate.
>- Inactive list: contiene le pagine inattive da molto tempo che sono quindi candidate per essere scaricate.
>  
>Entrambe le liste tengono le pagine in ordine di invecchiamento, tramite spostamento delle pagine da una lista all'altra per tenere conto degli accessi a memoria.
>
>Dato che l'`x64` non tiene traccia del numero di accessi alla memoria, Linux realizza un'approssimazione basata sul bit di accesso A presente nella PTE della pagina, che viene posto a $1$ da `x64` ogni volta che la pagina viene acceduta, e viene azzerato periodicamente dal sistema operativo.
>
>Ad ogni pagina viene inoltre associato un flag `ref`, oltre al bit di accesso A settato dall'hardware. Questo serve a raddoppiare il numero di accessi necessari per spostare una pagina da una lista all'altra.
>
>Definiamo un'algoritmo semplificato per vedere come si spostano le due pagine:
>
>Per le pagine nell'active list, se $A=1$ azzera $A$, e se $\text{ref}=1$ sposta P in testa alla active list, altrimenti pone $\text{ref}$ a $0$. Viceversa se $A=0$, se $\text{ref}=1$ lo pone a $0$, e se $\text{ref}=0$ sposta P in testa alla inactive list con $\text{ref}=1$.
>
>Per le pagine nell'inactive list (escluse le pagine precedentemente spostate), se $A=1$ azzera $A$, e se $\text{ref}=1$ sposta P in coda alla active list con $\text{ref=0}$, altrimenti pone $\text{ref}$ a $1$. Viceversa se $A=0$, se $\text{ref}=1$ lo pone a $0$, e se $\text{ref}=0$ sposta P in coda alla inactive list.
>
>
>Per quanto riguarda lo scaricamento prima si scaricano le pagine appartenenti alla page cache non più utilizzate da nessun processo, in ordine di NPF. Successivamente si parte dalla coda della lista inactive, tenendo conto della eventuale condivisione delle pagine. Una pagina virtuale è ritenuta scaricabile solo se tutte le pagine condivise sono nella inactive list in posizioni successive.

### Meccanismo di swapping
>[!note]
>È necessario che sia disponibile una Swap Area sul disco, e pertanto esiste lo Swap File, una sequenza di Page slots, ognuna di dimensioni pari ad una pagina, identificata da un numero (SWPN).
>
>Ogni swap area è identificata da un descrittore, che contiene un contatore per ogni page slot (`swap_map_counter`), che indica il numero di pagine virtuali condivise dalla pagina fisica copiata nel Page Slot.

Quando PFRA chiede di scaricare una pagina fisica (`swap_out`) viene allocato un page slot. La pagina fisica viene copiata nel page slot e liberata, e il `swap_map_counter` viene impostato al numero di pagine virtuali che condividono la pagina fisica. In ogni elemento della tabella delle pagine che condivideva la pagina fisica viene registrato il SWPN del page slot al posto di NPF e il bit di presenza viene azzerato.

Quando un processo poi accede ad una pagina virtuale swappata si verifica un page fault. Quindi il page fault handler attiva la procedura di caricamento (`swap_in`) e viene allocata una pagina fisica in memoria.

Il page slot indicato nella tabella delle pagine in corrispondenza della pagina virtuale cercata viene copiato nella pagina fisica, e la tabella delle pagine viene aggiornata inserendo il NPF della pagina fisica allocata al posto di SWPN del page slot.

>[!tip] `swap_out`
>Per ogni pagina da scaricare `swap_out` si determina, in base al dirty bit della pagina se salvarla su disco. Se questa è mappata su file e shared si salva su file, se invece è anonima si salva nella swap area.
>
>Una pagina fisica condivisa è dirty se il suo descrittore è marcato D a seguito di un TLB flush, oppure una delle pagine virtuali condivise sulla stessa pagina fisica è contenuta nel TLB ed è marcata dirty.

>[!tip] `swap_in`
>Quando una pagina viene ricaricata con lo `swap_in` in Linux non cancella la pagina dalla swap area, che rappresenta una backing store per quella pagina. L'informazione è mantenuta nella swap cache, cioè l'insieme delle pagine che sono state rilette dalla swap area e non sono state modificate, assieme ad alcune strutture ausiliarie che permettono di gestirla come se tali pagine fossero mappate sulla swap area.
>
>Consideriamo le pagine anonime e private: quando si esegue uno `swap_in` la pagina viene copiata in memoria fisica, la copia nella swap area non viene eliminata, la pagina letta in memoria è marcata in sola lettura. Nella swap cache index viene inserito un descrittore che contiene i riferimenti alla pagina fisica e al page slot. Se la pagina fisica viene solo letta, quando la pagina viene nuovamente scaricata non viene riscritta su disco, ma se viene scritta, si verifica un page fault e la pagina diventa private non appartiene più alla swap area.
>
>Per quanto riguarda le liste LRU il comportamento è come segue:
>- La NPV che è stata letta o scritta, cioè quella che ha causato lo `swap_in`, viene inserita in testa alla active con $\text{ref}=1$.
>- Eventuali altre NPV condivise all'interno della stessa pagina vengono poste in coda alla inactive con $\text{ref}=0$.

L'allocazione/deallocazione della memoria interferisce con i meccanismi di scheduling. Supponiamo che un processo $Q$ a bassa priorità sia una forte consumatore di memoria, e che contemporaneamente sia in funzione un processo $P$ molto interattivo. Potrebbe accadere che, mentre il processo $P$ è in attesa, il processo $Q$ carichi progressivamente tutte le sue pagine forzando fuori memoria le pagine del processo $P$.
Quando $P$ viene risvegliato e entra rapidamente in esecuzione grazie ai suoi elevati diritti di esecuzione si verificherà un ritardo dovuto al caricamento delle pagine che erano state caricate.

>[!tip] Out Of Memory Killer
>In sistemi molto carichi il PFRA può non riuscire a risolvere la situazione. In tal caso come estrema ratio deve invocare il OOMK, che seleziona un processo e lo elimina.
>
>La scelta del processo da eliminare è fatta in base ai seguenti criteri:
>- Molte pagine occupate.
>- Poco lavoro svolto.
>- Priorità bassa.
>- Senza privilegi di root.
>- Non gestisca componenti hardware.

I meccanismi adottati da Linux sono complessi e il loro comportamento è difficile da predire in diverse condizioni di carico. Non è quindi escluso che in certe situazioni si verifichi un fenomeno detto Thrashing, cioè il sistema continua a deallocare pagine che vengono nuovamente richieste dai processi.