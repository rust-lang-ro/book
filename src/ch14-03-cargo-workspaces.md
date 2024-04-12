## Spațiile de lucru Cargo

În Capitolul 12, am creat un pachet care conținea atât un crate binar, cât și unul de bibliotecă. Conform evoluției proiectului tău, este posibil să observi că mărimea crate-ului de bibliotecă se mărește și ai dori să împarți pachetul în mai multe crate-uri de bibliotecă. Cargo pune la dispoziție o caracteristică denumită *workspaces* (spații de lucru), care facilitează gestionarea unui set de pachete conexe ce sunt dezvoltate simultan.

### Crearea unui spațiu de lucru

Un *workspace* este un set de pachete care partajează același *Cargo.lock* și directoriu de păstrare a rezultatelor compilării. Să creăm un proiect folosind un workspace, unde vom utiliza cod simplu pentru a ne focaliza pe structura spațiului de lucru. Există diverse moduri de structurare a unui spațiu de lucru, însă vom prezenta doar o metodă frecvent utilizată. Vom avea un spațiu de lucru ce va conține un binar și două biblioteci. Binarul, care va fi responsabil de funcționalitatea principală, va depinde de cele două biblioteci. Prima bibliotecă va oferi funcția `add_one`, iar cea de-a doua funcția `add_two`. Aceste trei crate-uri vor forma unul și același workspace. Începem prin crearea unui nou directoriu pentru workspace:

```console
$ mkdir add
$ cd add
```

Ulterior, în directoriul *add*, vom crea fișierul *Cargo.toml*, care va stabili configurația întregului spațiu de lucru. Acest fișier nu va avea secțiunea `[package]`. În loc de aceasta, va începe cu secțiunea `[workspace]`, permițându-ne să adăugăm module la spațiul nostru de lucru prin specificarea căii pachetului ce conține binarul nostru crate. În cazul de față, acea cale este *adder*:

<span class="filename">Numele fișierului: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace-with-adder-crate/add/Cargo.toml}}
```

În pasul următor, vom crea crate-ul binar `adder`, executând `cargo new` în directoriul *add*:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
```

Acum putem construi workspace-ul executând `cargo build`. Fișierele din directoriul *add* ar trebui să aibă următoarea structură:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Zona de lucru dispune de un singur directoriu *target* la nivelul superior, locul unde se vor așeza artifactele compilate; pachetul `adder` nu deține un directoriu *target* separat. Chiar dacă am executa `cargo build` din interiorul directoriului *adder*, artifactele compilate tot în *add/target* ar ajunge, nu în *add/adder/target*. Cargo organizează directoriul *target* într-un spațiu de lucru în acest mod deoarece crate-urile din cadrul unui workspace sunt concepute să colaboreze între ele. Dacă fiecare crate ar avea propriul directoriu *target*, atunci fiecare din ele ar fi nevoit să recompileze toate celelalte crate pentru a plasa artifactele în directoriul sau *target*. Astfel, partajând un directoriu *target* comun, crate-urile evită recompilări inutile.

### Crearea celui de-al doilea crate în spațiul de lucru

În continuare, să creăm un alt crate membru în zona de lucru și să-l numim `add_one`. Modificăm *Cargo.toml* de nivel superior pentru a specifica calea *add_one* în lista `members`:

<span class="filename">Numele fișierului: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

Apoi generăm un nou crate de tip bibliotecă denumit `add_one`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
```

Directorul nostru *add* ar trebui acum să conțină aceste directoare și fișiere:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

În fișierul *add_one/src/lib.rs*, să adăugăm o funcție `add_one`:

<span class="filename">Numele fișierului: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Acum pachetul `adder`, care conține binarul nostru, poate depinde de crate-ul `add_one` cu biblioteca noastră. Pentru început, trebuie să adăugăm o dependență de cale pentru `add_one` în *adder/Cargo.toml*.

<span class="filename">Numele fișierului: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo nu presupune implicit că crate-urile dintr-un spațiu de lucru vor depinde unele de altele, deci trebuie să specificăm în mod explicit relațiile de dependență.

Acum, să utilizăm funcția `add_one` (din crate-ul `add_one`) în crate-ul `adder`. Deschideți fișierul *adder/src/main.rs* și inserați o linie `use` la început pentru a aduce crate-ul bibliotecă `add_one` în domeniul de vizibilitate. Modificați apoi funcția `main` pentru a chema funcția `add_one`, așa cum e ilustrat în Listarea 14-7.

<span class="filename">Numele fișierului: adder/src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

<span class="caption">Listarea 14-7: Utilizarea crate-ului bibliotecă `add_one` din crate-ul `adder`</span>

Construim spațiul de lucru executând `cargo build` în directorul superior *add*!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68s
```

Pentru a executa crate-ul binar din directorul *add*, putem preciza pachetul din zona de lucru pe care dorim să îl rulăm folosind argumentul `-p` și numele pachetului împreună cu `cargo run`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Această comandă rulează codul din *adder/src/main.rs*, care depinde de crate-ul `add_one`.

#### Utilizarea unei dependente externe într-un workspace

Observăm că există un singur fișier *Cargo.lock* situat la nivelul de sus al spaíului nostru de lucru, și nu un *Cargo.lock* în directoriul fiecărui crate individual. Acest aranjament garantează că toate crate-urile folosesc aceleași versiuni ale dependențelor. Dacă includem pachetul `rand` în fișierele *adder/Cargo.toml* și *add_one/Cargo.toml*, Cargo va coordona cele două referințe pentru a utiliza o singură versiune a lui `rand`, pe care o va înregistra în fișierul *Cargo.lock* comun. Utilizarea aceleiași versiuni a dependențelor de către toate crate-urile din zona de lucru asigură că acestea vor fi întotdeauna interoperabile. Să adăugăm crate-ul `rand` în secțiunea `[dependencies]` a fișierului *add_one/Cargo.toml*, pentru a-l putea utiliza în crate-ul `add_one`:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Filename: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Acum, putem scrie `use rand;` în fișierul *add_one/src/lib.rs*, și dacă executăm `cargo build` din directoriul *add*, acesta va descărca și compila crate-ul `rand`. O să primim un avertisment deoarece nu folosim `rand` după ce l-am introdus în domeniul de vizibilitate:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 10.18s
```

Fișierul *Cargo.lock* de la nivelul cel mai de sus conține acum informații despre faptul că `add_one` depinde de `rand`. Totuși, chiar dacă `rand` este folosit în unele părți ale workspace-ului, nu va putea fi utilizat în alte crate-uri decât dacă adăugăm `rand` și în fișierele lor *Cargo.toml*. De exemplu, introducerea `use rand;` în fișierul *adder/src/main.rs* al pachetului `adder` va conduce la o eroare:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Pentru a remedierea acestei probleme, actualizează fișierul *Cargo.toml* aferent pachetului `adder` indicând faptul că `rand` constituie de asemenea o dependență pentru el. Compilarea pachetului `adder` va adăuga `rand` în lista de dependențe pentru `adder` în fișierul *Cargo.lock*, dar nu vor fi descărcate versiuni suplimentare ale `rand`. Cargo a garantat că toate crate-urile din toate pachetele spaíului de lucru care folosesc pachetul `rand` se vor baza pe aceeași versiune, ajutându-ne să economisim spațiu și asigurând compatibilitatea între crate-urile din zona noastră de lucru.

#### Adăugarea unui test într-un workspace

Pentru o nouă îmbunătățire, să adăugăm un test pentru funcția `add_one::add_one` din crate-ul `add_one`:

<span class="filename">Numele fișierului: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Acum rulează `cargo test` în directoriul top-level *add*. Executând `cargo test` într-un spațiu de lucru configurat în acest mod, vor fi rulate testele pentru toate crate-urile din workspace:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.27s
     Running unittests src/lib.rs (target/debug/deps/add_one-f0253159197f7841)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-49979ff40686fa8e)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Prima parte a ieșierei indică faptul că testul `it_works` din crate-ul `add_one` a fost validat cu succes. Următoarea secțiune arată că nu au fost descoperite teste în crate-ul `adder`, iar ultima parte a output-ului revelează că nu au fost găsite teste de documentație pentru crate-ul `add_one`.

Putem, de asemenea, să rulăm teste doar pentru un singur crate din spațiul de lucur, făcând asta din directoriul top-level, folosind opțiunea `-p` și specificând numele crate-ului pe care îl testăm:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-b3235fea9a156f74)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Această parte a afișajului confirmă că `cargo test` a executat numai testele pentru crate-ul `add_one`, fără a include testele pentru crate-ul `adder`.

Dacă ai intenția de a publica crate-urile din spațiul de lucru pe [crates.io](https://crates.io/), fiecare crate trebuie publicat separat. Similar cu `cargo test`, putem publica un anumit crate din zona de lucru folosind opțiunea `-p` și precizând numele acestuia.

Ca exercițiu adițional, încearcă să adaugi un crate `add_two` în acest spațiu de lucru, într-o manieră asemănătoare cu `add_one`!

Pe măsură ce proiectul tău evoluează, iată de ce ar fi benefică folosirea zonelor de lucru: este mai ușor de înțeles componente individuale și mai mici decât un monolit de cod. În plus, menținerea crate-urilor într-un spațiu de lucru poate facilita coordonarea între ele, în special când sunt modificate simultan.
