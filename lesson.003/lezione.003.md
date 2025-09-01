# Impariamo il C: lezione 3

## 📜 Indice degli Argomenti

  - [Hello World e Introduzione a Concetti Fondamentali](https://www.google.com/search?q=%23hello-world-e-introduzione-a-concetti-fondamentali)
  - [Variabili e Assegnamento](https://www.google.com/search?q=%23variabili-e-assegnamento)
  - [Funzioni in C](https://www.google.com/search?q=%23funzioni-in-c)
  - [Scope delle Variabili: Locali, Globali e Statiche](https://www.google.com/search?q=%23scope-delle-variabili-locali-globali-e-statiche)
  - [Passaggio di Argomenti alle Funzioni](https://www.google.com/search?q=%23passaggio-di-argomenti-alle-funzioni)
  - [Tipi di Dato in C: `int`, `float`, `char` e `short`](https://www.google.com/search?q=%23tipi-di-dato-in-c-int-float-char-e-short)
  - [Promozione dei Tipi](https://www.google.com/search?q=%23promozione-dei-tipi)
  - [Operatori e Undefined Behavior](https://www.google.com/search?q=%23operatori-e-undefined-behavior)
  - [Considerazioni Finali](https://www.google.com/search?q=%23considerazioni-finali)

-----

### Hello World e Introduzione a Concetti Fondamentali

La terza lezione del corso continua a esplorare l'Hello World, approfondendo concetti chiave del linguaggio C. L'obiettivo è analizzare in dettaglio la struttura e il comportamento del codice, introducendo nuovi elementi come le **variabili**, le **funzioni** e i diversi **tipi di dato**.

-----

### Variabili e Assegnamento

Il concetto di variabile viene introdotto con l'esempio di una variabile locale chiamata `x` all'interno di una funzione. Si spiega come una variabile venga dichiarata, inizializzata e modificata.

Il codice mostra l'assegnamento (`=`) che non deve essere confuso con l'uguaglianza (`==`).

  * `=` è l'operatore di **assegnamento**, trasferisce il valore a destra alla variabile a sinistra.
  * `==` è l'operatore di **comparazione**, verifica se i due operandi sono uguali.

<!-- end list -->

```c
x = x + 1; // Assegna ad 'x' il valore di 'x + 1'.
if (x == 5) // Confronta 'x' con il valore 5.
{
    // ...
}
```

-----

### Funzioni in C

Viene creata una funzione chiamata `inc` per dimostrare il concetto di incremento. La funzione, inizialmente, viene definita come `void`, il che significa che non restituisce alcun valore e non accetta parametri. Si sottolinea che una funzione che ha solo un **effetto collaterale** (come stampare qualcosa a schermo) è un paradigma comune nei linguaggi imperativi come C, Go, Python e JavaScript.

**Funzione `inc` (Versione 1):**

```c
#include <stdio.h>

void inc() {
    int x = 1;
    x = x + 1;
    printf("x = %d\n", x);
}

int main() {
    inc(); // Stampa 2
    inc(); // Stampa 2
    inc(); // Stampa 2
    inc(); // Stampa 2
    return 0;
}
```

In questo esempio, ogni volta che `inc()` viene chiamata, una **variabile locale `x`** viene creata, incrementata e distrutta, senza mantenere lo stato tra le chiamate. Il risultato è che viene sempre stampato `2`.

-----

### Scope delle Variabili: Locali, Globali e Statiche

Viene introdotto il concetto di **scope** delle variabili, che definisce dove una variabile è accessibile.

  * **Variabili Locali**: Dichiarate all'interno di una funzione. Esistono solo per la durata della chiamata a quella funzione. Vengono create all'entrata e distrutte all'uscita.

  * **Variabili Globali**: Dichiarate al di fuori di qualsiasi funzione. Esistono per tutta la durata dell'esecuzione del programma. Il loro stato viene mantenuto tra le chiamate alle funzioni che le utilizzano.

**Funzione `inc` (Versione 2 - con variabile globale):**

```c
#include <stdio.h>

int x = 0; // Variabile globale

void inc() {
    x = x + 1;
    printf("x = %d\n", x);
}

int main() {
    inc(); // Stampa 1
    inc(); // Stampa 2
    inc(); // Stampa 3
    inc(); // Stampa 4
    return 0;
}
```

In questo caso, `x` mantiene il suo stato e si incrementa a ogni chiamata.

  * **Variabili Statiche**: Sono un ibrido. Vengono dichiarate all'interno di una funzione con la parola chiave `static`. Hanno il "lifetime" di una variabile globale (mantengono il loro valore tra le chiamate), ma lo "scope" (visibilità) è limitato alla funzione in cui sono definite.

**Funzione `inc` (Versione 3 - con variabile statica):**

```c
#include <stdio.h>

void inc() {
    static int x = 0; // Variabile statica
    x = x + 1;
    printf("x = %d\n", x);
}

int main() {
    inc(); // Stampa 1
    inc(); // Stampa 2
    inc(); // Stampa 3
    inc(); // Stampa 4
    return 0;
}
```

Si avverte che le variabili globali (e statiche) possono causare problemi in un contesto multi-thread a causa di problemi di sincronizzazione, richiedendo meccanismi di protezione come i mutex.

-----

### Passaggio di Argomenti alle Funzioni

Un punto cruciale del C è che gli argomenti vengono passati **per valore** (`pass-by-value`). Ciò significa che le funzioni ricevono una **copia** dei valori, non un riferimento all'originale.

**Funzione `inc` (Versione 4 - passaggio per valore):**

```c
#include <stdio.h>

int inc(int x) {
    x = x + 1;
    return x;
}

int main() {
    int a = 10;
    inc(a); 
    printf("a = %d\n", a); // Stampa 10
    
    a = inc(a);
    printf("a = %d\n", a); // Stampa 11
    return 0;
}
```

Nel primo caso, `inc(a)` riceve una copia del valore `10`. L'incremento avviene sulla copia, che poi viene ignorata, lasciando il valore originale di `a` invariato. Nel secondo caso, `a` viene assegnato al valore di ritorno della funzione, mutando il suo valore.

Il **passaggio per riferimento** in C si ottiene unicamente tramite l'uso dei **puntatori**, un argomento che verrà affrontato in seguito, ma che è fondamentale per la comprensione del linguaggio.

-----

### Tipi di Dato in C: `int`, `float`, `char` e `short`

Oltre a `int`, il corso introduce altri tipi di dati numerici.

  * `int`: Numero intero (positivo o negativo)
  * `unsigned int`: Numero intero senza segno (solo positivo).
  * `float`: Numero in virgola mobile a singola precisione (32 bit, standard IEEE 754).
  * `double`: Numero in virgola mobile a doppia precisione (64 bit).
  * `char`: Tipo intero di piccole dimensioni, solitamente un byte (8 bit).
  * `short`: Tipo intero, tradizionalmente 16 bit.

**Esempio di codice con `int` e `float`:**

```c
#include <stdio.h>

int main() {
    int a = 10;
    float y = 1.234;
    printf("intero: %d, float: %f\n", a, y); 
    printf("esadecimale: %X, float formattato: %.3f\n", a, y); 
    return 0;
}
```

Si sottolinea la pericolosità del C, che permette la compilazione anche in presenza di `warning` (ad esempio, passando un tipo di dato errato a `printf`).

-----

### Promozione dei Tipi

Il linguaggio C ha regole di **promozione automatica** dei tipi per semplificare l'utilizzo di funzioni variadiche come `printf`.

  * I tipi interi più piccoli di `int` (come `char` e `short`) vengono automaticamente convertiti in `int` quando passati a una funzione variadica.
  * I tipi `float` vengono promossi a `double`.

Questo comportamento, sebbene utile, può portare a confusioni e a comportamenti inattesi se non compreso appieno.

-----

### Operatori e Undefined Behavior

Viene introdotto l'operatore di incremento `++` (`c++` o `++c`), che è l'equivalente abbreviato di `c = c + 1`. L'esempio più famoso di questo operatore è nel nome del linguaggio **C++**, che si propone come una versione "incrementata" del C.

Viene discusso l'**Undefined Behavior (UB)**, un concetto cruciale del C. L'**overflow** su un intero con segno (`signed`) è un comportamento non specificato dalla specifica del linguaggio.

  * `char c = 127; c++;` : Il risultato è imprevedibile. Potrebbe essere `-128` (per **wrapping**), ma anche qualsiasi altro valore, o addirittura un crash del programma.
  * `unsigned char c = 255; c++;` : Il risultato è **garantito** dalla specifica C. L'overflow causa un "wrapping" che riporta il valore a `0`.

-----

### Considerazioni Finali

La lezione conclude sottolineando l'importanza di comprendere a fondo i tipi di dato, la loro rappresentazione in memoria (es. lo standard FP32 per i float) e le regole di conversione implicite. Il C, pur essendo un linguaggio potente, richiede una conoscenza approfondita delle sue peculiarità per evitare errori e comportamenti imprevedibili. La prossima puntata si concentrerà sui **puntatori**.
