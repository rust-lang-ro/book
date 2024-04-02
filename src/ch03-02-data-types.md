## Tipuri de date

Fiecare valoare în Rust este de un anumit *tip de date*, care indică Rust ce fel de
date sunt specificate pentru a ști cum să lucreze cu acele date. Vom analiza
două subansamble de tipuri de date: scalar și compus.

Amintiți-vă că Rust este un limbaj *cu tipizare statică*, ceea ce înseamnă că trebuie
să cunoaște tipurile tuturor variabilelor la momentul compilării. În general, compilatorul poate infera ce tip dorim să folosim în funcție de valoare și cum o utilizăm. În cazurile când sunt posibile mai multe tipuri, cum ar fi atunci când am convertit un `String` într-un tip numeric folosind `parse` în secțiunea [„Comparând guess-ul cu numărul secret”][comparing-the-guess-to-the-secret-number]<!-- ignore --> din Capitolul 2, trebuie să adăugăm o adnotare de tip, astfel:

```rust
let guess: u32 = "42".parse().expect("Nu este un număr!");
```

Dacă nu adăugăm adnotarea de tip `: u32` arătată în codul precedent, Rust va afișa următoarea eroare, ceea ce înseamnă că compilatorul are nevoie de mai multe informații de la noi pentru a ști ce tip dorim să folosim:

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

Vei vedea adnotări de tip diferite pentru alte tipuri de date.

### Tipuri scalare

Un *tip scalar* reprezintă o singură valoare. Rust are patru tipuri scalare principale: numere întregi, numere în virgulă mobilă, Booleani și caractere. S-ar putea ca acestea să ți se pară familiare din alte limbaje de programare. Să trecem la modul în care funcționează tipurile scalabile în Rust.

#### Tipuri de întregi

Un *întreg* este un număr fără componentă fracționară. Am folosit un tip de întreg în Capitolul 2, tipul `u32`. Această declarație de tip indică faptul că valoarea cu care este asociată ar trebui să fie un întreg fără semn (tipurile de întregi cu semn încep cu `i` în loc de `u`) care ocupă 32 de biți de spațiu. Tabelul 3-1 arată tipurile de întregi integrate în Rust. Putem folosi fiecare dintre aceste variante pentru a declara tipul unei valori întregi.

<span class="caption">Tabelul 3-1: Tipuri de întregi în Rust</span>

| Lungime  | Cu semn | Fără semn |
|----------|---------|-----------|
| 8-biți   | `i8`    | `u8`      |
| 16-biți  | `i16`   | `u16`     |
| 32-biți  | `i32`   | `u32`     |
| 64-biți  | `i64`   | `u64`     |
| 128-biți | `i128`  | `u128`    |
| arhitect.| `isize` | `usize`   |

Fiecare variantă poate fi fie cu semn, fie fără semn, și are o dimensiune explicită. *Cu semn* și *fără semn* se referă la faptul dacă este posibil ca numărul să fie negativ - cu alte cuvinte, dacă numărul are nevoie să aibă un semn asociat (signed) sau dacă va fi mereu pozitiv și deci poate fi reprezentat fără un semn (unsigned). Este similar cu scrierea numerelor pe hârtie: când semnul contează, un număr este prezentat cu un semn plus sau un semn minus; totuși, când este sigur că numărul este pozitiv, acesta este prezentat fără semn. Numerele cu semn sunt stocate folosind reprezentarea de [complement față de doi][twos-complement]<!-- ignore --> .

Fiecare variantă cu semn poate stoca numere de la -(2<sup>n - 1</sup>) la 2<sup>n - 1</sup> - 1 inclusiv, unde *n* este numărul de biți pe care îl folosește acea variantă. Așadar un `i8` poate stoca numere de la -(2<sup>7</sup>) la 2<sup>7</sup> - 1, care este egal cu -128 până la 127. Variantele fără semn pot stoca numere de la 0 la 2<sup>n</sup> - 1, așadar un `u8` poate stoca numere de la 0 la 2<sup>8</sup> - 1, care este egal cu 0 până la 255.

În plus, tipurile `isize` si `usize` depind de arhitectura computerului pe care rulează programul, care este notată în tabelul de mai sus ca „arhitectura sistemului”: 64 de biți dacă te afli pe o arhitectură de 64-biți și 32 de biți dacă te afli pe o arhitectură de 32-biți.

Poți scrie literali întregi în oricare dintre formele prezentate în Tabelul 3-2. Observă că literali numerici care pot fi de mai multe tipuri numerice permit un sufix de tip, cum ar fi `57u8`, pentru a desemna tipul. Literalii numerici pot de asemenea folosi `_` ca separator vizual pentru a face numărul mai ușor de citit, precum `1_000`, care va avea aceeași valoare ca și cum ai fi specificat `1000`.

<span class="caption">Tabelul 3-2: Literali întregi în Rust</span>

| Literali numerici | Exemplu       |
|-------------------|---------------|
| Zecimal           | `98_222`      |
| Hexadecimal       | `0xff`        |
| Octal             | `0o77`        |
| Binar             | `0b1111_0000` |
| Byte (doar `u8`)  | `b'A'`        |

Deci, cum să știi ce tip de întreg să folosești? Dacă nu ești sigur, valorile setate inițial de Rust sunt în general un punct bun de plecare: tipurile întregi au implicit setat `i32`. Principala situație în care ai folosi `isize` sau `usize` este atunci când indexezi o colecție de elemente.

> ##### Depășirea întregilor
>
> Să presupunem că ai o variabilă de tip `u8` care poate stoca valori între 0
> și 255. Dacă încerci să schimbi valoarea variabilei în afara acestui
> interval, cum ar fi 256, va apărea o *depășire a întregilor* (integer
> overflow), care poate produce unul dintre două comportamente. Atunci când
> compilezi în modul de depanare, Rust include verificări pentru depășirea
> întregilor care cauzează *panica* programului la runtime dacă acest
> comportament apare. Rust utilizează termenul de *panică* când un program se
> închide cu o eroare; vom discuta despre panici în profunzime în secțiunea
> [„Erorile irecuperabile cu `panic!`”][unrecoverable-errors-with-panic] în
> Capitolul 9. 
>
> Când compilezi în modul de release cu opțiunea `--release`, Rust *nu* include
> verificări pentru depășirea întregilor care cauzează panici. În schimb,
> dacă apare o depășire, Rust efectuează *împachetarea complementul lui doi*.
> Pe scurt, valorile mai mari decât valoarea maximă pe care o poate deține
> tipul „este împachetat” la minimul valorilor pe care tipul le poate deţine.
> În cazul unui `u8`, valoarea 256 devine 0, valoarea 257 devine 1 și așa mai
> departe. Programul nu va genera panică, dar variabila va avea o valoare care
> probabil nu este ceea ce te așteptai să aibă. Să te bazezi pe comportamentul
> de împachetare a întregilor depășiți este considerat greșit.
>
> Pentru a gestiona în mod explicit posibilitatea de depășire, poți folosi
> aceste familii de metode oferite de biblioteca standard pentru tipurile
> numerice primitive:
>
> * Împachetează toate operațiunile cu metodele `wrapping_*`, cum ar fi
>   `wrapping_add`.
> * Întoarce valoarea `None` dacă există depășire cu metodele
>   `checked_*`.
> * Întoarce valoarea și un boolean care indică dacă a existat depășire
>   cu metodele `overflowing_*`.
> * Saturează la valorile minime sau maxime ale valorii cu metodele
>   `saturating_*`.

#### Tipurile cu virgulă mobilă

Rust are, de asemenea, două tipuri primitive pentru *numerele în virgulă mobilă*, care sunt numerele cu puncte zecimale. Tipurile Rust cu virgulă mobilă sunt `f32` și `f64`, care au 32 de biți și 64 de biți în dimensiune, respectiv. Tipul implicit este `f64` deoarece pe procesoarele moderne este aproximativ la fel de rapid ca `f32`, dar este capabil de mai multă precizie. Toate tipurile cu virgulă mobilă sunt semnate.

Iată un exemplu care arată numerele cu virgulă mobilă în acțiune:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

Numerele cu virgulă mobilă sunt reprezentate conform standardului IEEE-754. Tipul `f32` este un număr cu precizie simplă, iar `f64` are precizie dublă.

#### Operațiuni numerice

Rust suportă operațiunile matematice de bază pe care le-ai aștepta pentru toate tipurile de numere: adunare, scădere, înmulțire, împărțire și rest. Împărțirea întreagă trunchiază spre zero la cel mai apropiat număr întreg. Următorul cod arată cum ai folosi fiecare operațiune numerică într-o declarație `let`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

Fiecare expresie în aceste declarații folosește un operator matematic și evaluează la o singură valoare, care este apoi legată la o variabilă. [Anexa B][appendix_b]<!-- ignore --> conține o listă cu toți operatorii pe care Rust îi pune la dispoziție.

#### Tipul Boolean

Ca în majoritatea celorlalte limbaje de programare, un tip Boolean în Rust are două posibile valori: `true` și `false`. Booleanii au dimensiunea de un octet. Tipul Boolean în Rust este specificat folosind `bool`. De exemplu:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

Principalul mod de a folosi valorile Boolean este prin condiționale, cum ar fi o expresie `if`. Vom discuta cum funcționează expresiile `if` în Rust în secțiunea [„Fluxul de control”][control-flow]<!-- ignore -->.

#### Tipul caracter

Tipul `char` al lui Rust este cel mai primitiv tip alfabetic al limbajului. Aici avem câteva exemple de declarare a valorilor `char`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

Observați că specificăm constantele de tip `char` cu ghilimele simple, spre deosebire de sirurile de caractere, care utilizează ghilimele duble. Tipul `char` al lui Rust este de patru octeți și reprezintă o valoare scalară Unicode, ceea ce înseamnă că poate reprezenta mult mai mult decât doar ASCII. Litere accentuate; caractere chinezești, japoneze și coreene; emoji și spații de lățime zero sunt toate valori `char` valide în Rust. Valorile scalare Unicode variază de la `U+0000` la `U+D7FF` și `U+E000` la `U+10FFFF` inclusiv. Totuși, un „caracter” nu este chiar un concept în Unicode, așa că intuiția ta umană pentru ceea ce este un „caracter” s-ar putea să nu se potrivească cu ceea ce este un `char` în Rust. Vom discuta acest subiect în detaliu în [„Stocarea textului codificat UTF-8 cu string-uri”][strings]<!-- ignore --> în Capitolul 8.

### Tipuri compuse

*Tipurile compuse* pot grupa mai multe valori într-un singur tip. Rust are două tipuri compuse primitive: tuple și array-uri.

#### Tipul tuplă

O *tuplă* este o modalitate generală de a grupa împreună un număr de valori cu o varietate de tipuri într-un singur tip compus. Tuplele au o lungime fixă: odată declarate, nu pot crește sau scădea în dimensiune.

Creăm o tuplă scriind o listă de valori separate prin virgulă între paranteze. Fiecare poziție din tuplă are un tip, și tipurile diferitelor valori din tuplă nu trebuie să fie aceleași. Am adăugat în acest exemplu opțional adnotări de tip:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

Variabila `tup` se leagă de întreaga tuplă pentru că o tuplă este considerată un singur element compus. Pentru a obține valorile individuale dintr-o tuplă, putem folosi potrivirea de modele pentru a destrucura o valoare a tuplei, așa cum este arătat aici:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

Acest program creează mai întâi o tuplă și o leagă la variabila `tup`. Apoi folosește un model cu `let` pentru a lua `tup` și a o transforma în trei variabile separate, `x`, `y`, și `z`. Acest lucru este numit *destructurare* pentru că descompune singura tuplă în trei părți. În final, programul afișează valoarea lui `y`, care este `6.4`.

Putem accesa direct un element al tuplei folosind un punct (`.`) urmat de indexul valorii pe care dorim să o accesăm. De exemplu:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

Acest program creează tupla `x` și apoi accesează fiecare element al tuplei folosind indicii lor respectivi. La fel ca în majoritatea limbajelor de programare, primul index într-o tuplă este 0.

Tupla fără nicio valoare are un nume special, *unit*. Această valoare și tipul său corespunzător sunt ambele scrise `()` și reprezintă o valoare goală sau un tip de returnare gol. Expresiile returnează implicit valoarea unit dacă nu returnează nicio altă valoare.

#### Tipul array

Un alt mod de a avea o colecție de valori multiple este cu un *array*. Spre deosebire de o tuplă, fiecare element al unui array trebuie să aibă același tip. Spre deosebire de array-urile din alte limbaje, array-urile din Rust au o lungime fixă.

Scriem valorile într-un array ca o listă separată prin virgule în interiorul parantezelor pătrate:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

Array-urile sunt utile când vrem ca datele să fie alocate pe stivă, nu pe heap (vom discuta mai mult despre stivă și heap în [Capitolul 4][stack-and-heap]<!-- ignore -->) sau când dorim să ne asigurăm că avem întotdeauna un număr fix de elemente. Un array nu este la fel de flexibil precum tipul vector, totuși. Un *vector* este un tip de colecție similar oferit de biblioteca standard dar care *are voie* să crească sau să scadă în dimensiune. Dacă nu ești sigur dacă să folosești un array sau un vector, atunci probabil că ar trebui să folosești un vector. [Capitolul 8][vectors]<!-- ignore --> discută vectorii în mai multe detalii.

Totuși, array-urile sunt mai utile când știi că numărul de elemente nu va avea nevoie să se schimbe. De exemplu, dacă ai folosi numele lunilor într-un program, ai folosi probabil un array în loc de un vector, deoarece știi că acesta va conține întotdeauna 12 elemente:

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Scrii tipul unui array folosind paranteze pătrate cu tipul fiecărui element, un punct și virgulă, și apoi numărul de elemente din array, așa:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

Aici, `i32` este tipul fiecărui element. După punctul și virgula, numărul `5` indică faptul că array-ul conține cinci elemente.

Poți de asemenea inițializa un array pentru a conține aceeași valoare pentru fiecare element, specificând valoarea inițială, urmată de un punct și virgulă, și apoi lungimea array-ului în paranteze pătrate, așa cum este afișat aici:

```rust
let a = [3; 5];
```

Array-ul numit `a` va conține `5` elemente care vor fi inițializate toate cu valoarea `3`. Acest lucru este la fel ca scrierea `let a = [3, 3, 3, 3, 3];`, dar într-un mod mai concis.

##### Accesarea elementelor unui array

Un array este un singur segment de memorie de o dimensiune cunoscută, fixă, care poate fi alocat pe stiva. Poți accesa elementele unui array folosind indexarea, în acest fel:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

În acest exemplu, variabila numită `first` va primi valoarea `1` pentru că aceasta
este valoarea la indexul `[0]` în array. Variabila numită `second` va primi
valoarea `2` de la indexul `[1]` în array.

##### Acces invalid la un element al unui array

Să vedem ce se întâmplă dacă începi să accesezi un element al unui array care este după sfârșitul array-ului. Să zicem că rulezi acest cod, asemănător cu jocul de ghicit din Capitolul 2, pentru a obține un index de array de la utilizator:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

Cod se compilează cu succes. Dacă rulezi acest cod folosind `cargo run` și introduci `0`, `1`, `2`, `3`, sau `4`, programul va afișa valoarea corespunzătoare acelui index în array. Dacă introduci un număr după sfârșitul array-ului, cum ar fi `10`, vei vedea o ieșire de acest gen:

```console
thread 'main' panicked at 'index out of bounds: the len is 5 but the index is 10', src/main.rs:19:19
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Programul a rezultat într-o eroare de *runtime* în momentul utilizării unei valori invalide în operația de indexare. Programul se închide cu un mesaj de eroare și nu execută instrucțiunea finală `println!`. Când încerci să accesezi un  element folosind indexarea, Rust va verifica dacă indexul specificat de tine este mai mic decât lungimea array-ului. Dacă indexul este mai mare sau egal cu lungimea, Rust va stopa cu panică. Verificarea dată trebuie să se întâmple la runtime, în special în așa caz, pentru că compilatorul nu poate ști cu siguranță ce valoare va introduce un utilizator când va rula codul mai târziu.

Acesta este un exemplu al principiilor de siguranță ale memoriei din Rust în acțiune. În multe limbaje de nivel scăzut, verificare de acces în array nu se face, și, când furnizezi un index incorect, poate fi accesată memorie invalidă. Rust te protejează împotriva acestui fel de erori prin ieșirea imediată în loc de a permitere accesul la memorie și continuare. Capitolul 9 discută mai multe despre gestionarea erorilor în Rust și cum poți să scrii un cod lizibil și sigur, care nu face niciun fel de panică nici nu permite acces la memorie invalidă.

[comparing-the-guess-to-the-secret-number]:
ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md
