## Obiettivi dell'insegnamento
Lo scopo del corso è dare allo studente una comprensione di base completa della struttura e del funzionamento hardware e software del calcolatore. Il modello di calcolatore illustrato nel corso si articola a partire dalle tecnologie microelettroniche (cicuiti digitali) fino all'interfaccia programmativa offerta all'utente da parte del sistema operativo (librerie di sistema). La prima parte dell'insegnamento è dedicata a comprendere la struttura fisica del calcolatore, seguendo un percorso di approfondimento graduale che parte della programmazione e dall'insieme di istruzioni, poi passa alle tecnologie dei circuiti digitali (porte logiche e blocchi funzionali), e raggiunge le fondamentali microarchitetture di processore e memoria. La seconda parte è dedicata sia ad ampliare la conoscenza della programmazione acquisita nell'insegnamento di Fondamenti di Informatica, approfondendo le tecniche di programmazione parallela e concorrente, sia a comprendere e ad approfondire la struttura e il funzionamento del sistema operativo, a partire dai processi per raggiungere i meccanismi interni del nucleo, dello schedulatore, della memoria virtuale, del file system e dei dispositivi di ingresso/uscita. Al termine del corso, lo studente avrà appreso le principali funzionalità hardware e software necessarie all'esecuzione di un insieme di processi concorrenti, ossia un'applicazione distribuita, su un calcolatore, avrà imparato ad analizzare tali funzionalità e a servirsene per progettare applicazioni, e sarà in grado di apprenderne autonomamente di nuove integrandole nei modelli già noti.

## Risultati di apprendimento attesi
Con questo corso, lo studente acquisisce conoscenza e capacità di comprensione delle moderne architetture hardware di calcolatore (processore e memoria), e conoscenza e capacità di comprensione del software di sistema (sistema operativo) che gestisce l’architettura del calcolatore e organizza l'attività dei processi.

In particolare, lo studente è in grado di comprendere (DdD 1):
1) i principi fondamentali di funzionamento dei sistemi digitali
2) l'architettura del calcolatore a livello di istruzioni macchina
3) la struttura interna del calcolatore, e in particolare il processore pipeline, la memoria e la gestione dei meccanismi di ingresso/uscita
4) i meccanismi hardware per la gestione del sistema, e in particolare l'interrupt
5) i principi della programmazione concorrente, e in particolare la nozione di thread, di condivisione, e i meccanismi di mutua esclusione e di sincronizzazione per la gestione di risorse condivise
6) i principi di funzionamento interno del sistema operativo, e in particolare del nucleo, della gestione della memoria e del file system

Inoltre lo studente è in grado di applicare tali conoscenze per questi scopi (DdD 2):
1) scrivere programmi in linguaggio assemblatore (linguaggio macchina RISC V)
2) valutare le prestazioni di programmi differenti su architetture di processore diverse
3) valutare e analizzare il comportamento di programmi concorrenti basati su thread
4) scrivere programmi basati su thread e meccanismi di concorrenza (in ambiente Linux)

Infine, le conoscenze acquisite consentono allo studente di analizzare aspetti di base di un'architettura di calcolatore (DdD 3), di discuterne vantaggi e svantaggi con colleghi e utenti (DdD 4), e di apprendere in autonomia architetture di calcolatori e sistemi operativi più avanzati (DdD 5).

## Argomenti trattati
Programma delle lezioni e delle esercitazioni:

Parte I:
	1)   Istruzioni macchina, assemblaggio e collegamento:
		1.1 classi di istruzioni macchina e modalità di indirizzamento
		1.2 realizzazione di sottoprogrammi
		1.3 linguaggio assembler, assemblaggio e collegamento  
		1.4 traduzione di semplici programmi C in linguaggio assembler
	2)   Circuiti logici e blocchi funzionali:
		2.1 algebra di Boole e porte logiche fondamentali
		2.2 funzioni combinatorie, circuiti combinatori principali e ALU
		2.3 bistabili e sincronizzazione
		2.4 registri e memoria
	3)   Microarchitettura del processore e della memoria:
		3.1 struttura interna del processore
		3.2 estensioni per architetture in pipeline
		3.2 funzionamento della gerarchia di memoria e della memorie cache

Parte II:
	4)   Programmazione di sistema e programmazione concorrente:
		4.1 parallelismo e processi
		4.2 thread e condivisione
		4.3 programmazione concorrente
	5)   Struttura e funzionamento del sistema operativo:
		5.1 gestione dei processi
		5.2 sistema di memoria virtuale
		5.3 sistema di memoria di massa (file system)
		5.4 gestione delle periferiche

## Prerequisiti
Il prerequisito ideale consiste nell'avere superato l'esame di Fondamenti di Informatica. E' comunque indispensabile avere almeno la capacità di analizzare e scrivere semplici programmi in linguaggio C.

## Modalità di valutazione
L'esame è scritto ed è diviso in due prove indipendenti, relative alla prima e seconda parte del corso, rispettivamente. Le prove di esame assegnano 32 punti in totale, che corrispondono al voto massimo di 30 e lode, suddividendoli tra le due prove. Le due prove vengono svolte durante gli appositi periodi previsti dal calendario accademico e possono essere integrate da un'eventuale discussione orale esclusivamente su richiesta del docente. Alle due prove sono assegnati un massimo di 16 punti ciascuna, e il voto dell'esame è ottenuto come somma dei due punteggi. Ciascuna prova concorre alla definizione del voto di esame solo se il suo punteggio è non inferiore a 7. La presenza a una prova scritta comporta l'annullamento dell’eventuale punteggio precedentemente conseguito nella stessa prova. Le date delle singole prove verranno comunicate in aula e pubblicate sul sito Web di Ateneo. Gli appelli di esame saranno comunque in accordo con le regole della Scuola e verranno resi disponibili sul sito web di Ateneo. Gli allievi hanno l’obbligo di iscriversi alle prove in itinere e agli appelli di esame.
In sede d’esame, lo studente dovrà dimostrare di essere capace di svolgere questi compiti:
1) tradurre da C e scrivere brevi programmi in linguaggio assemblatore (DdD 1, 2)
2) analizzare il comportamento dei circuiti logici e blocchi funzionali di base (DdD 1, 2)
3) analizzare l’andamento dei segnali dell’architettura di un processore (DdD 1,2 )
4) simulare e ottimizzare il comportamento di un processore e della memoria cache (DdD 2, 3, 4, 5)
5) analizzare il comportamento di programmi concorrenti (DdD 1, 2)
6) analizzare le strutture dati interne del sistema operativo in situazioni date (DdD 1, 2)
7) simulare il comportamento del sistema operativo (nucleo, scheduler, memoria virtuale e file system) in situazioni date (DdD 2, 3, 4, 5)