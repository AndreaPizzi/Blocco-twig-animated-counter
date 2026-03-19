# Blocco Numeri (Counter)

Blocco Gutenberg che mostra statistiche numeriche animate. Supporta due tipologie di counter (digit-by-digit e lineare), due dimensioni di visualizzazione (grande e piccolo) e una sequenza di animazione orchestrata via GSAP + ScrollTrigger.

---

## File

| File | Descrizione |
|---|---|
| `templates/blocks/blocco_numeri.twig` | Template del blocco Gutenberg |
| `assets/scss/_blocco_numeri.scss` | Stili blocco, celle, linee, numeri |
| `assets/js/blocco-numeri.js` | Animazioni GSAP + logica counter |

---

## Configurazione ACF

### Campo singolo

| Nome campo | Label | Tipo | Note |
|---|---|---|---|
| `titolo` | Titolo | Text | Titolo del blocco (opzionale) |

### Repeater: `counters`

Il blocco usa un repeater ACF con struttura fissa da **5 elementi**: 2 grandi + 3 piccoli.

| Nome campo | Label | Tipo | Note |
|---|---|---|---|
| `numero_inizio` | Numero inizio | Number | Valore di partenza del counter (default `0`) |
| `numero_fine` | Numero fine | Number | Valore finale del counter |
| `testo` | Testo | Text | Label descrittiva sotto il numero |
| `dimensione` | Dimensione | Select | `grande` / `piccolo` — determina il layout |
| `counter_linear` | Counter lineare | True/False | Se spuntato usa animazione lineare (0→N), altrimenti digit-by-digit |

Per il campo `dimensione` impostare in ACF:
- **Choices**: `grande : Grande` e `piccolo : Piccolo`
- **Return format**: Value

---

## Layout

Il blocco è diviso in due righe via Bootstrap grid:

**Riga grandi** (`dimensione = grande`) — Bootstrap `col-12 col-md-6`:
- 2 colonne al 50% su desktop
- 100% larghezza su mobile, impilate

**Riga piccoli** (`dimensione = piccolo`) — Bootstrap `col-12 col-md-4`:
- 3 colonne al 33% su desktop
- 100% larghezza su mobile, impilate

I bordi verticali tra le celle sono gestiti via classe CSS `border-end-custom` applicata automaticamente dal Twig su tutti gli elementi tranne l'ultimo di ogni riga.

---

## Modalità counter

### `digit` (default)

Ogni cifra del numero finale viene animata separatamente con uno stagger da sinistra a destra — effetto "slot machine" per cifra.

Esempio con il numero `1500`:
```
digit 0: 0 → 1   (parte per prima, durata maggiore)
digit 1: 0 → 5
digit 2: 0 → 0   (rimane statica)
digit 3: 0 → 0   (rimane statica)
```

### `linear`

Counter classico che incrementa di 1 in 1 dal valore iniziale fino al valore finale.

Esempio con il numero `1500`:
```
0 → 1 → 2 → ... → 1499 → 1500
```

Entrambe le modalità hanno un micro fade alpha (`opacity 0.35→1`) ad ogni cambio di cifra per rendere le transizioni più fluide.

Per selezionare la modalità, spuntare il campo **Counter lineare** in ACF sul singolo elemento del repeater.

---

## Sequenza animazione

L'animazione è triggerata da ScrollTrigger sull'**intero blocco** — le celle entrano indipendentemente da dove si trovano nel viewport.

```
1. Linea top      → scaleX 0→1 da sinistra          (0.7s)
2. Label testo    → fade in con stagger               (0.55s, stagger 0.07s)
3. Celle          → fade in con stagger               (0.5s,  stagger 0.08s)
4. Linea mid      → scaleX 0→1 da sinistra           (0.56s)
5. Linea bottom   → scaleX 0→1 da sinistra           (0.56s)
6. Counter        → digit-by-digit o linear           (2.0s,  stagger 0.15s tra counter)
```

Il testo label appare **prima** dei numeri — i counter partono ad animarsi solo dopo che tutte le celle sono visibili.

---

## Setup

### Importare e inizializzare il JS

In `assets/js/main.js`:

```javascript
import bloccoNumeri from './blocco-numeri.js';
bloccoNumeri();
```

---

## Titolo del blocco

Il titolo usa il sistema di animazioni globale `animations.js` con `data-animate="clip-up"`:

```twig
<h2 data-animate="clip-up"
    data-style="soft"
    data-duration="0.6"
    data-delay="0.2">
    {{ fields.titolo }}
</h2>
```

Per modificare lo stile dell'animazione del titolo si possono usare i data attributes del sistema `animations.js` — vedere la relativa documentazione.

---

## Responsive

- **Numeri grandi**: `clamp(72px, 8vw, 140px)` su desktop, `clamp(64px, 18vw, 100px)` su mobile
- **Numeri piccoli**: `clamp(32px, 3.5vw, 48px)` su desktop, `clamp(28px, 10vw, 48px)` su mobile
- **Bordi verticali**: visibili su desktop, sostituiti da bordi orizzontali su mobile
- **Layout**: colonne affiancate su desktop, impilate al 100% su mobile

---

## Note tecniche

Gli stati iniziali di opacità e transform sono impostati tramite `gsap.set()` in modo **sincrono** prima del `window.load` — questo evita il flash di contenuto visibile prima che le animazioni vengano inizializzate.

Le linee orizzontali usano `scaleX: 0` con `transform-origin: left center` per l'effetto di rivelazione da sinistra a destra. I valori iniziali sono impostati anche nel SCSS (`transform: scaleX(0)`) come ulteriore protezione contro il flash.
