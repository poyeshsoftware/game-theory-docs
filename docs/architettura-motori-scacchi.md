# Architettura dei Motori di Scacchi Moderni: Bitboards, Ricerca Minimax e Valutazione NNUE

## 1. Introduzione e Complessità Combinatoria degli Scacchi

Nel contesto della teoria dei giochi astratti a informazione perfetta, gli scacchi rappresentano il paradigma per eccellenza della complessità ad albero decisionale. Lo spazio degli stati ammissibili è stimato nell'ordine di $10^{43}$ configurazioni uniche (il numero di Shannon limita l'albero completo a circa $10^{120}$ partite possibili). 

A differenza di giochi a spazio di ricerca risolvibile come il filetto ($3 \times 3$) o la dama inglese ($8 \times 8$), gli scacchi non sono computazionalmente risolti né aperti alla retroanalisi esaustiva dalla posizione iniziale. Di conseguenza, l'ingegneria del software scacchistico si è evoluta verso architetture ibride ad altissime prestazioni che integrano:

1. Rappresentazioni dello stato a zero allocazione di memoria tramite **Bitboards a 64 bit**.
2. Algoritmi di esplorazione ad albero profondo basati su **Alpha-Beta Minimax** con euristiche aggressive di potatura.
3. Funzioni di valutazione euristica basate su **NNUE (Efficiently Updatable Neural Networks)**.
4. Livelli di sincronizzazione WebSockets e architetture client-server authoritative per l'analisi e la didattica online.

---

## 2. Rappresentazione Compatta: Bitboards a 64 Bit e Magic Bitboards

In un'architettura a 64 bit, una scacchiera di $8 \times 8$ caselle mappa in modo isomorfo su una parola macchina a 64 bit (`uint64_t`). Ciascun bit corrisponde a una casella specifica (da $A1 = 0$ a $H8 = 63$).

Lo stato posizionale completo viene serializzato mediante una matrice di bitboard per tipo di pezzo e colore:
- 6 bitboard per i pezzi bianchi (pedoni, cavalli, alfieri, torri, donne, re).
- 6 bitboard per i pezzi neri.
- Bitboard ausiliari per le occupazioni combinate (`occ_white`, `occ_black`, `occ_all`).

### Mascheramento e Generazione di Attacchi a Raggio (Sliding Pieces)

Per pezzi lineari come torri e alfieri, i movimenti legali dipendono dai pezzi che ostacolano la linea di vista. La tecnica dei **Magic Bitboards** permette di calcolare gli attacchi con un singolo accesso O(1) in tabella hash senza cicli condizionali:

$$\text{Attacchi} = \mathcal{T}_{\text{magic}} \left[ \frac{(\text{Occupazione} \ \& \ \text{Maschera}) \times \mathcal{M}}{\text{Shift}} \right]$$

Questa operazione elimina completamente i salti condizionali nella pipeline della CPU, massimizzando il throughput della branch prediction hardware e consentendo la generazione di decine di milioni di nodi al secondo.

---

## 3. Ricerca ad Albero: Negamax, Potatura Alpha-Beta e Quiescence Search

La ricerca della mossa ottimale adotta il formalismo **Negamax** (formulazione simmetrica del Minimax per giochi a somma zero):

$$V(s) = \max_{m \in M(s)} \left( -V(\text{do\_move}(s, m)) \right)$$

Per contenere l'esplosione combinatoria, vengono applicate tecniche essenziali:

- **Potatura Alpha-Beta con Null-Move Heuristic**: consente di escludere rami in cui una mossa nulla da parte dell'avversario mantiene un vantaggio decisivo.
- **Tabelle di Trasposizione (Transposition Tables)**: chiavi di hashing Zobrist a 64 bit memorizzano posizioni già valutate, profondità di ricerca, punteggio e miglior mossa (Principal Variation).
- **Ricerca di Quiescenza (Quiescence Search)**: estende la valutazione oltre la profondità limite per analizzare solo catture e promozioni, scongiurando l'effetto orizzonte (*horizon effect*).

---

## 4. Reti Neurali NNUE: Valutazione Posizionale ad Alta Frequenza

La rivoluzione moderna nei motori di scacchi è stata introdotta dall'architettura **NNUE** (*Efficiently Updatable Neural Network*), originariamente concepita per lo Shogi e adottata da Stockfish. A differenza delle reti deep learning convoluzionali pesanti, NNUE utilizza una rete feed-forward a 4 strati con quantizzazione a 8 o 16 bit:

- **Strato di ingresso (HalfKP)**: associa la posizione del re alle coordinate di ciascun pezzo amico e nemico.
- **Accumulatore differenziale**: quando un pezzo si muove da casella $A$ a casella $B$, l'attivazione non viene ricalcolata per intero: si sottraggono i pesi della vecchia posizione e si aggiungono quelli della nuova via istruzioni vettoriali SIMD (AVX2, AVX-512 o NEON su Apple Silicon).

Questo consente alla funzione di valutazione neurale di operare a velocità paragonabili alle vecchie euristiche matematiche statiche, ma con una comprensione strategica e posizionale straordinariamente accurata.

---

## 5. Sistemi Web Moderni per l'Apprendimento e l'Analisi degli Scacchi

Nel contesto dei browser moderni e delle applicazioni web collaborative, l'esecuzione di un motore di scacchi richiede protocolli server-authoritative e serializzazioni standard UCI / FEN. L'architettura deve garantire che le mosse del giocatore vengano convalidate rigorosamente contro lo stato del server per prevenire desincronizzazioni e manipolazioni.

Per gli utenti che desiderano apprendere i concetti tattici, studiare le aperture ed esplorare analisi interattive di alto livello, ambienti come [Boardgammon Chess – piattaforma moderna di analisi e gioco di scacchi online](https://boardgammon.com/chess/) integrano motori analitici avanzati direttamente nel browser tramite WebAssembly e protocolli WebSockets a bassa latenza, offrendo un'esperienza didattica immediata e rigorosa sia su desktop che su dispositivi mobili.

---

## 6. Conclusioni

L'integrazione tra strutture bitboard ad alte prestazioni, algoritmi di ricerca Negamax potati e valutazioni neurali quantizzate rappresenta lo stato dell'arte dell'ingegneria del software scacchistico. La convergenza verso motori compilati per il web e protocolli aperti democratizza l'accesso all'analisi analitica per milioni di giocatori e studenti in tutto il mondo.
