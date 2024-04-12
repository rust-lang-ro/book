## Executarea codului în etapa de curățare cu trăsătura `Drop`

A doua trăsătură esențială pentru design-ul pointerilor inteligenți este `Drop`, care îți oferă posibilitatea de a personaliza acțiunile ce au loc atunci când o valoare va ieși din domeniul de vizibilitate. Ai posibilitatea să implementezi trăsătura `Drop` pentru orice tip, iar codul respectiv poate fi utilizat pentru a elibera resurse, cum ar fi fișiere sau conexiuni de rețea.

Vorbim despre `Drop` în contextul pointerilor inteligenți pentru că, de obicei, funcționalitatea asociată cu trăsătura `Drop` este folosită în cadrul implementării unui pointer inteligent. De exemplu, atunci când un `Box<T>` este descărcat, el va dealoca spațiul pe heap la care se referă acesta.

În alte limbaje de programare, pentru anumite tipuri de date, dezvoltatorul trebuie să execute manual cod pentru a elibera memoria sau resursele de fiecare dată când termină de utilizat o instanță a acestor tipuri. Sunt incluse cazuri precum descriptorii de fișiere, socket-uri sau blocările de resurse. Dacă ar omite, sistemul ar putea deveni supraîncărcat și ar putea cădea. În Rust, poți specifica un anumit segment de cod care să fie rulat când o valoare părăsește domeniul de vizibilitate, iar compilatorul va insera acest segment de cod automat. Astfel, nu ești nevoit să introduci cod de curățare oriunde într-un program doar pentru că ai terminat de folosit o instanță de un anumit tip — și nu vei pierde resurse!

Specifici codul care urmează să fie executat când o valoare iese din domeniul de vizibilitate implementând trăsătura `Drop`. `Drop` necesită implementarea unei metode denumite `drop` care primește o referință mutabilă către `self`. Pentru a vedea momentul în care Rust invocă `drop`, să implementăm momentan metoda `drop` cu instrucțiuni `println!`.

Listarea 15-14 prezintă structura `CustomSmartPointer` care, prin unica ei funcționalitate particulară, va afișa mesajul `Dropping CustomSmartPointer!` la ieșirea instanței din domeniul de vizibilitate, demonstrând astfel momentul în care Rust execută funcția `drop`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

<span class="caption">Listarea 15-14: Structura `CustomSmartPointer` implementând trăsătura `Drop`, unde am plasa codul nostru de curățare</span>

Trăsătura `Drop` este inclusă în preludiu, prin urmare, nu este nevoie să o facem vizibilă în domeniul de aplicabilitate. Implementăm trăsătura `Drop` pe `CustomSmartPointer` și oferim o implementare pentru metoda `drop` care invocă macro-ul `println!`. În corpul funcției `drop` ai include logica pe care dorești să o executi când o instanță a tipului tău este pe cale să iasă din domeniul de vizibilitate. În exemplul nostru, afișăm un text pentru a arăta vizual momentul la care Rust va chema `drop`.

În funcția `main`, construim două instanțe de `CustomSmartPointer` și apoi afișăm `CustomSmartPointers created`. La sfârșitul `main`, instanțele noastre de `CustomSmartPointer` vor ieși din domeniu de vizibilitate, și Rust va chema codul pe care l-am pus în metoda `drop`, imprimând mesajul final. Este de notat că nu e necesar să chemăm metoda `drop` în mod direct.

Când rulăm acest program, se va afișa următorul rezultat:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust a chemat automat `drop` pentru noi când instanțele noastre au ieșit din domeniu de aplicabilitate, executând codul pe care l-am definit. Variabilele sunt eliberate în ordinea inversă creării lor, așadar `d` a fost eliberată înaintea lui `c`. Scopul acestor exemple este să îți ofere o ilustrare vizuală a modului în care acționează metoda `drop`; în mod normal ai seta codul de curățare necesar tipului tău, în loc de un mesaj tipărit.

### Eliberarea anticipată a unei valori folosind `std::mem::drop`

Din păcate, nu este un proces simplu să dezactivezi funcționalitatea automată de `drop`. De altfel, dezactivarea `drop` nu este necesară de obicei; esența trăsăturii `Drop` este aceea că este gestionată în mod automat. Totuși, uneori s-ar putea să vrei să eliberezi o valoare mai devreme. Un exemplu ar fi utilizarea pointerilor inteligenți care controlează lock-uri (instrucțiunea *lock* e o directivă de bază a programării concurente): s-ar putea să vrei să forțezi metoda `drop` care eliberează un lock, astfel încât alt cod din același domeniu de vizibilitate să-l poată prelua. Rust nu îți permite să apelezi manual metoda `drop` a trăsăturii `Drop`; în loc de aceasta trebuie să folosești funcția `std::mem::drop` oferită de biblioteca standard, când dorești să forțezi eliberarea unei valori înainte de terminarea domeniului său de vizibilitate.

Dacă încercăm să apelăm manual metoda `drop` a trăsăturii `Drop` prin modificarea funcției `main` din Listarea 15-14, așa cum este arătat în Listarea 15-15, vom întâmpina o eroare de compilare:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

<span class="caption">Listarea 15-15: Încercarea de a invoca manual metoda `drop` din trăsătura `Drop` pentru a realiza o curățare prematură</span>

Când încercăm să compilăm acest cod, vom primi următoarea eroare:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

Acest mesaj de eroare ne spune că nu ne este permis să apelăm explicit `drop`. Mesajul de eroare folosește termenul *destructor*, ceea ce în terminologia programării se referă la o funcție care face curățenie după o instanță. *Destructorul* este analog cu *constructorul*, care inițiază o instanță. Funcția `drop` din Rust este un exemplu de destructor.

Rust nu ne permite să apelăm metoda `drop` în mod explicit pentru că Rust ar apela automat metoda `drop` pentru valoarea respectivă la terminarea funcției `main`. Acest lucru ar duce la o eroare de *eliberare dublă* (double free), deoarece Rust ar încerca să curețe aceeași valoare de două ori.

Nu putem dezactiva inserția automată a metodei `drop` atunci când o valoare iese din domeniu de vizibilitate, și nici să apelăm explicit metoda `drop`. Astfel, dacă dorim să forțăm curățarea unei valori mai devreme, trebuie să utilizăm funcția `std::mem::drop`.

Funcția `std::mem::drop` diferă de metoda `drop` din trăsătura `Drop`. Pentru a o apela, pasăm ca argument valoarea pe care vrem să o ștergem forțat. Această funcție este inclusă în preludiu, astfel că putem modifica funcția `main` din Listarea 15-15 pentru a apela funcția `drop`, așa cum se arată în Listarea 15-16:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

<span class="caption">Listarea 15-16: Apelând funcția `std::mem::drop` pentru a elibera explicit o valoare înainte de a ieși din domeniu de vizibilitate</span>

Executând acest cod va genera următoarea afișare:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

Textul ```'Dropping CustomSmartPointer with data `some data`!'``` este afișat între `'CustomSmartPointer created.'` și `'CustomSmartPointer dropped before the end of main.'`, indicând faptul că codul metodei `drop` este executat pentru a elibera `c` la acel moment.

Putem utiliza codul specificat într-o implementare a trăsăturii `Drop` în diverse moduri pentru a asigura o curățare convenabilă și sigură: de exemplu, am putea să îl folosim pentru a dezvolta un propriul nostru alocator de memorie! Beneficiind de trăsătura `Drop` și de sistemul de posesiune al limbajului Rust, nu trebuie să ne amintim să realizăm curățarea deoarece Rust o face în mod automat.

De asemenea, nu trebuie să ne îngrijorăm referitor la problemele care pot apărea din cauza eliberării accidentale a valorilor încă în folosință: sistemul de posesiune, care se asigură în permanență că referințele sunt valide, mai și garantează că `drop` este apelat doar o singură dată, când valoarea nu mai este utilizată.

Acum, după ce am investigat `Box<T>` și unele din caracteristicile pointerilor inteligenți, să explorăm câțiva alți pointeri inteligenți definiți în biblioteca standard.
