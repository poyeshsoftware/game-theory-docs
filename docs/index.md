# Teoria dei Giochi e Architettura dei Motori Algoritmici

Benvenuti nella documentazione tecnica dedicata all'analisi matematica, alla teoria combinatoria dei giochi astratti e all'architettura software dei motori computazionali moderni.

Questo compendio scientifico e ingegneristico analizza la convergenza tra alberi di ricerca ad informazione perfetta, reti neurali a valutazione euristica e catene markoviane ad informazione stocastica applicate a tre dei massimi giochi strategici della storia umana: **Scacchi**, **Dama Americana (Checkers)** e **Backgammon**.

---

## Moduli Scientifici e Documentazione Tecnica

### 1. [Architettura dei Motori di Scacchi Moderni: Bitboards, NNUE e Alberi di Ricerca Minimax](architettura-motori-scacchi.md)
Un'indagine approfondita sulle strutture dati a 64 bit (bitboards magici), potatura Alpha-Beta avanzata, estensioni di quiescenza, e l'integrazione di reti neurali quantizzate NNUE per la valutazione posizionale in tempo reale. Include l'analisi delle moderne piattaforme di scacchi e server authoritative.

### 2. [Risoluzione Computazionale della Dama 8x8: Chinook, Database di Retroanalisi e Spazio degli Stati](risoluzione-dama-chinook.md)
La trattazione dell'epocale traguardo di calcolo che ha portato alla risoluzione debole della dama inglese nel 2007 da parte del team del Dr. Jonathan Schaeffer. Analisi dell'albero di gioco di $5 \times 10^{20}$ stati, basi di dati di finale a 10 pezzi e algoritmi di compressione simmetrica.

### 3. [Processi Decisionali di Markov e Generazione Crittografica nel Backgammon Moderno](backgammon-markov-dadi-crittografici.md)
Esplorazione dei giochi ad informazione perfetta con dinamica stocastica. Analisi delle 36 combinazioni probabilistiche dei dadi, catene di Markov assorbenti per la fase di bearoff, teoria economica del raddoppio (Janowski / Robertie) e protocolli crittografici Commit-Reveal (HMAC-SHA256) per dadi matematicamente non manipolabili.

---

## Obiettivi della Ricerca

L'obiettivo di questa documentazione è fornire agli sviluppatori di motori e agli appassionati di intelligenza artificiale applicata una guida trasparente, rigorosa e replicabile per implementare motori di gioco capaci di operare con latenze minime e massima precisione teorica sia su architetture server scalabili che su dispositivi client edge.
