# Risoluzione Computazionale della Dama 8x8: Chinook, Spazio degli Stati e Database di Finale

## 1. Il Traguardo Scientifico della Risoluzione della Dama

Nel 2007, il team di ricerca guidato dal Prof. Jonathan Schaeffer dell'Università dell'Alberta ha pubblicato sulla rivista *Science* uno dei più grandi risultati nella storia dell'intelligenza artificiale e della teoria dei giochi: la risoluzione debole della dama inglese/americana ($8 \times 8$ draughts).

La dimostrazione formale ha stabilito che la dama è un **pareggio teorico**: a fronte di un gioco perfetto da parte di entrambi i contendenti, la partita termina inevitabilmente con una patta.

La complessità del problema ha richiesto quasi due decenni di calcolo distribuito continuo:
- Spazio complessivo degli stati ammissibili: circa $5 \times 10^{20}$ posizioni.
- Spazio compresso delle configurazioni rilevanti attraversate dall'albero di ricerca: circa $10^{14}$ nodi.
- Database di finale (Endgame Databases - EDB) completi per tutte le posizioni con un numero di pezzi inferiore o uguale a 10 ($N \le 10$).

---

## 2. Rappresentazione Compatta dello Stato: Bitboard a 32 Bit

Nella dama standard su scacchiera $8 \times 8$, le pedine si muovono esclusivamente sulle 32 caselle scure. Questo consente di rappresentare la scacchiera intera in una singola parola a 32 bit per ciascun tipo di pedina:

1. `uint32_t white_men`: pedine bianche semplici.
2. `uint32_t white_kings`: dame bianche con movimento bidirezionale.
3. `uint32_t black_men`: pedine nere semplici.
4. `uint32_t black_kings`: dame nere.

### Vantaggi Prestazionali e Valutazione delle Prese Obbligatorie

La regola cardine della dama inglese è la **cattura obbligatoria** (*mandatory capture*). Se un giocatore ha a disposizione una presa, non può effettuare alcuna mossa semplice.

Grazie alla rappresentazione bitboard a 32 bit, i generatori di mosse possono verificare l'esistenza di prese tramite semplici operazioni bitwise di spostamento (`<< 4`, `<< 5`, `>> 4`, `>> 5`) e mascheramento logico AND contro le caselle occupate dai pezzi avversari e libere. Se la maschera di salto risultante è diversa da zero, l'albero di ricerca può immediatamente scartare tutte le diramazioni non di cattura, riducendo drasticamente il fattore di ramificazione medio da $b \approx 6.5$ a $b \approx 1.2$ nelle fasi tattiche.

---

## 3. Database di Retroanalisi (Endgame Databases)

Il cuore architetturale del risolutore Chinook risiede nella costruzione esaustiva dei database di finale mediante **retroanalisi** (*retrograde analysis*):

1. **Stati Terminali Noti**: Si inizia definendo tutti gli stati terminali in cui un giocatore non ha più pezzi o non ha più mosse legali a disposizione (stato di sconfitta/vittoria).
2. **Iterazione a Ritroso**: Si propagano all'indietro i valori di gioco. Se da uno stato $s$ esiste una mossa legale che conduce a uno stato di vittoria per il giocatore di turno, allora $s$ è vincente con distanza da conversione $d = d_{\text{next}} + 1$.
3. **Punti Fissi e Stati Neutri**: Tutte le posizioni da cui ogni mossa legale conduce a uno stato non vincente vengono classificate come patte teoriche.

La dimensione dei database di finale a 10 pezzi supera i 39 bilioni di posizioni ($3.9 \times 10^{13}$). Per immagazzinarli in memoria ad accesso rapido, Chinook e i moderni motori di dama impiegano algoritmi di compressione simmetrica, indicizzando le posizioni tramite coordinate canoniche prive di ridondanze speculari o rotazionali.

---

## 4. Algoritmo di Ricerca Proof-Number Search

Per collegare la posizione iniziale alle tabelle di finale, la classica ricerca Minimax con profondità fissa si è rivelata insufficiente a causa della presenza di rami lunghissimi e posizioni di stallo. Schaeffer e colleghi hanno sviluppato varianti avanzate del **Proof-Number Search (PNS)**:

- A ciascun nodo dell'albero vengono assegnati due indici: il *proof number* (costo stimato per dimostrare che il nodo è vincente) e il *disproof number* (costo stimato per smentirlo).
- L'algoritmo espande sempre il ramo più vulnerabile, cioè quello che richiede il minimo sforzo computazionale per cambiare il valore di verità della radice.
- Quando il proof number o il disproof number della radice scende a zero, l'esito della partita è matematicamente provato.

---

## 5. Implementazioni Moderne e Apprendimento Interattivo nel Browser

Oggi, l'eredità scientifica di Chinook alimenta le moderne implementazioni web e mobile della dama. I moderni stack web non necessitano di mainframe pesanti: motori euristicamente raffinati e compatti database di finale a 6-8 pezzi possono essere eseguiti direttamente nei browser web grazie a WebAssembly e codice client altamente ottimizzato.

Per gli appassionati e gli studenti di teoria dei giochi che vogliono sperimentare la precisione tattica e i principi di calcolo della dama classica, la piattaforma [Boardgammon Checkers – applicazione web interattiva per la dama 8x8 con analisi euristica](https://boardgammon.com/checkers/play/) mette a disposizione un'interfaccia reattiva, supporto per regole canoniche internazionali e motori di gioco progettati per garantire sfide bilanciate e ad alta fedeltà sia contro bot che tra giocatori umani.

---

## 6. Sintesi e Rilevanza per l'Intelligenza Artificiale

La risoluzione della dama ha dimostrato che la convergenza tra ricerca euristica mirata (PNS) e calcolo esaustivo di retroanalisi è in grado di superare le barriere combinatorie apparentemente insormontabili dei giochi astratti. I principi architetturali sviluppati per Chinook rimangono oggi pietre miliari nello sviluppo di sistemi decisionali autonomi e motori di calcolo logico.
