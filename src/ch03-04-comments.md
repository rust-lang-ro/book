## Comentarii

Toți programatorii se străduiesc să facă codul lor ușor de înțeles, dar uneori este necesară o explicație suplimentară. În aceste cazuri, programatorii lasă *comentarii* în codul lor sursă pe care compilatorul îl va ignora, dar persoanele care citesc codul sursă le-ar considera utile.

Acesta este un comentariu simplu:

```rust
// salutare, lume
```

În Rust, stilul idiomatic de a scrie comentarii începe cu două liniuțe, și comentariul continuă până la sfârșitul liniei. Pentru comentarii care se extind mai mult decât o linie, va trebui să includeți `//` pe fiecare linie, așa:

```rust
// Deci facem ceva complex aici, suficient de lung încât avem nevoie
// de mai multe linii de comentarii pentru a face asta! Uff! 
// Sperăm că acest comentariu va clarifica ce se întâmplă.
```

Comentariile pot fi, de asemenea, plasate la sfârșitul liniilor care conțin cod:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Dar le vei vedea mai des utilizate în acest format, cu comentariul pe o linie separată, deasupra codului pe care îl comentează:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust dispune și de un alt tip de comentariu, comentariile de documentare, despre care vom discuta în secțiunea [„Publicarea unui crate la crates.io”][publishing] <!-- ignore --> a Capitolului 14.

[publishing]: ch14-02-publishing-to-crates-io.html
