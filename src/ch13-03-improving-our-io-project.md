## Îmbunătățim proiectul nostru I/O

Grație cunoștințelor noastre recent dobândite despre iteratori, avem posibilitatea să perfecționăm proiectul I/O din Capitolul 12 prin utilizarea iteratorilor, pentru a face codul nu doar mai concis, dar și mai ușor de înțeles. Analizăm modul în care iteratorii pot optimiza implementarea funcției `Config::build` și a funcției `search`.

### Înlăturăm un `clone` cu ajutorul unui iterator

În Listarea 12-6, am introdus cod care procesa o secțiune de `String` și care iniția o instanță a structurii `Config`, apelând la indexarea secțiunii și clonarea valorilor, pentru a-i permite structurii `Config` să aibă propriile valori. În Listarea 13-17, reprezentăm implementarea funcției `Config::build` așa cum era în Listarea 12-23:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/lib.rs:ch13}}
```

<span class="caption">Listarea 13-17: Reconstituirea funcției `Config::build` din Listarea 12-23</span>

Inițial, am menționat că nu trebuie să ne preocupăm de ineficiența apelurilor `clone`, deoarece aveam de gând să le eliminăm ulterior. Acest moment a venit.

Ne-am văzut nevoiți să utilizăm `clone` deoarece avem o secțiune cu elemente `String` în parametrul `args`, iar funcția `build` nu avea posesiunea asupra `args`. Pentru a putea returna o instanță `Config` care să-și posede propriile valori, a fost necesar să clonăm valorile din câmpurile `query` și `file_path` ale structurii `Config`.

Înarmați cu înțelegerea nouă despre iteratori, putem modifica funcția `build` astfel încât să preia un iterator, și nu o secțiune (împrumutată) drept argument, preluând posesiunea acestuia. Vom aplica funcționalităţile iteratorului în locul verificării lungimii secțiunii și a indexărilor. Acest lucru va clarifica intențiile funcției `Config::build` prin utilizarea iteratorului care va parcurge valorile în mod direct.

Odată ce `Config::build` va prelua posesiunea iteratorului și va renunța la operațiuni de indexare ce presupun împrumuturi, vom putea transfera valorile `String` direct din iterator în `Config`, eliminând astfel necesitatea de a apela `clone` și de a crea alocări suplimentare.

#### Utilizăm direct iteratorul returnat

Deschide fișierul *src/main.rs* din proiectul de I/O, care ar trebui să arate în felul următor:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

În primul rând, vom modifica începutul funcției `main` pe care l-am prezentat în Listarea 12-24, aplicând în schimb codul din Listarea 13-18, care de această dată implică utilizarea unui iterator. Codul nu va compila până nu efectuăm o actualizare și a funcției `Config::build`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

<span class="caption">Listarea 13-18: Predarea valorii de retur a `env::args` către `Config::build`</span>

Funcția `env::args` returnează un iterator! În loc de a colecta valorile iteratorului într-un array și apoi de a transmite o secțiune către `Config::build`, acum predăm direct posesiunea iteratorului rezultat din `env::args` către `Config::build`.

Următorul pas este actualizarea definiției `Config::build`. În fișierul *src/lib.rs* al proiectului de I/O, modificăm semnătura `Config::build` astfel încât să corespundă Listării 13-19. Totuși, acesta nu va compila deoarece este necesar să actualizăm și corpul funcției.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/lib.rs:here}}
```

<span class="caption">Listarea 13-19: Actualizarea semnăturii metodei `Config::build` pentru a accepta un iterator</span>

Documentația bibliotecii standard pentru funcția `env::args` indică faptul că tipul iteratorului returnat este `std::env::Args`, iar acest tip implementează trăsătura `Iterator` și returnează obiecte de tip `String`.

Semnătura metodei `Config::build` a fost actualizată astfel încât parametrul `args` să fie de un tip generic cu delimitările de trăsătură `impl Iterator<Item = String>`, în loc de `&[String]`. Această folosire a sintaxei `impl Trait` pe care am discutat-o în secțiunea [„Trăsături ca parametri”][impl-trait]<!-- ignore --> din Capitolul 10 sugerează că `args` poate fi de orice tip care implementează tipul `Iterator` și returnează elemente de tip `String`.

Întrucât preluăm posesiunea `args` și intenționăm să modificăm `args` iterând peste acesta, putem introduce cuvântul cheie `mut` în definiția parametrului `args` pentru a-l face mutabil.

#### Folosim metodele trăsăturii `Iterator` în locul indexării

Acum vom corecta corpul funcției `Config::build`. Dat fiind faptul că `args` implementează trăsătura `Iterator`, știm că putem apela metoda `next` asupra-i! Listarea 13-20 actualizează codul din Listarea 12-23 pentru a utiliza metoda `next`:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/lib.rs:here}}
```

<span class="caption">Listarea 13-20: Modificarea corpului lui `Config::build` pentru utilizarea metodelor iteratorului</span>

Să ne amintim că prima valoare în afișajul returnat de `env::args` este numele programului. Dorim să ignorăm acesta și să accesăm următoarea valoare, prin urmare inițial apelăm `next` fără să utilizăm valoarea returnată. Apoi, apelăm `next` pentru a extrage valoarea pe care intenționăm s-o alocăm câmpului `query` din `Config`. Dacă `next` întoarce un `Some`, folosim un `match` pentru a extrage valoarea dorită. Dacă întoarce `None`, acest lucru sugerează că nu s-au furnizat destule argumente și întrerupem procesarea anticipat, returnând o valoare de tip `Err`. Procedăm similar și pentru valoarea `file_path`.

### Clarificăm codul cu ajutorul adaptoarelor de iteratori

De asemenea, putem utiliza iteratorii și în funcția `search` din proiectul nostru de I/O, reprodusă aici în Listarea 13-21, exact cum a apărut în Listarea 12-19:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

<span class="caption">Listarea 13-21: Implementarea funcției `search` din Listarea 12-19</span>

Folosind metode adaptoare de iteratori, putem rescrie codul într-un mod mai succint. Aceasta ne permite, de asemenea, să evităm utilizarea unui vector intermediar mutabil `results`. Programarea funcțională preferă să minimizeze cât mai mult posibil stările mutabile, pentru a crește claritatea codului. Eliberând codul de starea mutabilă, s-ar putea deschide posibilitatea de a îmbunătăți funcția `search`, permițând căutarea să se desfășoare în paralel, deoarece nu va mai fi nevoie să gestionăm accesul concurent la vectorul `results`. În Listarea 13-22 este prezentată această modificare:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

<span class="caption">Listarea 13-22: Aplicarea metodelor adaptoare de iteratori în implementarea funcției `search`</span>

Reamintim că scopul funcției `search` este de a returna toate liniile din `contents` care conțin termenul `query`. Similar cu exemplul `filter` din Listarea 13-16, aici utilizăm adaptorul `filter` pentru a selecta doar liniile care, după apelul `line.contains(query)`, returnează `true`. Acestea sunt apoi agregate într-un alt vector prin metoda `collect`. Mult mai simplu! Aplică aceeași metodă, folosind adaptoare de iteratori și pentru funcția `search_case_insensitive`.

### Alegerem între bucle și iteratori

Întrebarea care se ridică este ce stil ar trebui să adoptăm în codul nostru și care este motivația alegerii: implementarea originală din Listarea 13-21 sau versiunea cu iteratori din Listarea 13-22. Majoritatea programatorilor Rust optează pentru utilizarea stilului cu iteratori. Deși la început poate părea mai dificil, odată ce te obișnuiești cu adaptoarele pentru iteratori și cu rolul lor, codul cu iteratori poate deveni mult mai intuitiv. În loc să ajustezi detaliile buclelor și să construiești noi array-uri, codul se focalizează asupra obiectivului principal al buclei. Acest lucru face codul mai abstract, ușurând identificarea conceptelor specifice acestui cod, precum condiția de filtrare pe care trebuie să o îndeplinească fiecare element în cadrul iteratorului.

Cu toate acestea, sunt cele două implementări de fapt echivalente? Ar putea fi tentant să presupunem că bucla mai de nivel scăzut este mai rapidă. Să examinăm aspectele legate de performanță.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
