## Utilizarea firelor de execuție pentru a executa cod simultan

În cele mai multe sisteme de operare actuale, codul unui program executat este rulat în cadrul unui *proces*, iar sistemul de operare gestionează în același timp multiple procese. În interiorul unui program, poți avea de asemenea părți independente care rulează simultan. Capabilitățile care execută aceste părți independente se numesc *fire de execuție*, or *thread-uri*. De exemplu, un server web poate avea mai multe thread-uri pentru a răspunde simultan la mai multe cereri.

Împărțirea calculului din programul tău în mai multe fire de execuție pentru a rula simultan mai multe sarcini poate îmbunătăți performanța, dar adaugă și complexitate. Fiindcă thread-urile pot rula simultan, nu există o garanție implicită asupra ordinii în care secțiunile de cod de pe diferite thread-uri se vor executa. Aceasta poate cauza probleme, cum ar fi:

* Condiții de cursă (Race conditions), când thread-urile accesează date sau resurse într-un ordine neconsistentă
* Interblocaje (Deadlocks), în situația unde două thread-uri așteaptă unul pe celălalt, împiedicând astfel continuarea ambelor thread-uri
* Erori care apar numai în anumite condiții și sunt dificil de reprodus și de reparat în mod fiabil

Rust încearcă să minimizeze efectele negative ale utilizării thread-urilor, însă programarea într-un context multithreaded necesită o gândire meticuloasă și impune o structură a codului diferită de cea a programelor rulate într-un singur fir.

Limbajele de programare implementează thread-uri în diferite moduri, iar multe sisteme de operare furnizează un API pe care limbajul îl poate folosi pentru a crea noi thread-uri. Biblioteca standard Rust utilizează un model de implementare *1:1* pentru thread-uri, prin care un program folosește un thread de sistem de operare pentru fiecare thread de limbaj. Există crate-uri care pun în practică alte modele de threading ce oferă compromisuri diferite comparativ cu modelul 1:1.

### Crearea unui fir nou de execuție cu `spawn`

Pentru a crea un fir nou de execuție, apelăm funcția `thread::spawn` la care pasăm o închidere (discutate în Capitolul 13) care conține codul ce urmează să fie executat în nou-creatul fir de execuție. Exemplul din Listarea 16-1 afișează un text din firul principal și alt text dintr-un fir nou-creat:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

<span class="caption">Listarea 16-1: Crearea unui nou fir de execuție pentru a printa ceva, în timp ce firul principal printează altceva</span>

Notăm că atunci când firul principal al unui program Rust se finalizează, toate firele secundare care au fost create sunt închise, indiferent dacă și-au finisat execuția sau nu. Rezultatul executării acestui program poate varia la fiecare rulare, dar va arăta similar cu următoarea afișare:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

Apelurile la `thread::sleep` determină un fir de execuție să își întrerupă temporar execuția, ceea ce permite altui fir să preia controlul. Este posibil ca firele să se alterneze, dar nu este garantat, deoarece acest lucru depinde de cum sistemul de operare gestionează secvențierea firelor de execuție. În această execuție, firul principal a făcut prima afișare, chiar dacă instrucțiunea de printare din firul secundar apare prima în cod. Chiar dacă i s-a specificat firului secundar să continue printarea până când `i` ajunge la 9, acesta a reușit doar să ajungă la 5 înainte ca firul principal să fie închis.

În cazul în care executând acest cod observi doar rezultate din partea firului principal, sau nu există nicio suprapunere, încearcă să extinzi valorile maxime pentru a oferi mai multe șanse sistemului de operare să alterneze între firele de execuție.

### Așteptarea finalizării tuturor firelor de execuție folosind `join`

Codul din Listarea 16-1 nu doar că oprește prematur firul de execuție creat din cauza terminării firului principal, dar dat fiind faptul că nu există o garanție cu privire la ordinea execuției firelor de execuție, de asemenea nu putem asigura că firul de execuție creat va rula în genere!

Problema firului de execuție creat care nu rulează sau care se încheie prematur poate fi rezolvată salvând valoarea returnată de `thread::spawn` într-o variabilă. Tipul returnat de `thread::spawn` este un descriptor `JoinHandle`. Un `JoinHandle` este o valoare deținută (owned) care, la apelarea metodei `join` pe aceasta, va aștepta finalizarea firului de execuție asociat. Listarea 16-2 ilustrează utilizarea lui `JoinHandle` pentru firul de execuție creat în Listarea 16-1 și invocarea lui `join` pentru a ne asigura că firul de execuție inițiat se finalizează înainte de terminarea funcției `main`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

<span class="caption">Listarea 16-2: Salvând un `JoinHandle` de la `thread::spawn` pentru a asigura finalizarea completă a firului de execuție</span>

Apelarea lui `join` pe descriptor blochează firul de execuție curent până când firul reprezentat de acel descriptor se termină. *Blocarea* unui fir de execuție împiedică acesta să efectueze sarcini sau să se termine. De aceea, punând apelul la `join` după bucla `for` a main thread-ului, executarea Listării 16-2 ar trebui să genereze un afișaj similar cu acesta:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Ambele fire de execuție continuă să alterneze, însă firul principal așteaptă datorită apelului metodei `join` pe descriptor și nu se va încheia până nu se finalizează firul de execuție inițiat.

Acum, să analizăm ce se întâmplă dacă mutăm `handle.join()` înainte de bucla `for` în funcția `main`, în felul următor:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

Thread-ul principal va aștepta finalizarea firului de execuție inițiat și apoi va executa propria buclă `for`, astfel încât afișajul nu va mai fi intercalat, după cum putem vedea aici:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Detalii subtile, precum locația unde este invocat `join`, pot influența dacă firele de execuție se vor rula simultan sau nu.

### Folosirea închiderilor `move` cu fire de execuție

Frecvent, utilizăm cuvântul cheie `move` cu închiderile transmise la `thread::spawn`, deoarece în acest mod închiderea preia posesiunea asupra valorilor utilizate din mediu, transferând astfel posesiunea acestor valori de la un fir de execuție la altul. În secțiunea [„Capturarea referințelor sau transferul posesiunii”][capture] din Capitolul 13, am discutat despre `move` în contextul închiderilor. Acum, vom aprofunda interacțiunea dintre `move` și `thread::spawn`.

În Listarea 16-1 putem observa că închiderea pe care o trecem la `thread::spawn` nu are argumente: nu utilizăm date din firul de execuție principal în codul firului de execuție nou. Pentru a folosi date din firul principal în cel nou, închiderea firului de execuție trebuie să captureze valorile necesare. Listarea 16-3 ne arată o tentativă de a folosi un vector creat în firul de execuție principal într-un fir secundar. Cu toate acestea, momentan acest lucru nu funcționează, cum vom vedem mai jos.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

<span class="caption">Listarea 16-3: Încercarea de utilizare a unui vector creat în firul de execuție principal într-un alt fir de execuție</span>

Închiderea utilizează `v`, așadar va captura `v` și o va integra în mediul închiderii. Deoarece `thread::spawn` execută această închidere într-un fir de execuție nou, teoretic ar trebui să fie posibilă accesarea variabilei `v` în acest fir. Dar când încercăm să compilăm exemplul, întâmpinăm eroarea următoare:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust face *inferențe* cu privire la modul de capturare a lui `v`, și pentru că macro-ul `println!` necesită doar o referință la `v`, închiderea încearcă să împrumute `v`. Există, însă, o dificultate: Rust nu poate determina durata de execuție a firului nou, deci nu poate garanta că referința la `v` va rămâne validă.

Listarea 16-4 ilustrează un scenariu în care este improbabil ca referința la `v` să fie validă:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

<span class="caption">Listarea 16-4: Un fir de execuție cu o închidere care încearcă să capteze o referință la `v` din firul de execuție principal care apoi renunță la `v`</span>

Dacă Rust ar permite execuția acestui cod, am putea fi confruntați cu situația în care firul nou este mutat în fundal fără să fie executat. Firul de execuție are o referință la `v` la interior, dar firul principal renunță la `v` folosind funcția `drop` descrisă în Capitolul 15. Când apoi firul nou începe execuția, `v` nu mai este disponibil, făcând referința la acesta invalidă. O, nu!

Pentru rezolvarea erorii de compilare din Listarea 16-3, putem urma sugestia furnizată de mesajul de eroare:

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Prin adăugarea cuvântului cheie `move` în fața închiderii, obligăm închiderea să preia posesiunea asupra valorilor pe care le utilizează, în loc de a-l lăsa pe Rust să facă inferențe despre împrumutarea acestora. Modificările efectuate în Listarea 16-3, reprezentate în Listarea 16-5, vor permite compilarea și executarea codului așa cum intenționam:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

<span class="caption">Listarea 16-5: Folosirea cuvântului cheie `move` pentru a obliga o închidere să preia posesiunea valorilor pe care le utilizează</span>

Am putea fi ispitiți să încercăm același lucru pentru a remedia codul din Listarea 16-4 în care firul principal a invocat `drop`, folosind o închidere `move`. Totuși, această reparație nu va funcționa pentru că ceea ce încearcă să realizeze Listarea 16-4 este interzis dintr-un alt motiv. Adăugând `move` la închiderea respectivă, am transfera `v` în contextul închiderii și nu am mai putea apela funcția `drop` pe acesta în firul principal. Ne-am confrunta în loc cu următoarea eroare de la compilator:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Regulile de posesiune din Rust ne-au salvat încă o dată! Am primit o eroare pentru codul din Listarea 16-3 pentru că Rust a fost precaut, alegând doar să împrumute variabila `v` firului secundar de execuție, ce ar fi putut cauza ca firul principal să invalideze referința firului pornit. Instruind Rust să mute posesiunea lui `v` către firul pornit, îi garantăm că firul principal nu o va mai folosi. Schimbând Listarea 16-4 în același mod, încălcăm regulile de posesiune când încercăm să accesăm `v` în firul principal. Cuvântul cheie `move` anulează comportamentul implicit precaut al Rust de a împrumuta; nu ne permite să încălcăm regulile de posesiune.

Înarmat cu noțiuni fundamentale despre firele de execuție și API-ul lor, să descoperim ce putem *realiza* cu ajutorul acestor fire.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
