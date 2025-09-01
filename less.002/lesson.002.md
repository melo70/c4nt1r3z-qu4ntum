# Impariamo il C: lezione 2

Nel secondo episodio del corso, il relatore si addentra nei dettagli della **struttura di un programma C**, analizzando in profondità la funzione `main`, il concetto di variabili locali e l'importanza del valore di ritorno per il sistema operativo.

### Indice degli argomenti

  - [La Funzione `main`](https://www.google.com/search?q=%23la-funzione-main)
  - [Variabili e Tipi di Dato](https://www.google.com/search?q=%23variabili-e-tipi-di-dato)
  - [La Funzione `printf`](https://www.google.com/search?q=%23la-funzione-printf)
  - [Il Valore di Ritorno di `main`](https://www.google.com/search?q=%23il-valore-di-ritorno-di-main)
  - [Workflow del Programma](https://www.google.com/search?q=%23workflow-del-programma)

-----

### La Funzione `main`

La funzione **`main`** è il punto di ingresso di ogni programma C. È una funzione speciale che non ha bisogno di essere chiamata da un'altra funzione, poiché il sistema operativo la invoca automaticamente all'avvio del programma.

La sua firma canonica è `int main(void)`, dove:

  - **`int`**: Indica che la funzione ritorna un **valore intero**.
  - **`main`**: È il **nome** della funzione.
  - **`(void)`**: Specifica che la funzione **non accetta argomenti** in input.

Il relatore suggerisce di usare sempre `void` per esplicitare l'assenza di parametri, anche se i compilatori moderni potrebbero tollerare le parentesi vuote.

### Variabili e Tipi di Dato

A differenza di linguaggi come Python, in C ogni **variabile deve essere dichiarata** con il suo tipo di dato prima di essere utilizzata. Non c'è inferenza del tipo automatica.

#### Esempio di dichiarazione e inizializzazione

```c
#include <stdio.h>

int main(void) {
    int a = 10;
    int b = 20;
    int c = a + b;
    
    printf("Ciao: %d\n", c);
    
    return 0;
}
```

  - **`int a = 10;`**: Dichiara una variabile di tipo intero (`int`) chiamata `a` e la inizializza con il valore 10.
  - **`int c = a + b;`**: Inizializza la variabile `c` con il risultato di un'**espressione** (`a + b`). Le espressioni possono essere usate in diversi contesti, come nel ritorno di una funzione o nell'assegnazione di variabili.

#### Variabili Locali

Le variabili create all'interno di una funzione (come `a`, `b`, e `c` nell'esempio precedente) sono **variabili locali**. Queste variabili:

  - Vengono **create** quando la funzione viene chiamata.
  - Hanno un ciclo di vita limitato all'**esecuzione della funzione**.
  - Vengono **distrutte** una volta che la funzione termina.

Questo significa che il loro valore non può essere utilizzato al di fuori dello "scope" della funzione in cui sono state dichiarate. Lo stesso vale per gli argomenti di una funzione, che sono anch'essi variabili locali.

-----

### La Funzione `printf`

La funzione **`printf`** (che sta per "print formatted") viene usata per stampare output formattato. Il suo primo argomento è una **stringa di formato** che può contenere segnaposto per i valori da stampare.

  - **`%d`**: Indica che il prossimo argomento da stampare è un **numero intero decimale**.
  - **`printf`** è una funzione **variadica**, cioè accetta un numero variabile di argomenti.

#### Esempio di utilizzo

```c
printf("Hello World %d\n", sum(10, 20));
```

In questo caso, `printf` si aspetta un intero al posto di `%d`. Il compilatore C valuta prima l'espressione `sum(10, 20)` e poi passa il risultato, che è 30, a `printf`.

Il relatore sottolinea che se il numero di argomenti passati non corrisponde a quelli richiesti dalla stringa di formato, il compilatore genera un **warning** (un avviso) e il comportamento del programma non è definito, potendo causare errori o un'uscita inaspettata.

-----

### Il Valore di Ritorno di `main`

Anche se `main` è il punto di partenza del programma, il suo valore di ritorno non è superfluo. Questo valore serve a comunicare al **sistema operativo** l'esito dell'esecuzione del programma:

  - **`return 0`**: Indica che il programma si è concluso con **successo**.
  - **`return` (diverso da zero)**: Indica un **fallimento** dell'esecuzione.

Questo meccanismo è fondamentale in ambienti come la **shell di Unix** dove è possibile concatenare comandi (ad es. `command1 && command2`). Il secondo comando (`command2`) viene eseguito solo se il primo (`command1`) ha avuto successo (ovvero, ha ritornato 0).

-----

### Workflow del Programma

Il relatore descrive l'ordine di esecuzione di un programma che contiene una chiamata a una funzione.

**Workflow di Esecuzione**

1.  **Chiamata a `main`**: Il sistema operativo avvia il programma eseguendo la funzione `main`.
2.  **Valutazione degli argomenti**: La funzione `printf` deve sapere il valore di tutti i suoi argomenti.
3.  **Chiamata a `sum`**: Per risolvere l'argomento `sum(10, 20)`, il compilatore esegue la funzione `sum`.
4.  **Calcolo in `sum`**: All'interno di `sum`, vengono create le variabili locali `a`, `b` e `c`, viene eseguita la somma e il risultato viene restituito.
5.  **Distruzione delle variabili locali**: Al termine di `sum`, le variabili `a`, `b` e `c` vengono rimosse dalla memoria.
6.  **Esecuzione di `printf`**: Il valore restituito da `sum` (30) viene passato a `printf`, che stampa l'output.
7.  **`return` di `main`**: `main` restituisce 0 al sistema operativo, segnalando il successo.

-----

Vuoi approfondire il concetto di variabili locali e il loro ciclo di vita in C, o preferisci esplorare come vengono gestite le stringhe nel linguaggio C?
