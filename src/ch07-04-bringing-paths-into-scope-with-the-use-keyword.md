## Utilizarea cuvântului cheie `use` pentru a aduce căile în domeniul de vizibilitate

Scrierea completă a căilor pentru apelarea funcțiilor poate fi incomod de repetitivă. În Listarea 7-7, indiferent dacă optăm pentru drumul absolut sau cel relativ către funcția `add_to_waitlist`, am trebuit de fiecare dată să specificăm și `front_of_house` și `hosting`. Din fericire, există o modalitate de a simplifica acest proces: putem crea o scurtătură către o cale cu ajutorul cuvântului cheie `use`. Astfel, putem folosi un nume mai scurt în restul domeniului de vizibilitate.

În Listarea 7-11, introducem modulul `crate::front_of_house::hosting` în domeniul de vizibilitate al funcției `eat_at_restaurant`. Astfel, pentru apelarea funcției `add_to_waitlist` în contextul `eat_at_restaurant`, trebuie doar să specificăm `hosting::add_to_waitlist`.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

<span class="caption">Listarea 7-11: Introducerea unui modul în domeniul de vizibilitate cu `use`</span>

Adăugarea `use` și a unei căi într-un domeniu este similară cu operațiunea de creare a unui link simbolic în sistemul de fișiere. Prin introducerea `use crate::front_of_house::hosting` la nivelul rădăcinii crate-ului, `hosting` devine un nume valid în cadrul acestui domeniu de vizibilitate, ca și cum modulul `hosting` ar fi fost definit chiar în rădăcina crate-ului. Căile aduse în vizibilitate cu `use` au capacitatea de a verifica confidențialitatea, la fel ca toate celelalte căi.

Trebuie să ținem minte că `use` creează o scurtătură doar în cadrul specific al domeniului de vizibilitate în care este folosit. Listarea 7-12 plasează funcția `eat_at_restaurant` într-un nou submodul numit `customer`. Acesta reprezintă un domeniu diferit de cel al declarației `use`, așa că funcția nu va putea fi compilată:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

<span class="caption">Listarea 7-12: Instrucțiunea `use` este aplicabilă doar în domeniul de vizibilitate în care se găsește</span>

Eroarea de compilare indică faptul că scurtătura nu mai este valabilă în interiorul modulului `customer`:

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

Este important de observat că există de asemenea un avertisment care ne informează că instrucțiunea `use` nu mai este folosită în domeniul său de vizibilitate! Pentru a remediat această problemă, putem muta instrucțiunea `use` direct în interiorul modulului `customer` sau putem face referire la scurtătură din modulul părinte folosind `super::hosting`, în interiorul modulului copil `customer`.

### Căi `use` idiomatice

E posibil să te fi întrebat, privind Listarea 7-11, de ce am utilizat `use crate::front_of_house::hosting` și apoi am apelat funcția `hosting::add_to_waitlist` în cadrul funcției `eat_at_restaurant`. De ce nu am specificat calea completă în `use` până la funcția `add_to_waitlist` pentru a obține același rezultat? Acest ultim mod de a proceda este ilustrat în Listarea 7-13.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

<span class="caption">Listarea 7-13: Introducerea funcției `add_to_waitlist` în domeniul de vizibilitate cu `use`, o abordare care nu respectă normele idiomatice ale limbajului</span>

Deși ambele metode, ilustrate în Listările 7-11 și 7-13, îndeplinesc aceeași sarcină, abordarea din Listarea 7-11 este considerată a fi cea corectă, conform obiceiurilor limbajului. Utilizarea lui `use` pentru a introduce modulul părinte al funcției în domeniul de vizibilitate necesită specificarea modulului părinte când apelăm funcția. Însă, astfel, devine evident că funcția nu este definită local, minimizând în același timp repetarea căii complete. În schimb, codul din Listarea 7-13 lasă în incertitudine locul unde `add_to_waitlist` este definită.

Cu toate acestea, când introducem în domeniul de vizibilitate structuri, enumerări și alte elemente cu ajutorul lui `use`, este idiomatic să se specifice calea completă. Listarea 7-14 arată metoda adecvată de a aduce structura `HashMap` din biblioteca standard în domeniul de vizibilitate al unui crate binar.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

<span class="caption">Listarea 7-14: Introducerea `HashMap` în domeniu de vizibilitate în modul consecvent</span>

Această idiomă nu are o cauză anume, este doar o tradiție care a fost adoptată pentru scrierea și citirea codului Rust.

Excepția la această convenție este dacă aducem în domeniul de vizibilitate două elemente care au același nume, folosind declarațiile `use`. Rust nu permite asta. Listarea 7-15 ilustrează cum să introducem în domeniul de vizibilitate două tipuri `Result` cu același nume, dar provenind din module părinte diferite și cum să le referim.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

<span class="caption">Listarea 7-15: Introducerea simultană a două tipuri cu același nume în același domeniu de vizibilitate impune utilizarea modulelor părinte.</span>

După cum poți observa, specificarea modulelor părinte ne ajută să distingem între cele două tipuri `Result`. Dacă am fi folosit `use std::fmt::Result` și `use std::io::Result`, am fi ajuns cu două tipuri `Result` în același domeniu și Rust nu ar fi putut distinge la care tip `Result` ne referim.

### Utilizarea cuvântului cheie `as` pentru a atribui nume noi

O altă soluție la problema aducerii a două tipuri cu același nume în același domeniu de vizibilitate cu `use` implică folosirea cuvântului cheie `as`. După ce specificăm calea, putem adăuga `as` și un nume local nou, sau un *alias*, pentru tipul respectiv. Listarea 7-16 ne prezintă o altă variantă de a scrie codul din Listarea 7-15 prin redenumirea unuia dintre cele două tipuri `Result` folosind `as`.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

<span class="caption">Listarea 7-16: Redenumirea unui tip la momentul introducerii acestuia în domeniul de vizibilitate, folosind cuvântul cheie `as`</span>

În cel de-al doilea enunț `use`, noi am decis să alegem numele `IoResult` pentru tipul `std::io::Result`. Acesta nu va intra în conflict cu tipul `Result` din `std::fmt` pe care tot l-am adus în domeniul de vizibilitate. Atât Listarea 7-15 cât și Listarea 7-16 reprezintă abordări idiomatice, deci ai libertatea de a alege cea care ți se potrivește cel mai bine!

### Re-exportarea numelor cu `pub use`

Când aducem un nume în domeniul de vizibilitate cu ajutorul cuvântului cheie `use`, numele disponibil în noul domeniu de vizibilitate este privat. Pentru a permite codului care apelează codul nostru să se refere la acel nume ca și cum ar fi fost definit în domeniul de vizibilitate al acelui cod, putem combina `pub` și `use`. Această tehnică se numește *re-exportare* deoarece aducem un element în domeniul de vizibilitate, dar de asemenea face acel element disponibil și pentru cei ce importă codul nostru.

Listarea 7-17 prezintă codul din Listarea 7-11 cu `use` în modulul root modificat în `pub use`.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

<span class="caption">Listarea 7-17: Facem un nume disponibil pentru orice cod de utilizat dintr-un nou domeniu de vizibilitate cu `pub use`</span>

Înainte de această modificare, codul extern ar trebui să apeleze funcția `add_to_waitlist` utilizând calea `restaurant::front_of_house::hosting::add_to_waitlist()`. Acum, având în vedere că `pub use` a re-exportat modulul `hosting` din modulul root, codul extern poate acum utiliza calea `restaurant::hosting::add_to_waitlist()` în schimb.

Re-exportarea este utilă când structura internă a codului tău este diferită de felul în care programatorii care apelează codul tău ar gândi despre domeniu. De exemplu, în această metaforă despre restaurant, oamenii care administrează restaurantul gândesc despre "fața casei" și "spatele casei". Dar clienții care vizitează un restaurant probabil nu vor gândi despre părțile restaurantului în acești termeni. Cu `pub use`, putem scrie codul nostru cu o structură, dar expunem o structură diferită. Făcând acest lucru, biblioteca noastră este bine organizată atât pentru programatorii care lucrează în bibliotecă cât și pentru programatorii care apelează biblioteca. Vom privi un alt exemplu de `pub use` și cum acesta afectează documentația crate-ului tău în secțiunea [„Exportarea unui API public și accesibil cu `pub use`”][ch14-pub-use] din Capitolul 14.

### Utilizarea pachetelor externe

În capitolul 2, am creat un joc de ghicit numere, care se baza pe un pachet extern numit `rand` pentru generarea numerelor aleatorii. Pentru a folosi `rand` în cadrul proiectului nostru, am inclus următoarea linie în fișierul *Cargo.toml*:

<!-- Când actualizezi versiunea `rand` folosită, nu uita să actualizezi și versiunea `rand` din următoarele fișiere, astfel încât să fie toate sincronizate:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Numele fișierului: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

Adăugarea `rand` ca o dependență în fișierul *Cargo.toml* instruiește sistemul Cargo să descarce pachetul `rand` și orice dependențe asociate de pe [crates.io](https://crates.io/) și să-l predea la dispoziția proiectului nostru.

Ulterior, pentru a aduce definițiile din `rand` în domeniul de vizibilitate al pachetului nostru, am inclus o instrucțiune `use`. Aceasta era precedată de numele pachetului, `rand`, și continuată cu lista elementelor pe care am dorit să le aducem în vizibilitate. Îți poți aduce aminte de secțiunea [„Generarea unui număr aleator”][rand]<!-- ignore --> din capitolul 2, unde am introdus trăsătura `Rng` în domeniul de vizibilitate și am apelat funcția `rand::thread_rng`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Membrii comunității Rust au pus la dispoziție multe pachete pe [crates.io](https://crates.io/), iar integrarea oricăruia dintre acestea în pachetul tău  presupune aceeași pași: listarea pachetelor în fișierul *Cargo.toml* și utilizarea instrucțiunii `use` pentru a introduce elemente din aceste pachete în domeniul de vizibilitate.

Este important de notat că și biblioteca standard `std` este de asemenea un crate extern pachetului nostru. Totuși, deoarece biblioteca standard este inclusă în setul de livrare standard al limbajului Rust, nu trebuie să modificăm *Cargo.toml* pentru a include `std`. Trebuie doar să folosim instrucțiunea `use` pentru a introduce elemente din aceasta în domeniul de vizibilitate al pachetului nostru. De exemplu, pentru a folosi `HashMap`, am utiliza următoarea linie:

```rust
use std::collections::HashMap;
```

Aceasta este o cale absolută care începe cu `std`, numele crate-ului bibliotecii standard.

### Utilizarea căilor îmbinate pentru a economisi spațiu în listele lungi de `use`

Când lucrăm cu numeroase elemente definite în același crate sau în același modul, este obositor și consumator de spațiu să le listăm pe fiecare în parte pe linii separate. Să luăm ca exemplu jocul Ghicitoarea din Listarea 2-4, unde am folosit două declarații `use` pentru a importa elemente din `std` în domeniul nostru de vizibilitate:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

În loc de aceasta, putem utiliza căi îmbinate pentru a aduce aceleași elemente în domeniul de vizibilitate într-o singură linie. Acest lucru se face prin specificarea părții comune a căii, continuată cu două puncte și cu acolade ce îngrădesc părțile diferite ale căilor. Acest concept este prezentat in Listarea 7-18.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

<span class="caption">Listarea 7-18: Utilizarea unei căi îmbinate pentru a importa mai multe
elemente cu același prefix în domeniul de vizibilitate</span>

În cadrul programelor mari, tehnica importării unui număr mare de elemente din același crate sau modul prin căi îmbinate poate reduce considerabil numărul de declarații `use` separate.

O cale îmbinată poate fi utilizată la orice nivel într-o cale, fapt util atunci când dorim să combinăm două declarații `use` care au o sub-cale comună. De exemplu, Lista 7-19 ne arată două declarații `use`: una ce aduce `std::io` în domeniul de vizibilitate și una care aduce `std::io::Write` în domeniul de vizibilitate.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

<span class="caption">Listarea 7-19: Două declarații `use` cu o sub-cale comună </span>

Pentru a combina cele două căi într-o singură declarație `use`, folosim `self` în cadrul căii îmbinate, așa cum vedem în Listarea 7-20.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

<span class="caption">Listarea 7-20: Combinarea căilor din Lista 7-19 într-o singură declarație `use`</span>

În urma acestei operațiuni, `std::io` și `std::io::Write` sunt aduse în domeniul de vizibilitate.

### Utilizarea operatorului `*` (glob) 

Daca dorim să includem *toate* elementele publice definite printr-o anumită cale în domeniul nostru de vizibilitate, atunci trebuie sa utilizăm calea dorită completată cu operatorul `*`:

```rust
use std::collections::*;
```

Această instrucțiune `use` face ca toate elementele publice definite în `std::collections` să fie disponibile în domeniul de vizibilitate actual. Trebuie să fim prudenți când utilizăm operatorul `*`! Ne poate face dificilă identificarea numelor care sunt în domeniul de vizibilitate și localizarea locului unde a fost definit un nume folosit în programul nostru.

De obicei, utilizăm operatorul `*` atunci când realizăm teste pentru a include totul în modulul `tests`. Vom aborda acest subiect în secțiunea [„Cum să scriem teste”][writing-tests]<!-- ignore --> din Capitolul 11. De asemenea, uneori operatorul `*` face parte din pattern-ul 'preludiu'. Poți accesa [documentația bibliotecii standard](../std/prelude/index.html#other-preludes)<!-- ignore --> pentru a afla mai multe detalii despre acest pattern.

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
