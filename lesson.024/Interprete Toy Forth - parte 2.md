## 🧭 Indice degli Argomenti

  - [1. Strutture Dati e Contesto di Esecuzione](#1-strutture-dati-e-contesto-di-esecuzione)
      - [1.1. L'Oggetto Base (`tfobj`)](#11-loggetto-base-tfobj)
      - [1.2. La Struttura di Stato del Parser (`TFParser`)](#12-la-struttura-di-stato-del-parser-tfparser)
  - [2. Ottimizzazione: Array Dinamico vs. Linked List](#2-ottimizzazione-array-dinamico-vs-linked-list)
      - [2.1. Vantaggi dell'Array Dinamico (Cache Locality e O(1) Ammortizzato)](#21-vantaggi-dellarray-dinamico-cache-locality-e-o1-ammortizzato)
      - [2.2. Implementazione di `listPush` con `realloc`](#22-implementazione-di-listpush-con-realloc)
  - [3. Gestione I/O: Caricamento del Sorgente da File](#3-gestione-io-caricamento-del-sorgente-da-file)
      - [3.1. Flusso di Dati per la Lettura Portabile (Sequence Diagram)](#31-flusso-di-dati-per-la-lettura-portabile-sequence-diagram)
      - [3.2. Estratto di Codice per la Lettura del File](#32-estratto-di-codice-per-la-lettura-del-file)
  - [4. Il Processo di Compilazione (Parsing)](#4-il-processo-di-compilazione-parsing)
      - [4.1. Logica Concettuale della Funzione `compile` (Workflow)](#41-logica-concettuale-della-funzione-compile-workflow)
      - [4.2. Implementazione delle Funzioni di Parsing in C](#42-implementazione-delle-funzioni-di-parsing-in-c)
  - [5. Linguaggi Concatenativi e Variable Capturing](#5-linguaggi-concatenativi-e-variable-capturing)

-----

## 1\. Strutture Dati e Contesto di Esecuzione

### 1.1. L'Oggetto Base (`tfobj`)

[cite\_start]La struttura `tfobj` (Toy Fort Object) è l'oggetto **polimorfico** fondamentale [cite: 4] [cite\_start]usato sia per gli elementi sintattici durante il parsing [cite: 5] [cite\_start]che per lo stato di esecuzione (variabili, stack)[cite: 7].

```c
// L'equivalente di un RedisObject o PyObject
typedef struct tfobj {
    size_t type;            // TFOBJ_TYPE_INT, TFOBJ_TYPE_LIST, ecc.
    // int ref_count;       // Per la gestione della memoria (non ancora implementata)
    union {
        long i;             // Valore per TFOBJ_TYPE_INT
        struct {
            struct tfobj **ele; // Array di puntatori (per lista/stack)
            size_t len;
            // size_t alloc_len; // Per l'ottimizzazione dell'allocazione
        } list;
    };
} tfobj;
```

### 1.2. La Struttura di Stato del Parser (`TFParser`)

[cite\_start]Per gestire lo stato del parsing in modo efficiente e centrale, si usa la struttura `TFParser`[cite: 9, 12]. [cite\_start]Questo evita la necessità di passare continuamente puntatori a puntatori per aggiornare la posizione corrente[cite: 11].

```c
// Struttura per tracciare lo stato e il contesto del parsing
typedef struct tfparser {
    char *prg; // Puntatore all'inizio del programma sorgente
    char *p;   // Puntatore alla posizione corrente di parsing (avanza)
    // int line; // Numero di linea corrente (utile per segnalare errori a runtime/sintassi)
} tfparser;
```

-----

## 2\. Ottimizzazione: Array Dinamico vs. Linked List

[cite\_start]Il relatore spiega perché la scelta di implementare liste e stack con un **array dinamico** (in C, puntatore e `realloc`) sia superiore alla *linked list* in un'implementazione seria, sebbene la *linked list* garantisca O(1) costante[cite: 19].

### 2.1. Vantaggi dell'Array Dinamico (Cache Locality e O(1) Ammortizzato)

  * [cite\_start]**Cache Locality (Località della Cache)**: I dati sono contigui in memoria, garantendo una maggiore efficienza d'accesso per la CPU e riducendo lo spreco di memoria[cite: 21, 35]. Questo è un fattore cruciale nei sistemi ad alte prestazioni.
  * [cite\_start]**Accesso Casuale O(1)**: Un array consente l'accesso all'elemento $i$-esimo in tempo costante [cite: 35, 36][cite\_start], a differenza della *linked list* che richiederebbe $O(N)$[cite: 37].
  * [cite\_start]**Complessità Ammortizzata ($O(1)$)**: Sovra-allocando l'array per potenze di due (`alloc_len`) [cite: 33][cite\_start], l'operazione costosa di `realloc` avviene sempre meno frequentemente[cite: 35]. [cite\_start]Il costo medio dell'inserimento (`push`) è quindi **costante** ($O(1)$)[cite: 34].

### 2.2. Implementazione di `listPush` con `realloc`

[cite\_start]La funzione `listPush` utilizza `realloc` in modo opportunistico: se la lista era vuota (`l->list.ele` è `NULL`), `realloc` agisce come `malloc`; altrimenti, espande o sposta il blocco di memoria[cite: 175, 174].

```c
// listPush: aggiunge un elemento alla fine della lista 'l'
void listPush(tfobj *l, tfobj *ele) {
    size_t new_len = l->list.len + 1;
    
    // Riallaca lo spazio necessario per l'array di puntatori
    // sizeof(tfobj*) * new_len
    tfobj **new_ele = realloc(l->list.ele, sizeof(tfobj*) * new_len);

    // [omesso: controllo di errore per realloc]

    l->list.ele = new_ele; // Aggiorna il puntatore all'array

    // Aggiunge il nuovo elemento
    l->list.ele[l->list.len] = ele;
    
    // Aggiorna la lunghezza
    l->list.len++; [cite_start]// [cite: 171]

    /*
    NOTA SUL REF COUNTING:
    Si stabilisce che è compito del chiamante (caller) incrementare il reference count 
    [cite_start]dell'elemento 'ele' se necessario, non della funzione listPush. [cite: 169]
    */
}
```

-----

## 3\. Gestione I/O: Caricamento del Sorgente da File

[cite\_start]Il programma deve leggere l'intero file sorgente in un buffer di memoria in modo **portatile** (usando solo le funzioni della Lib C come `fseek`, `ftell`, `fread`)[cite: 47].

### 3.1. Flusso di Dati per la Lettura Portabile (Sequence Diagram)

```mermaid
sequenceDiagram
    participant App as Programma C
    participant FS as File Stream (fp)
    participant Heap as Memoria (prgtext)

    App->FS: fp = fopen(argv[1], "r")
    [cite_start]Note over App: Gestione errore if (fp == NULL) [cite: 74]
    
    App->FS: fseek(fp, 0, SEEK_END)
    App->FS: file_size = ftell(fp)
    
    App->Heap: prgtext = xmalloc(file_size + 1)
    
    App->FS: fseek(fp, 0, SEEK_SET)
    [cite_start]Note over App: Riporta il cursore all'inizio (SEEK_SET è fondamentale dopo SEEK_END) [cite: 66, 68]
    
    App->FS: fread(prgtext, 1, file_size, fp)
    App->Heap: prgtext[file_size] = '\0'
    
    App->FS: fclose(fp)
    Note over App: Sorgente pronto per compile()
```

### 3.2. Estratto di Codice per la Lettura del File

```c
#include <stdio.h>
#include <stdlib.h> // Per xmalloc, realloc, atol
#include <string.h> // Per memcpy
#include <ctype.h>  // Per isspace, isdigit

// ... (dichiarazioni di tfobj, tfparser, xmalloc)

// Lettura e allocazione del sorgente
void main(int argc, char **argv) {
    if (argc < 2) {
        printf("Usage: %s <program_file>\n", argv[0]);
        return;
    }

    FILE *fp = fopen(argv[1], "r");
    if (fp == NULL) {
        perror("Opening Toy Fort program"); [cite_start]// [cite: 75]
        return 1;
    }

    [cite_start]// 1. Determina dimensione del file [cite: 47]
    fseek(fp, 0, SEEK_END);
    long file_size = ftell(fp); [cite_start]// [cite: 52]

    [cite_start]// 2. Alloca buffer (file_size + 1 per il null term) [cite: 58]
    char *prgtext = xmalloc(file_size + 1);

    [cite_start]// 3. Riposiziona cursore all'inizio [cite: 67, 68]
    fseek(fp, 0, SEEK_SET);

    [cite_start]// 4. Legge il contenuto e null-termina [cite: 61]
    fread(prgtext, 1, file_size, fp);
    prgtext[file_size] = 0;

    fclose(fp); [cite_start]// [cite: 48]
    
    tfobj *parsed = compile(prgtext);
    // ...
}
```

-----

## 4\. Il Processo di Compilazione (Parsing)

[cite\_start]La funzione `compile(char *prg)` trasforma la stringa sorgente in una lista di oggetti (`tfobj`)[cite: 109, 110]. [cite\_start]Questo processo è un parser scritto "a mano", preferito per implementazioni serie[cite: 114].

### 4.1. Logica Concettuale della Funzione `compile` (Workflow)

```mermaid
graph TD
    A[Inizia compile(prg)] --> B(Inizializza TFParser & tfobj *parsed = createListObject());
    B --> C(Loop: while (parser->p[0] != '\0'));
    C --> D(parseSpaces(parser));
    D --> E{parser->p[0] == '\0'?};
    E -- Sì --> F[Break/End Loop];
    E -- No --> G(token_start = parser->p);
    G --> H{È Numero (isdigit o '-')?};
    H -- Sì --> I(o = parseNumber(parser));
    H -- No --> J(Altro Token: Simbolo, Boolean, ecc.);
    I |o| J --> K{o == NULL? (Errore di sintassi)};
    K -- Sì --> L(Segnala Errore 'syntax error near...' token_start);
    L --> M[Return NULL / Cleanup];
    K -- No --> N(listPush(parsed, o));
    N --> C;
    F --> P[Ritorna parsed (tfobj Lista)];
```

### 4.2. Implementazione delle Funzioni di Parsing in C

**1. `parseSpaces`**

[cite\_start]Questa funzione è il primo passo in ogni iterazione del ciclo di parsing per saltare gli spazi bianchi[cite: 122, 127].

```c
[cite_start]// Consuma gli spazi, i tab e i newline usando ctype.h [cite: 125, 126]
void parseSpaces(tfparser *parser) {
    #include <ctype.h>
    while (isspace((unsigned char)parser->p[0])) {
        parser->p++;
    }
}
```

**2. `parseNumber`**

[cite\_start]Questa funzione estrae un numero intero con segno e avanza il puntatore del parser[cite: 132, 184].

```c
#define MAX_NUM_LEN 128

// Restituisce un nuovo tfobj di tipo INT, o NULL in caso di errore
tfobj *parseNumber(tfparser *parser) {
    char *start = parser->p; [cite_start]// Inizio del token [cite: 147, 191]
    char buff[MAX_NUM_LEN];

    [cite_start]// 1. Gestione del segno negativo iniziale [cite: 131, 192]
    if (parser->p[0] == '-') {
        parser->p++;
    }

    [cite_start]// 2. Consumo cifre e avanzamento di parser->p [cite: 188, 192]
    while (isdigit((unsigned char)parser->p[0])) {
        parser->p++;
    }

    char *end = parser->p;
    int numlen = end - start; [cite_start]// Lunghezza del numero (sottrazione tra puntatori) [cite: 196]

    if (numlen == 0 || numlen >= MAX_NUM_LEN) return NULL; [cite_start]// [cite: 198]

    [cite_start]// 3. Copia e null-termination [cite: 201]
    memcpy(buff, start, numlen);
    buff[numlen] = '\0';

    [cite_start]// 4. Conversione (si usa la semplice atoi/atol) e creazione oggetto [cite: 202, 204]
    long value = atol(buff);
    // Assumendo che esista:
    // tfobj *o = createIntObject(value);
    // return o; 
    
    return NULL; // Placeholder per l'oggetto parsato
}
```

-----

## 5\. Linguaggi Concatenativi e Variable Capturing

[cite\_start]Toy Fort appartiene alla famiglia dei linguaggi **concatenativi** (come Forth, Joy [cite: 81][cite\_start], Qort [cite: 83][cite\_start]), basati sullo **stack**[cite: 96].

**Esempio Stack-Based:** `5 dup * print`

1.  [cite\_start]**5**: Push 5. (Stack: `[5]`) [cite: 93]
2.  **dup**: Duplica l'elemento in cima. (Stack: `[5, 5]`) [cite\_start][cite: 93]
3.  **\***: Pop 5 e 5, moltiplica. [cite\_start]Push 25. (Stack: `[25]`) [cite: 94]
4.  [cite\_start]**print**: Pop 25. Output 25. [cite: 95]

### 5.1. La Proposta delle Tuple per il Variable Capturing

[cite\_start]Il relatore menziona una sua modifica storica [cite: 84, 87] [cite\_start]a questi linguaggi per introdurre variabili locali senza violare la loro semantica *stack-based*, usando una **Tupla** `[a, b]` per la cattura (capture) di variabili[cite: 97, 105, 107].

Questa tecnica:

  * [cite\_start]Permette di implementare costrutti complessi come `swap` (`5 10 swap` $\rightarrow$ `10 5`) in modo più leggibile: `[a, b]`[cite: 105].
  * [cite\_start]Rende i programmi "molto più semplici da utilizzare"[cite: 100, 106].
  * [cite\_start]La cattura avviene in una fase di **precompilazione** o **compilazione in bytecode**, dove i simboli (`a`) sono associati a un indice fisso in una tabella di variabili locali[cite: 107].

[cite\_start]**Prossimo Passo:** Vuoi che implementi la funzione `exec(tfobj *prg)` in modo minimale (ad esempio, popolare uno stack per gli interi) per vedere il programma compilato in azione, come suggerito dal relatore[cite: 242]?