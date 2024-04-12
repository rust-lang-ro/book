## Organizarea testării

Așa cum am menționat la începutul acestui capitol, testarea reprezintă o disciplină sofisticată, iar diferite persoane aplică o varietate de terminologii și abordări organizatorice. În comunitatea Rust, testele sunt privite prin prisma a două categorii principale: *testele unitare* și *testele de integrare*. Testele unitare sunt precise și se concentrează pe testarea unui modul în izolare într-un moment dat, cu posibilitatea de a verifica interfețele private. Testele de integrare, în contrast, sunt complet externe în raport cu biblioteca ta și utilizează codul exact cum ar face-o orice alt consumator extern, limitându-se la interfața publică și explorând, adeseori, multiple module într-un singur test.

Elaborarea ambelor categorii de teste este crucială pentru a confirma că componentele individuale ale bibliotecii tale funcționează conform așteptărilor, atât izolat, cât și în combinație.

### Testele de unitate

Testele de unitate au ca scop verificarea fiecărei unități de cod în izolare de restul programului, pentru a determina rapid ce porțiuni funcționează sau nu conform așteptărilor. Aceste teste sunt amplasate în directoriul *src*, în același fișier cu codul pe care îl testează. Este o convenție uzuală să se creeze un modul cu numele `tests` în fiecare fișier, ce conține funcțiile de test, și să se adnoteze acest modul cu `cfg(test)`.

#### Modulul de teste și adnotația `#[cfg(test)]`

Adnotarea `#[cfg(test)]` aplicată pe modulul de teste instruiește Rust să compileze și să ruleze codul de test atunci când folosim `cargo test`, și nu în timpul executării `cargo build`. Această abordare economisește timp la compilare când intenționăm să construim doar biblioteca și reduce dimensiunea artifactului compilat rezultat, prin omisiunea testelor. Observăm că, deoarece testele de integrare sunt stocate într-un directoriu distinct, acestea nu au nevoie de adnotația `#[cfg(test)]`. Contrar, testele unitare, fiind localizate în aceleași fișiere ca și codul, necesită utilizarea `#[cfg(test)]` pentru a indica faptul că nu trebuie incluse în versiunea compilată finală.

Reamintim că atunci când am generat proiectul nou `adder` în prima parte a acestui capitol, Cargo a creat automat următorul cod pentru noi:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Codul afișat constituie modulul de teste produs automat. Atributul `cfg` reprezintă o scurtătură la *configuration* și indică lui Rust că elementul care urmează să fie definit ar trebui inclus doar într-o configurație specifică. În acest caz, configurația este `test`, pusă la dispoziție de Rust pentru a compila și executa teste. Aplicând atributul `cfg`, Cargo va compila codul de test doar dacă demarăm explicit testele folosind comanda `cargo test`. Acest lucru include și orice funcții auxiliare care ar putea exista în acest modul, alături de funcțiile adnotate cu `#[test]`.

#### Testarea funcțiilor private

În comunitatea de testare există un debate privind oportunitatea testării directe a funcțiilor private. Alte limbaje de programare fac dificilă sau chiar imposibilă testarea funcțiilor private. Independent de ideologia de testare pe care o urmezi, regulile de confidențialitate din Rust îți permit să testezi funcțiile private. Să analizăm codul din Listarea 11-12, care include funcția privată `internal_adder`.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

<span class="caption">Listarea 11-12: Testarea unei funcții private</span>

Observă că funcția `internal_adder` nu este etichetată ca `pub`. Testele sunt pur și simplu cod Rust, iar modulul `tests` este doar un alt modul. Așa cum am discutat în secțiunea [„Utilizarea căilor pentru a face referire la un element în structura de module”][paths]<!-- ignore -->, elementele din modulele copil au acces la elementele din modulele strămoșilor lor. În acest test, includem toate elementele modulului părinte al `test` în domeniul de aplicare cu `use super::*`, permițând testului să apeleze `internal_adder`. Dacă ești de părere că funcțiile private nu ar trebui testate, Rust nu te va forța să faci acest lucru.

### Testele de integrare

În Rust, testele de integrare sunt complet externe în raport cu biblioteca ta. Acestea utilizează biblioteca în exact aceeași manieră cum ar face orice alt segment de cod, ceea ce înseamnă că ele pot apela doar funcții care fac parte din interfața API publică a bibliotecii tale. Scopul lor este să verifice dacă diverse componente ale bibliotecii tale lucrează corect împreună. Unități de cod care funcționează corect izolat pot prezenta probleme atunci când sunt combinate, deci este important să se asigure o acoperire prin teste și pentru codul integrat. Pentru realizarea testelor de integrare, este necesar să creezi mai întâi un directoriu numit *tests*.

#### Directoriul *tests*

În structura de top a directoriului nostru de proiect, alături de *src*, creăm un directoriu *tests*. Cargo cunoaște locația acestui directoriu și va căuta aici fișierele cu testele de integrare. Aici putem adăuga oricâte fișiere de teste dorim, și Cargo le va compila individual ca pe niște crate-uri separate.

Pentru a crea un test de integrare, având codul din Listarea 11-12 în fișierul *src/lib.rs*, deschidem directoriul *tests* și creăm un fișier cu numele *tests/integration_test.rs*. Iată cum ar arăta structura directoriului:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Scriem codul din Listarea 11-13 în fișierul *tests/integration_test.rs*:

<span class="filename">Numele fișierului: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

<span class="caption">Listarea 11-13: Test de integrare pentru o funcție din crate-ul `adder`</span>

Dat fiind că fiecare fișier din `tests` este tratat ca un crate separat, este necesar să includem în domeniul de vizibilitate al fiecărui test crate biblioteca pe care o testăm. Astfel, adăugăm `use adder` la începutul codului; aspect pe care nu l-am avut în testele unitare.

Nu este necesar să adnotăm codul din *tests/integration_test.rs* cu `#[cfg(test)]`. Cargo consideră `tests` un directoriu special și compilează fișierele conținute doar când executăm `cargo test`. Să executăm `cargo test` acum:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Afișajul conține trei secțiuni: testele unitare, testul de integrare și testele de documentație. Este important de reținut că dacă un test dintr-o anumită secțiune eșuează, secțiunile următoare nu vor fi executate. De exemplu, dacă un test unitar nu trece, nu vom vedea niciun output pentru testele de integrare și pentru cele de documentație, deoarece vor fi executate doar dacă testele unitare au trecut cu succes.

Prima secțiune, cea a testelor unitare, ne prezintă ceea ce am văzut și înainte: o linie pentru fiecare test unitar (inclusiv `internal` adăugat în Listarea 11-12) și o linie de sumar pentru toate testele unitare.

Secțiunea pentru testele de integrare debutează cu `Running tests/integration_test.rs`, urmată de o linie pentru fiecare funcție de test din testul de integrare, încheind cu o linie de sumar a rezultatelor testului de integrare înainte de începerea secțiunii `Doc-tests adder`.

Dacă adăugăm mai multe fișiere de test în directoriul *tests*, fiecare dintre acestea va avea propria sa secțiune de test de integrare.

Putem rula o anume funcție de test de integrare direct specificând numele funcției ca argument la `cargo test`. Pentru a rula toate testele dintr-un fișier specific de test de integrare, folosim argumentul `--test` cu numele fișierului după `cargo test`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

Această comandă execută exclusiv testele din *tests/integration_test.rs*.

#### Submodule în teste de integrare

Când adaugi tot mai multe teste de integrare, e posibil să dorești noi fișiere în directoriul *tests* pentru a le organiza mai bine; de exemplu, pOți grupa funcțiile de test după funcționalitățile pe care le verifică. După cum am menționat anterior, fiecare fișier din directoriul *tests* este compilat ca un crate separat, aspect util pentru crearea de domenii de vizibilitate separate care să imite cât mai fidel modul în care utilizatorii finali vor folosi crate-ul tău. Însă, acest lucru înseamnă că fișierele din directoriul *tests* nu se comportă la fel ca cele din *src*, așa cum am învățat în Capitolul 7 cu privire la separarea codului în module și fișiere.

Diferența de comportament a fișierelor din directoriul *tests* devine evidentă atunci când avem un set de funcții ajutătoare pe care dorim să le utilizăm în diverse fișiere de teste de integrare și încercăm să urmăm pașii din secțiunea [„Separarea modulelor în fișiere diferite”][separating-modules-into-files]<!-- ignore --> din Capitolul 7 pentru a le extrage într-un modul comun. De exemplu, dacă creăm *tests/common.rs* și adăugăm în el o funcție denumită `setup`, putem include în `setup` codul pe care dorim să-l apelăm din diverse funcții de testare în multiple fișiere de test:

<span class="filename">Numele fișierului: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

La rularea testelor din nou, vom observa în afișajul testelor o nouă secțiune pentru fișierul *common.rs*, cu toate că acest fișier nu conține nicio funcție de test și nici n-am invocat funcția `setup` de undeva:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

Prezența `common` în rezultatele de test cu mențiunea `running 0 tests` nu este ceea ce intenționam. Intenția noastră era pur și simplu să partajăm codul cu alte fișiere de test de integrare.

Pentru a preveni apariția secțiunii `common` în rezultatele de test, în loc de *tests/common.rs*, vom crea *tests/common/mod.rs*. Structura directorului proiectului va arăta acum astfel:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Aceasta reprezintă convenția de denumire mai veche, pe care Rust o recunoaște și care a fost menționată în secțiunea [„Căi alternative pentru Fișiere”][alt-paths]<!-- ignore --> din Capitolul 7. A alege acest nume de fișier îi indică lui Rust să nu considere modulul `common` ca fiind un fișier de teste de integrare. Când mutăm codul funcției `setup` în *tests/common/mod.rs* și ștergem fișierul *tests/common.rs*, secțiunea respectivă nu va mai apărea în afișajul testelor. Fișierele din subdirectoarele directoriului *tests* nu sunt compilate în crate-uri separate și nu au secțiuni în afișajul testelor.

După ce am creat fișierul *tests/common/mod.rs*, putem să-l utilizăm ca modul din orice fișier de teste de integrare. Iată cum apelăm funcția `setup` din testul `it_adds_two` din *tests/integration_test.rs*:

<span class="filename">Numele fișierului: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

Remarcăm că declarația `mod common;` este identică cu declarația de modul pe care am prezentat-o în Listarea 7-21. Acum în cadrul funcției de test, putem apela funcția `common::setup()`.

#### Teste de integrare pentru crate-uri binare

Dacă proiectul nostru este un crate binar conținând numai un fișier *src/main.rs* și fără un fișier *src/lib.rs*, nu putem crea teste de integrare în directoriul *tests* și nici să aducem funcțiile definite în *src/main.rs* în domeniu de vizibilitate folosind o instrucțiune `use`. Doar crate-urile de tip bibliotecă expun funcții care pot fi folosite de alte crate-uri; crate-urile binare sunt create pentru a fi executate independent.

Acesta este unul din raționamentele pentru care proiectele Rust ce oferă un binar au un fișier *src/main.rs* concis, ce apelează logica implementată în fișierul *src/lib.rs*. Având această structură, teste de integrare *pot* să testeze crate-ul bibliotecă utilizând `use` pentru a accesa funcționalitățile esențiale. Dacă aceste funcționalități esențiale funcționează corect, atunci și cantitatea mică de cod din *src/main.rs* va opera corect, iar acest segment redus de cod nu necesită testare.

## Sumar

Facilitățile de testare oferite de Rust ne permit să specificăm modul în care codul ar trebui să opereze pentru a garanta că acesta continuă să funcționeze conform așteptărilor, chiar și atunci când sunt aplicate modificări. Testele unitare verifică separat părți diferite ale unei biblioteci și pot testa detalii ale implementării private. Testele de integrare asigură că diferite componente ale bibliotecii lucrează corect împreună, utilizând API-ul public al acesteia pentru a verifica codul în același mod în care va fi folosit și de codul extern. Deși sistemul de tipizare și regulile de posesiune din Rust previn anumite categorii de defecțiuni, testele rămân esențiale pentru a diminua erorile de logică legate de comportamentul așteptat al codului.

În continuare să folosim cunoștințele acumulate în acest capitol și în cele anterioare pentru a dezvolta un proiect!

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]:
ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
