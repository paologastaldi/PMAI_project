# Processi Musicali Automatici ed Interattivi

### Gastaldi Paolo

| Progetto esame del 18-09-2020 | |
| :- | :-: |
| Versione | 1.0.0 |
| Data | 12-09-2020 |

## Indice

+ [Abstract](#abstract)
+ [Obiettivi](#obiettivi)
+ [Risultati](#risultati)
+ [Dettagli tecnici](#dettagli-tecnici)
    - [Funzionamento](#funzionamento)
    - [Dipendenze](#dipendenze)
    - [Difficoltà riscontrate](#difficoltà-riscontrate)

## Abstract

Troppo spesso quando ascoltiamo un discorso ci facciamo influenzare da chi lo stia dicendo, perdendo il significato delle parole che dice. Distaccare le due parti sembra impossibile, a meno che non intervengano elementi di contrasto tali da spostarci l'attenzione su cosa stia avvenendo. Cosa succederebbe se un bellissimo discorso venisse interrotto da una conversazione con termini terribili e crudeli? E se invece le parole del peggiore degli esseri umani venissero rimpiazzate da espressioni di pace e fratellanza?

In questo progetto ho provato a cercare i punti di congiunzione tra discorsi differenti ed influenzarli a vicenda, facendone interagire uno con le parole dell'altro.

## Obiettivi

Gli obiettivi del progetto sono di:

- estrapolare le sillabe da un discorso
- analizzare ogni sillaba e riuscire a trovarne dei parametri caratterizzanti tali da descriverla nella maniera più accurata possibile
- ricreare un discorso "in diretta" con i frammenti di un altro discorso precedentemente analizzati

## Risultati

Nella sottocartella ```examples``` è possibile trovare dei file  di esempio utilizzati per la taratura e il test del sistema.

In particolare, i file che terminano con *_model* sono dei modelli precedentemente generati dal sistema e che possono essere caricati dalla patch con i pulsanti *load* e successivamente *train* (per l'aggiornamento dei buffer interni).

I file che terminano con *_results* sono delle registrazioni ottenuti dal sistema. A volte sono del singolo risultato generato, a volte alternano l'audio originale con la parte generata.

## Dettagli tecnici

### Funzionamento

La patch (o meglio, l'insieme di patch) è basato su 4 parti principali, di cui 2 sono componenti di base e 2 invece di più alto livello.

Il primo step è il riconoscimento delle sillabe. Per farlo nella patch ```glottal_core``` viene utilizzato il sistema della ZFF, Zero Frequency Filter, una tecnica per individuare i movimenti gutturali basata su un filtro risonante a (circa) 0 Hz (per maggiori dettagli: Yegnanarayana & Gangashetty, 2011).

Il frammento della sillaba così estratto viene quindi analizzato dalla patch ```extractor``` (contenuta in ```extractor_usystem```). L'analisi consiste nell'estrarre 5 descrittori che vadano a caratterizzare la sillaba:

- total energy: per individuare la vocale associata
- perceptual spectral centroid: brillantezza
- perceptual spectral decrease: coda della sillaba
- perceptual crest:
- perceptual skewness: rapporto tra l'attacco e la decadenza della sillaba

Questi valori vengono quindi riscalati per occupare tutto il range di valori a loro disposizione.

Queste 2 patch appena descritte vengono utilizzate per l'analisi e il training del sistema nella patch ```syllable_train_engine```, che permette di analizzare una serie di file pre-caricati in un apposito container MuBu. Per ogni file verranno estratti i riferimenti temporali di ogni singola sillaba e i corrispettivi valori dell'analisi. Al termine dell'analisi viene creato il modello (basato su un algoritmo knn) per un sistema di ricerca della sillaba più vicina.

Per l'esecuzione invece si utilizza la patch ```syllable_play_engine```, che permette di rilevare ed analizzare "in diretta" le sillabe contenute nel segnale in ingresso, andare a cercare per ciascuna la sillaba più prossima dal set di training e riprodurla. 

### Dipendenze

Elenco dei pacchetti e programmi aggiuntivi esterni a Max necessari per il funzionamento della patch.

| Dipendenza | Versione |
| :-: | :-: |
| MuBu| 1.9.14 |

### Difficoltà riscontrate

La taratura del sistema è molto delicata e fine. Per cercare i parametri che il sistema caricare di default sono stati effettuati dei test con il file *practicing_monologue*, che come voce recitata dispone di un'ampia gamma di dinamica all'interno del discorso.

Il riconoscimento delle sillabe è basato su un filtro risonante che se tarato troppo "stretto" a 0Hz e con un Q molto selettivo rischia di andare facilmente in feedback. Per questo e per permettere di rilevare anche sillabe sussurrate è stato tarato con parametri più "larghi".
Di contro, questa taratura comporta che il sistema rilevi molte più sillabe spurie dovute alle oscillazioni, soprattutto se vengono pronunciate da una voce ad alto volume o urlata.