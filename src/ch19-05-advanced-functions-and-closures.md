## Funcții și închideri avansate

Această secțiune abordează unele funcționalități avansate legate de funcții și închideri, incluzând pointeri de funcții și returnarea închiderilor.

### Pointeri de funcții

Am discutat despre cum să pasăm închideri funcțiilor; de asemenea, poți pasa și funcții obișnuite funcțiilor! Această tehnică este utilă atunci când vrei să pasezi o funcție pe care deja ai definit-o în loc să definești o nouă închidere. Funcțiile se pot transforma în tip `fn` (cu 'f' mic), pentru a nu se confunda cu trăsătura închiderii `Fn`. Tipul `fn` este numit *pointer de funcție*. Utilizarea pointerilor de funcție pentru pasarea funcțiilor ne permite să folosim funcții ca argumente ale altor funcții.

Sintaxa pentru specificarea că un parametru este un pointer de funcție este similară cu cea a închiderilor, așa cum este ilustrat în Listarea 19-27, unde am definit funcția `add_one` care adaugă unitate la parametrul său. Funcția `do_twice` acceptă doi parametri: un pointer de funcție către orice funcție care acceptă un parametru `i32` și returnează un `i32`, și o valoare `i32`. Funcția `do_twice` invocă funcția `f` de două ori, pasându-i valoarea `arg`, apoi sumează rezultatele celor două apeluri de funcție. Funcția `main` invocă `do_twice` cu argumentele `add_one` și `5`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-27/src/main.rs}}
```

<span class="caption">Listarea 19-27: Utilizarea tipului `fn` pentru a accepta un pointer de funcție ca argument</span>

Această porțiune de cod afișează `The answer is: 12`. Am specificat că parametrul `f` în funcția `do_twice` este de tip `fn` care acceptă un parametru de tip `i32` și returnează `i32`. Apoi putem invoca `f` în corpul funcției `do_twice`. În funcția `main` putem trece numele funcției `add_one` ca prim argument pentru `do_twice`.

Spre deosebire de închideri, `fn` este un tip și nu o trăsătură, prin urmare specificăm direct `fn` ca tip de parametru, în loc să declarăm un parametru de tip generic cu una dintre trăsăturile `Fn` ca o delimitare de trăsătură.

Pointerii de funcție implementează toate cele trei trăsături ale închiderilor (`Fn`, `FnMut`, `FnOnce`), ceea ce înseamnă că poți mereu să utilizezi un pointer de funcție ca argument pentru o funcție care așteaptă o închidere. Este mai recomandat să scriem funcții folosind un tip generic și una dintre trăsăturile închiderii, astfel încât funcțiile noastre să poată accepta atât funcții cât și închideri.

Totuși, un exemplu când ai prefera să accepți doar `fn` și nu închideri este când interacționezi cu cod extern care nu dispune de închideri: de exemplu, funcțiile în limbajul C pot primi funcții ca argumente, însă C nu suportă închideri.

Pentru a ilustra o situație unde ai putea folosi fie o închidere definită direct în cod, fie o funcție cu nume, să examinăm utilizarea metodei `map` oferită de trăsătura `Iterator` din biblioteca standard. Pentru a aplica funcția `map` pentru a converti un vector de numere într-un vector de string-uri, putem folosi o închidere, astfel:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-15-map-closure/src/main.rs:here}}
```

Alternativ, am putea folosi o funcție definită cu nume drept argument pentru `map` în locul unei închideri, în felul următor:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-16-map-function/src/main.rs:here}}
```

Este important de reținut că trebuie să utilizăm sintaxa complet calificată despre care am discutat anterior în secțiunea [„Trăsături avansate”][advanced-traits]<!-- ignore -->, deoarece există multiple funcții disponibile sub numele `to_string`. Aici, utilizăm funcția `to_string` definită în trăsătura `ToString`, pe care biblioteca standard o implementează pentru orice tip ce implementează `Display`.

Reamintește-ți din secțiunea [„Valori ale enum-urilor”][enum-values]<!-- ignore --> a Capitolului 6 că denumirea fiecărei variante a enum-ului pe care o definim devine de asemenea o funcție de inițializare. Aceste funcții de inițializare pot fi folosite ca pointeri către funcții care implementează trăsăturile închiderii, ceea ce înseamnă că putem specifica funcțiile de inițializare ca argumente pentru metodele ce acceptă închideri, astfel:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-17-map-initializer/src/main.rs:here}}
```

Aici, creăm instanțe de tip `Status::Value` folosind fiecare valoare `u32` situată în diapazonul pe care `map` este invocat, utilizând funcția de inițializare pentru `Status::Value`. Unii preferă acest stil, în timp ce alții optează pentru utilizarea de închideri. Fiindcă acestea compilează la același cod, alege stilul care îți pare mai clar.

### Returnarea închiderilor

Închiderile sunt reprezentate de trăsături, ceea ce înseamnă că nu se pot returna direct închideri. De cele mai multe ori, când ai vrea să returnezi o trăsătură, poti folosi tipul concret care implementează trăsătura ca valoare de retur pentru funcție. Totuși, acest lucru nu este posibil cu închiderile, întrucât nu au un tip concret care să fie returnabil; nu este permis, de exemplu, să folosiți pointerul de funcție `fn` ca tip de retur.

Următorul cod încearcă să returneze direct o închidere, însă nu va reuși să compileze:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-18-returns-closure/src/lib.rs}}
```

Eroarea raportată de compilator este următoarea:

```console
{{#include ../listings/ch19-advanced-features/no-listing-18-returns-closure/output.txt}}
```

Eroarea invocă din nou trăsătura `Sized`! Rust nu poate determina cât spațiu va fi necesar pentru stocarea închiderii. Am întâlnit o soluție pentru această problemă anterior. Putem utiliza un obiect de tip trăsătură:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-19-returns-closure-trait-object/src/lib.rs}}
```

Acest cod va compila fără probleme. Pentru mai multe detalii despre obiectele de tip trăsătură, vedem secțiunea [“Using Trait Objects That Allow for Values of Different Types”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore --> din Capitolul 17.

Acum, să ne îndreptăm atenția către macrouri!

[advanced-traits]:
ch19-03-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[using-trait-objects-that-allow-for-values-of-different-types]:
ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
