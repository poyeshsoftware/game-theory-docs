# Processi Decisionali di Markov e Generazione Crittografica nel Backgammon Moderno

## 1. Natura Matematica del Backgammon: Informazione Perfetta e Dinamica Stocastica

A differenza di scacchi e dama, che sono giochi puramente deterministici a informazione perfetta, il **Backgammon** appartiene alla classe dei giochi stocastici a due giocatori a somma zero. 

Lo stato del tabellone è sempre pienamente osservabile da entrambi i giocatori (informazione perfetta), ma la transizione tra stati successivi è governata da una variabile aleatoria discreta: il lancio di due dadi a sei facce ($D \in \{1..6\}^2$), con $6 \times 6 = 36$ esiti possibili e 21 combinazioni uniche equiprobabili.

Di conseguenza, il formalismo di risoluzione matematica non è il semplice Minimax, bensì l'**Expectiminimax**, in cui l'albero di gioco alterna nodi decisionali (Max e Min) a nodi di probabilità (*chance nodes*):

$$V(s) = \sum_{r \in \mathcal{R}} P(r) \cdot \max_{m \in M(s, r)} \left( -V(\text{do\_move}(s, r, m)) \right)$$

dove $P(r) = \frac{1}{18}$ per i tiri non doppi (ordinati in due permutazioni) e $P(r) = \frac{1}{36}$ per i tiri doppi (che conferiscono 4 mosse del medesimo valore).

---

## 2. Catene di Markov Assorbenti e Risoluzione Esatta del Bearoff

Nelle fasi conclusive della partita, quando tutte le pedine di un giocatore si trovano nella propria tavola interna (*home board*, punti 1-6) e non vi sono pedine avversarie a ostacolare la corsa, il gioco si trasforma in un processo Markoviano puro.

Questo sottospazio di stati è formalizzato come una **Catena di Markov Assorbente**:
- Uno stato è definito dal vettore delle pedine presenti sui punti interni: $\mathbf{x} = (x_1, x_2, x_3, x_4, x_5, x_6)$ con $\sum_{i=1}^6 x_i \le 15$.
- Lo stato terminale unico è lo stato di assorbimento $\mathbf{x}_{\text{goal}} = (0, 0, 0, 0, 0, 0)$ (tutte le pedine rimosse dal tabellone).
- La matrice di transizione può essere partizionata in forma canonica:

$$P = \begin{pmatrix} Q & R \\ 0 & I \end{pmatrix}$$

dove $Q$ è la matrice di transizione tra i 54.264 stati transitori, $R$ è il vettore di assorbimento e $I$ è la matrice identità dello stato assorbente.

### Calcolo del Numero Medio di Tiri e Matrice Fondamentale

La matrice fondamentale della catena è definita da:

$$N = (I - Q)^{-1} = \sum_{k=0}^{\infty} Q^k$$

L'elemento $N_{ij}$ rappresenta il numero medio di volte in cui il processo visita lo stato $j$ partendo dallo stato iniziale $i$. Il numero atteso di tiri per completare l'uscita è dato semplicemente dalla somma riga della matrice fondamentale:

$$\mathbf{t} = N \mathbf{1}$$

I moderni motori di backgammon (come GNU Backgammon e WildBG) memorizzano la matrice fondamentale e le distribuzioni di probabilità cumulate in tabelle binarie ultra-compatte (*bearoff databases*), consentendo la valutazione del 100% dell'equity di fine partita in tempo O(1).

---

## 3. Economia della Teoria dei Giochi: Il Cubo del Raddoppio

Una caratteristica distintiva del backgammon strategico competitivo è il **cubo del raddoppio** (*doubling cube*). Il cubo introduce un mercato azionario continuo all'interno della partita:

- Un giocatore può proporre di raddoppiare la posta prima del proprio lancio.
- L'avversario può rifiutare (*drop*), perdendo 1 punto, oppure accettare (*take*), giocando per 2 punti con il possesso esclusivo del cubo per futuri rilanci.

### La Soglia del 25% di Janowski e l'Opzione Americana

In condizioni di gioco monetario (*money play*), la condizione di indifferenza per l'accettazione del raddoppio è:

$$\text{Valore(Drop)} = -1$$
$$\text{Valore(Take)} = p \cdot (+2) + (1 - p) \cdot (-2) = 4p - 2$$

Uguagliando i due valori:

$$4p - 2 \ge -1 \implies 4p \ge 1 \implies p \ge 0.25$$

Se la probabilità di vittoria $p$ è pari ad almeno il **25%**, l'avversario ha convenienza matematica ad accettare. Tuttavia, il possesso del cubo attribuisce un valore economico addizionale assimilabile a un'opzione finanziaria americana (*ownership equity*), consentendo rilanci strategici che riducono ulteriormente la soglia pratica di accettazione fino al 21.5% in determinate configurazioni volatili.

---

## 4. Crittografia dei Dadi Provabilmente Equi: Protocollo Commit-Reveal HMAC-SHA256

Nei giochi online in cui le decisioni dipendono da variabili aleatorie, la fiducia nell'imparzialità del generatore di numeri pseudocasuali (PRNG) è critica. L'architettura software moderna deve garantire l'assoluta impossibilità di manipolazione sia da parte del server che da parte del client.

La soluzione matematica a questo dilemma è il protocollo crittografico **Commit-Reveal**:

1. **Fase di Impegno (Commitment)**: Prima dell'inizio del round, il server genera un seme segreto crittografico casuale di 256 bit (`server_seed`). Il server calcola l'hash SHA-256 del seme e lo trasmette pubblicamente al client:
   $$\text{Commitment} = \text{SHA-256}(\text{server\_seed})$$
2. **Fase di Entropia Client (Client Seed)**: Il client genera a sua volta un seme arbitrario (`client_seed`) o un nonce incrementale, inviandolo al server.
3. **Generazione Deterministica**: Il valore finale dei dadi viene calcolato tramite un codice di autenticazione di messaggio basato su hash (HMAC):
   $$\text{Digest} = \text{HMAC-SHA256}(\text{server\_seed}, \ \text{client\_seed} \parallel \text{round\_nonce})$$
   I byte risultanti vengono estratti e mappati uniformemente nell'intervallo $[1..6]$ per ciascun dado.
4. **Fase di Rivelazione e Verifica**: Al termine del turno o della partita, il server rivela in chiaro il `server_seed`. Il client e qualsiasi osservatore terzo possono verificare indipendentemente che l'hash corrisponda al commitment originale e che i tiri generati siano stati una conseguenza puramente matematica delle due componenti di entropia combinata.

Per un approfondimento ingegneristico dettagliato sulle prove matematiche, sui vettori di test e sul codice open-source del generatore crittografico, consultare la documentazione formale del [protocollo crittografico per dadi provabilmente equi basato su Commit-Reveal HMAC-SHA256](https://boardgammon.com/fair-dice/), dove vengono esposti gli standard algoritmici implementati in Boardgammon.

---

## 5. Conclusioni

Il backgammon unisce in modo armonico la modellazione stocastica avanzata mediante catene di Markov assorbenti, l'economia decisionale delle opzioni finanziarie attraverso il cubo del raddoppio e la trasparenza matematica garantita dai moderni protocolli crittografici a prova di manomissione. Questa sintesi lo rende uno dei campi più affascinanti e rigorosi per la ricerca informatica teorica e applicata.
