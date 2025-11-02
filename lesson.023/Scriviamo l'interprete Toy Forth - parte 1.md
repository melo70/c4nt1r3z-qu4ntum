## 🧭 Indice degli Argomenti e Timeline

  - [1. Filosofia di Design e Contesto di Sviluppo](#1-filosofia-di-design-e-contesto-di-sviluppo) (0:40 - 2:20)
  - [2. Introduzione a Forth e Architettura Stack-Based](#2-introduzione-a-forth-e-architettura-stack-based) (1:55 - 4:40)
      - [2.1. Esempio di Programma Forth](#21-esempio-di-programma-forth) (2:02)
  - [3. Definizione della Struttura Dati Centrale: `tfobj`](#3-definizione-della-struttura-dati-centrale-tfobj) (4:40 - 15:40)
      - [3.1. Uso della `union` per il Polimorfismo](#31-uso-della-union-per-il-polimorfismo) (6:04)
      - [3.2. Campi Essenziali di `tfobj`](#32-campi-essenziali-di-tfobj) (7:44)
  - [4. Gestione della Memoria e Funzioni di Allocazione](#4-gestione-della-memoria-e-funzioni-di-allocazione) (15:50 - 21:05)
      - [4.1. L'Involucro di Sicurezza `xmalloc`](#41-linvolucro-di-sicurezza-xmalloc) (17:35)
      - [4.2. Definizione dei Tipi di Oggetto](#42-definizione-dei-tipi-di-oggetto) (19:22)
  - [5. Implementazione delle Funzioni di Creazione Oggetti](#5-implementazione-delle-funzioni-di-creazione-oggetti) (24:00 - 37:43)
      - [5.1. `createObject` (Base)](#51-createobject-base) (26:05)
      - [5.2. `createIntObject` e `createSymbolObject`](#52-createintobject-e-createsymbolobject) (32:38)
      - [5.3. `createListObject`](#53-createlistobject) (37:07)

-----

## 1\. Filosofia di Design e Contesto di Sviluppo (0:40 - 2:20)

**Obiettivo:** Realizzare qualcosa di non banale con gli strumenti di programmazione C acquisiti (strutture, puntatori, manipolazione della memoria). L'interprete è il banco di prova ideale.

**Linguaggio Scelto:** Una versione giocattolo di **Forth** (Toy Forth), un linguaggio noto per la sua semplicità e la sua natura **stack-based**.

## 2\. Introduzione a Forth e Architettura Stack-Based (1:55 - 4:40)

**Forth** è un linguaggio che opera interamente manipolando uno **stack** di dati. Un'operazione preleva i dati necessari dallo stack e vi ripone il risultato.

### 2.1. Esempio di Programma Forth (2:02)

Il programma base `5 dup * print` illustra il funzionamento.

```text
1. 5             // Push 5. Stack: [5]
2. dup           // Duplica il valore in cima. Stack: [5, 5]
3. * // Pop (5, 5). Moltiplica 5*5=25. Push 25. Stack: [25]
4. print         // Pop 25. Stampa a video.
```

Questa architettura impone che l'interprete abbia una struttura per lo stack e una rappresentazione uniforme di tutti i tipi di dati.

-----

## 3\. Definizione della Struttura Dati Centrale: `tfobj` (4:40 - 15:40)

La struttura `tfobj` (Toy Forth Object) è la rappresentazione in C di tutti i tipi di dato che il linguaggio gestirà.

### 3.1. Uso della `union` per il Polimorfismo (6:04)

Perché un singolo oggetto possa rappresentare sia un intero, sia una stringa, sia una lista, si usa la `union`. Questo è un meccanismo tipico di C per implementare un **tipo polimorfico** come `PyObject` in Python o `redisObject` in Redis.

```c
// Definizione dei tipi di TFObj. Usiamo enum/define per il campo 'type' (19:22)
enum {
    TFOBJ_TYPE_INT, 
    TFOBJ_TYPE_SYM, 
    TFOBJ_TYPE_LIST,
    // ... altri
};

// Struttura tfobj: il cuore dell'interprete (6:04)
typedef struct tfobj {
    size_t type;            // Indica il tipo di dato contenuto (8:10)
    int ref_count;          // Conteggio delle referenze (per il GC manuale) (9:45)
    union {                 // Dati effettivi, uno esclude l'altro (6:59)
        long i;             // Valore per TFOBJ_TYPE_INT (7:44)
        char *s;            // Puntatore alla stringa per simbolo o stringa (7:50)
        struct {
            struct tfobj **ele; // Array di puntatori ad oggetti per LISTA/STACK (7:56)
            size_t len;
        } list;
    };
} tfobj;
```

### 3.2. Campi Essenziali di `tfobj` (7:44)

  * **`size_t type` (Tipo)**: Un campo esplicito per sapere quale membro della `union` è valido in quel momento.
  * **`int ref_count` (Reference Counting)**: Un contatore per la gestione manuale della memoria (simile ad ARC o `shared_ptr` semplificato), essenziale in C. Quando un oggetto non è più referenziato, può essere liberato.
  * **`union` (Dati)**:
      * `long i`: Per i numeri interi.
      * `char *s`: Per i simboli (nome delle funzioni) o stringhe.
      * `list`: Struttura per liste e stack (Array Dinamico).

-----

## 4\. Gestione della Memoria e Funzioni di Allocazione (15:50 - 21:05)

### 4.1. L'Involucro di Sicurezza `xmalloc` (17:35)

In programmazione di sistema, le chiamate di sistema (come `malloc`) sono spesso avvolte in **funzioni helper** per aggiungere funzionalità come l'**exit in caso di errore** o la diagnostica. `xmalloc` (e `xfree` e `xrealloc`) sono standard in molti progetti C.

```c
// xmalloc: Allocazione sicura che esce dal programma in caso di fallimento (17:35)
void *xmalloc(size_t size) {
    void *p = malloc(size); // 17:50
    if (p == NULL) {
        fprintf(stderr, "Fatal error: Out of memory (xmalloc(%zu))\n", size);
        exit(1); // Esci con errore
    }
    return p;
}
```

### 4.2. Definizione dei Tipi di Oggetto (19:22)

Per rendere il codice più leggibile e robusto, si definiscono costanti per i tipi di oggetto.

```c
// Definizioni per il campo 'type' (19:22)
#define TFOBJ_TYPE_INT  0
#define TFOBJ_TYPE_SYM  1
#define TFOBJ_TYPE_LIST 2
// ...
```

-----

## 5\. Implementazione delle Funzioni di Creazione Oggetti (24:00 - 37:43)

Le funzioni `createXObject` sono wrapper che allocano e inizializzano correttamente i campi di base di un `tfobj`.

### 5.1. `createObject` (Base) (26:05)

Questa è la funzione base che alloca l'oggetto e ne imposta i campi fondamentali (tipo e `ref_count`).

```c
// createObject: Alloca la memoria per un nuovo oggetto e imposta i campi base (26:05)
tfobj *createObject(size_t type) {
    tfobj *o = xmalloc(sizeof(*o)); // 26:22. (*o) garantisce la dimensione corretta

    // Inizializzazione standard
    o->type = type; // 26:35
    o->ref_count = 1; // Un oggetto appena creato ha sempre 1 referenza (26:40)

    // Inizializzazione specifica per la union (opzionale, ma buona pratica)
    // memset(&o->list, 0, sizeof(o->list)); // 26:50 - Azzerare la union

    return o;
}
```

### 5.2. `createIntObject` e `createSymbolObject` (32:38)

Le funzioni specializzate usano `createObject` e impostano il campo specifico della `union`.

```c
// createIntObject: Crea un oggetto di tipo intero (33:05)
tfobj *createIntObject(long value) {
    tfobj *o = createObject(TFOBJ_TYPE_INT);
    o->i = value; // Imposta il campo 'i' della union (33:23)
    return o;
}

// createSymbolObject: Crea un oggetto di tipo simbolo (35:10)
tfobj *createSymbolObject(char *s) {
    tfobj *o = createObject(TFOBJ_TYPE_SYM);
    // [FIXME: Bisogna copiare la stringa 's' per evitare problemi di vita/morte (35:28)]
    o->s = s; // Assegnazione diretta (temporanea, richiede copia in un'implementazione seria)
    return o;
}
```

### 5.3. `createListObject` (37:07)

Crea un oggetto lista/stack, inizializzandolo come **vuoto**.

```c
// createListObject: Crea un oggetto di tipo lista (37:07)
tfobj *createListObject(void) {
    tfobj *o = createObject(TFOBJ_TYPE_LIST);
    
    // Inizializza la struttura interna 'list' (37:20)
    o->list.ele = NULL; // Array di elementi inizialmente a NULL (37:31)
    o->list.len = 0;    // Lunghezza zero (37:35)

    return o;
}
```