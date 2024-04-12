## Publicarea unui crate pe crates.io

Folosim crate-uri de pe [crates.io](https://crates.io/) ca dependențe pentru proiectul nostru, dar poți de asemenea să-ți împărtășești codul cu alți programatori publicând propriile crate-uri. Registrul de crate-uri de pe [crates.io](https://crates.io/) difuzează codul sursă al pachetelor tale, găzduind în principal cod open source.

Rust și Cargo dispun de caracteristici care fac mai ușoară găsirea și utilizarea crate-ului tău publicat de către alții. Vom vorbi în continuare despre unele dintre aceste caracteristici și apoi vom explica cum se publică un crate.

### Crearea de comentarii documentație utile

Documentarea corectă a pachetelor voastre va ajuta alți utilizatori să înțeleagă cum și când să le folosească, de aceea este important să alocați timp pentru a redacta documentația. În Capitolul 3, am învățat cum se pot adăuga comentarii codului Rust folosind `//`. Rust oferă de asemenea un tip special de comentariu pentru documentație, cunoscut ca *comentariu de documentație*, ce generează documentație în format HTML. HTML-ul prezintă conținutul comentariilor de documentație pentru elementele API publice, orientate către programatori interesați să cunoască modul în care pot *utiliza* crate-ul dumneavoastră, nu neapărat cum este el *implementat*.

Comentariile de documentație folosesc trei slash-uri, `///`, în locul celor două și permit folosirea notației Markdown pentru formatarea textelor. Aceste comentarii de documentație se plasează imediat înaintea elementului documentat. Listarea 14-1 prezintă comentariile de documentație pentru funcția `add_one` dintr-un crate denumit `my_crate`.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

<span class="caption">Listarea 14-1: Un comentariu de documentație pentru o funcție</span>

Aici, descriem funcționalitatea funcției `add_one`, demarăm o secțiune cu titlul `Examples` și furnizăm cod ce exemplifică modul de utilizare al funcției `add_one`. Putem crea documentația HTML aferentă acestui comentariu de documentație executând comanda `cargo doc`. Această comandă pornește instrumentul `rustdoc` care vine odată cu instalarea Rustului și plasează documentația HTML generată în directorul *target/doc*.

Pentru a facilita accesul, executând `cargo doc --open` va construi documentația HTML pentru crate-ul curent (inclusiv documentația pentru toate dependențele crate-ului) și va deschide rezultatul într-un navigator web. Navigând către funcția `add_one`, veți observa cum este randat textul din comentariile de documentație, așa cum se observă în Figura 14-1:

<img alt="Documentație HTML pentru funcția `add_one` a `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Figura 14-1: Documentația HTML pentru funcția `add_one`</span>

#### Secțiuni comun utilizate

Am folosit antetul Markdown `# Examples` în Listarea 14-1 pentru a crea o secțiune în HTML cu titlul "Examples". Alte secțiuni comune pe care autorii de crate-uri le includ în documentațiile lor sunt:

* **Panics**: Situațiile în care funcția descrisă ar putea provoca o panică. Cei care apelează funcția trebuie să se asigure că nu o fac în aceste condiții, pentru a evita erori critice în programul lor.
* **Errors**: Dacă funcția returnează un `Result`, este util pentru apelanți să cunoască tipurile de erori posibile și condițiile care le-ar putea cauza, astfel încât să poată gestiona corespunzător fiecare tip de eroare.
* **Safety**: În cazul unei funcții marcate ca `unsafe` (ceea ce vom aborda în Capitolul 19), este important să existe o secțiune care explică motivele pentru care funcția este considerată nesigură și care sunt invarianții pe care funcția îi presupune respectați de către apelanți.

Nu este necesar ca toate secțiunile de documentație să fie prezente, dar această listă poate servi ca o reamintire utilă despre elementele codului care ar putea fi de interes utilizatorilor.

#### Comentarii de documentație ca teste

Includerea exemplelor de cod în comentariile de documentație poate ilustra cum să fie folosită biblioteca ta, iar un avantaj suplimentar este că, atunci când rulezi `cargo test`, exemplele de cod din comentariile tale vor fi executate ca teste! Nu există ceva mai util decât documentație însoțită de exemple. Totodată, nu există ceva mai problematic decât exemple care nu funcționează pentru că s-a modificat codul de la scrierea documentației. Dacă executăm `cargo test` cu documentația pentru funcția `add_one` prezentată în Listarea 14-1, vom observa în rezultatele testelor o secțiune similară cu aceasta:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Dacă acum schimbăm fie funcția, fie exemplul, astfel încât `assert_eq!` din exemplu să producă panică și executăm `cargo test` din nou, vom constata că testele din documentație detectează neconcordanța dintre exemplu și cod!

#### Comentarea elementelor încapsulate

Stilul de comentariu doc `//!` adaugă documentație la elementul care include comentariile în loc să le adauge la cele care urmează după comentarii. Folosim aceste comentarii doc în mod obișnuit în interiorul fișierului rădăcină al crate-ului (*src/lib.rs* conform convenției) sau în cadrul unui modul pentru a documenta întregul crate sau modul.

De exemplu, pentru a adăuga documentație care descrie scopul crate-ului `my_crate`, care conține funcția `add_one`, inserăm comentarii de documentație începând cu `//!` la începutul fișierului *src/lib.rs*, așa cum este arătat în Listarea 14-2:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

<span class="caption">Listarea 14-2: Documentația pentru crate-ul `my_crate` ca întreg</span>

Remarcăm faptul că nu se găsește niciun cod după ultima linie care începe cu `//!`. Alegând să începem comentariile cu `//!` și nu cu `///`, ne-am documentat elementul care conține acest comentariu și nu un element care ar urma după comentariu. În acest caz, respectivul element este fișierul *src/lib.rs*, rădăcina crate-ului. Aceste comentarii descriu crate-ul în totalitatea sa.

Când rulăm `cargo doc --open`, aceste comentarii vor fi vizibile pe pagina principală a documentației `my_crate`, plasate deasupra listei de elemente publice ale crate-ului, așa cum putem observa în Figura 14-2:

<img alt="Documentație HTML generată cu un comentariu la nivelul întregului crate" src="img/trpl14-02.png" class="center" />

<span class="caption">Figura 14-2: Documentația generată pentru `my_crate`, incluzând comentariul ce descrie crate-ul în întregime</span>

Comentariile de documentație din interiorul elementelor sunt deosebit de utile pentru descrierea crate-urilor și modulelor. Le utilizăm pentru a clarifica scopul general al containerului, facilitând astfel înțelegerea organizării crate-ului de către utilizatori.

### Exportarea unui API public și accesibil cu `pub use`

Structura API-ului public este esențială atunci când vine vorba de publicarea unui crate. Utilizatorii acestuia pot avea dificultăți în a localiza componentele dorite, mai ales dacă structura de module este labirintică.

În Capitolul 7, am explicat cum să facem elemente publice folosind cuvântul cheie `pub` și cum să le importăm într-un domeniu de vizibilitate cu `use`. Dar structura internă pe care ai dezvoltat-o ar putea să nu fie prea accesibilă pentru utilizatori. De exemplu, dacă ai structuri aranjate după o ierarhie complexă, utilizatorii s-ar putea să nu descopere ușor anumite tipuri de date sau să fie deranjați de necesitatea de a folosi o cale lungă de module: `use my_crate::some_module::another_module::UsefulType;` în loc de `use my_crate::UsefulType;`.

Norocul este că, dacă structura internă *nu* este convenabilă utilizatorilor din alte librării, nu e nevoie să o reorganizezi. În schimb, poți re-exporta elemente pentru a crea o structură publică diferită folosind `pub use`. Această tehnică permite ca elementele publice dintr-un loc să fie disponibile într-un altul, ca și cum ar fi fost definite acolo.

Să ne imaginăm, de exemplu, că am dezvoltat o bibliotecă numită `art` pentru reprezentarea conceptelor artistice. În interiorul acesteia avem două module: `kinds`, care conține două enumerări, `PrimaryColor` și `SecondaryColor`, și `utils` cu funcția `mix`, așa cum este prezentat în Listarea 14-3:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

<span class="caption">Listarea 14-3: Biblioteca `art` cu elementele organizate în modulele `kinds` și `utils`</span>

Figura 14-3 ilustrează cum arată pagina principală a documentației pentru crate-ul `art`, generată de `cargo doc`:

<img alt="Documentația generată pentru crate-ul `art` care prezintă modulele `kinds` și `utils`" src="img/trpl14-03.png" class="center" />

<span class="caption">Figura 14-3: Pagina principală a documentației pentru `art`, care prezintă modulele `kinds` și `utils`</span>

Remarcăm că tipurile `PrimaryColor` și `SecondaryColor` nu sunt prezentate pe pagina principală, nici funcția `mix`. Trebuie să selectăm modulele `kinds` și `utils` pentru a le vedea.

Pentru a utiliza această bibliotecă, un alt crate ar necesita declarații `use` care să includă elementele din `art` în domeniul de vizibilitate, specificând structura modulară curent definită. Listarea 14-4 demonstrează un exemplu de crate care integrează elementele `PrimaryColor` și `mix` din crate-ul `art`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

<span class="caption">Listarea 14-4: O crate care integrează elementele din crate-ul `art` cu structura sa internă expusă</span>

Autorul codului prezentat în Listarea 14-4, care utilizează crate-ul `art`, a fost nevoit să descifreze că `PrimaryColor` este parte din modulul `kinds` și `mix` din modulul `utils`. Structura modulară a crate-ului `art` este mai semnificativă pentru cei care dezvoltă `art` decât pentru utilizatorii săi. Structura internă nu furnizează informații utile pentru cineva care dorește să învețe cum să folosească `art`, ci mai degrabă creează confuzie, fiind necesar ca dezvoltatorii să deducă locațiile specifice și să specifice modulele în declarațiile `use`.

Pentru a îndepărta organizarea internă din API-ul public, putem ajusta codul crate-ului `art` din Listarea 14-3 prin adăugarea de declarații `pub use` care re-exportă elementele la nivelul cel mai de sus, așa cum este prezentat în Listarea 14-5:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

<span class="caption">Listarea 14-5: Adăugarea de declarații `pub use` pentru re-exportarea elementelor</span>

Documentația API creată de `cargo doc` pentru acest crate va prezenta acum re-exportările pe pagina de start, după cum se poate vedea în Figura 14-4, facilitând găsirea tipurilor `PrimaryColor` și `SecondaryColor` și a funcției `mix`.

<img alt="Documentația creată pentru crate-ul `art` cu re-exportările pe pagina principală" src="img/trpl14-04.png" class="center" />

<span class="caption">Figura 14-4: Pagina de start a documentației pentru `art` care afișează re-exportările</span>

Utilizatorii crate-ului `art` încă pot vedea și utiliza structura internă din Listarea 14-3 așa cum este ilustrat în Listarea 14-4, sau pot opta pentru structura mai accesibilă din Listarea 14-5, așa cum este exemplificat în Listarea 14-6:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

<span class="caption">Listarea 14-6: Un program care utilizează elementele re-exportate din crate-ul `art`</span>

Atunci când avem de-a face cu multe module încapsulate, re-exportarea tipurilor la nivelul cel mai de sus folosind `pub use` poate îmbunătăți semnificativ experiența utilizatorilor crate-ului. O altă practică frecventă a `pub use` este re-exportarea definițiilor unei dependențe în crate-ul actual, pentru a integra definițiile acelui crate în API-ul public al propriului crate.

Elaborarea unei structuri a API-ului public eficiente este mai degrabă o artă decât o știință, și multe ori e nevoie de a itera asupra ei pentru a dezvolta API-ul care răspunde cel mai bine nevoilor utilizatorilor tăi. Folosirea `pub use` permite flexibilitate în structurarea internă a crate-ului tău și separă acea structură internă de ceea ce oferi utilizatorilor tăi. Examinează codul din unele crate-uri pe care le-ai instalat pentru a observa dacă structura lor internă se diferențiază de API-ul public.

### Configurarea unui cont pe crates.io 

Pentru a publica crate-uri, trebuie mai întâi să îți creezi un cont pe [crates.io](https://crates.io/) și să obții o cheie (token) de API. Pentru aceasta, accesează pagina principală [crates.io](https://crates.io/) și conectează-te folosind un cont de GitHub. (Deocamdată este obligatoriu să folosești un cont de GitHub, dar pe viitor s-ar putea adăuga și alte metode de înregistrare.) După autentificare, mergi la setările contului tău accesând [https://crates.io/me/](https://crates.io/me/) și copie-ți cheia API. În continuare, execută comanda `cargo login` cu cheia ta API, după cum urmează:

```console
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

Această comandă îi va comunica lui Cargo cheia ta API și o va salva local în fișierul *~/.cargo/credentials*. Fii atent, această cheie este un *secret* și nu trebuie să o împărtășești cu nimeni. Dacă, dintr-un anumit motiv, o vei împărtăși, revoc-o și generează una nouă pe [crates.io](https://crates.io/).

### Adăugarea metadatelor la un crate nou

Să zicem că ai în plan să publici un crate. Înainte de a trece la publicare, e necesar să adaugi anumite metadate în secțiunea `[package]` a fișierului *Cargo.toml* corespunzător crate-ului.

Crate-ul tău trebuie să aibă un nume unic. Atunci când lucrezi la un crate pe plan local, poți opta pentru orice nume dorești. Însă, numele pentru crate-uri pe [crates.io](https://crates.io/) se atribuie pe sistemul 'primul venit, primul servit'. Odată ce un nume este ocupat, nu se mai permite publicarea unui alt crate cu același nume. Verifică dacă numele pe care îl dorești este disponibil înainte de a încerca să publici crate-ul. Dacă descoperi că numele a fost deja folosit, va trebui să alegi un altul și să actualizezi câmpul `name` din *Cargo.toml*, în secțiunea `[package]`, pentru a putea folosi noul nume la publicare, astfel:

<span class="filename">Numele fișierului: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Dacă ai ales un nume care este unic, când rulezi comanda `cargo publish` pentru publicarea crate-ului în acest punct, vei întâmpina prima dată un avertisment, urmat de o eroare:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error: missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for how to upload metadata
```

Apare această eroare pentru că lipsesc informații cruciale: descrierea și licența sunt obligatorii, pentru ca alții să poată înțelege ce face crate-ul tău și sub ce termeni pot să-l utilizeze. În fișierul *Cargo.toml*, adaugă o descriere scurtă, de una-două propoziții, aceasta urmând să apară împreună cu crate-ul tău în rezultatele de căutare. Câmpul `license` necesită introducerea unei *valori de identificare a licenței*. Identificatorii pe care îi poți folosi pentru acest scop sunt listati de [Software Package Data Exchange (SPDX) a Fundației Linux][spdx]. De exemplu, dacă ai licențiat crate-ul cu Licența MIT, adaugi identificatorul `MIT`:

<span class="filename">Numele fișierului: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Dacă preferi o licență care nu se află în listele SPDX, atunci textul acelei licențe trebuie inclus într-un fișier în proiectul tău și specificat folosind `license-file` pentru a indica numele fișierului, în loc să folosești cheia `license`.

Alegerea licenței potrivite pentru proiectul tău este un subiect care nu este acoperit de această carte. Este comun în comunitatea Rust ca proiecte să fie licențiate similar cu Rust, prin utilizarea unei licențe duble `MIT OR Apache-2.0`. Acest lucru arată că e posibil să specifici multiple identificatoare de licență, separate prin `OR`, pentru a oferi mai multe opțiuni de licențiere pentru proiectul tău.

Cu un nume unic, o versiune, o descriere adăugată și o licență definită, fișierul *Cargo.toml* al unui proiect pregătit pentru publicare poate arăta în felul următor:

<span class="filename">Numele fișierului: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "Un joc distractiv unde încerci să ghicești numărul ales de calculator."
license = "MIT OR Apache-2.0"

[dependencies]
```

Poți găsi în [documentația Cargo](https://doc.rust-lang.org/cargo/) detalii suplimentare despre alte metadate care îmbunătățesc descoperirea și utilizarea crate-ului tău de către alții.

### Publicarea pe crates.io

Acum că ți-ai creat un cont, ai salvat cheia API, ai ales un nume pentru crate-ul tău și ai specificat toate metadatele necesare, ești pregătit să publici! Publicarea unui crate trimite o versiune specifică pe [crates.io](https://crates.io/)<!-- ignore --> pentru utilizare de către alții.

Trebuie să fii precaut, deoarece o publicare este *ireversibilă*. Versiunea nu poate fi înlocuită, iar codul nu poate fi retras. Unul dintre principalele obiective ale [crates.io](https://crates.io/)<!-- ignore --> este să servească ca o arhivă permanentă de cod, pentru ca toate proiectele care depind de crate-uri de pe [crates.io](https://crates.io/)<!-- ignore --> să poată fi construite și în viitor. Permițând ștergerea de versiuni, am împiedica atingerea acestui obiectiv. Cu toate acestea, poți publica un număr nelimitat de versiuni ale crate-ului tău.

Execută din nou comanda `cargo publish`. De data aceasta ar trebui să fie cu succes:

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

Felicitări! Codul tău a fost acum distribuit comunității Rust, astfel încât toți pot să includă foarte ușor crate-ul tău ca dependență în proiectele lor.

### Publicarea unei versiuni noi a unui crate existent

Când ai efectuat modificări în crate-ul tău și ești pregătit să faci o versiune nouă, trebuie să modifici valoarea `version` din fișierul tău *Cargo.toml* și să re-publici. Urmărește [regulile versionării semantice][semver] ca să determini care ar trebui să fie numărul de versiune adecvat, bazat pe modificările efectuate. Apoi execută `cargo publish` pentru a încărca noua versiune.

### Retragerea versiunilor de pe crates.io folosind `cargo yank`

Cu toate că nu poți șterge versiunile anterioare ale unui crate, poți să previi adăugarea lor ca o nouă dependență de către proiecte viitoare. Acest lucru este de ajutor atunci când o versiune a unui crate este deficientă dintr-un oarecare motiv. În asemenea situații, Cargo oferă posibilitatea de a *retrage* o versiune de crate.

Retragerea unei versiuni împiedică proiectele noi să depindă de acea versiune, permițând în același timp tuturor proiectelor existente care o folosesc să continue fără probleme. În esență, a retrage o versiune semnifică faptul că toate proiectele cu un *Cargo.lock* deja existent nu vor avea de suferit și niciun viitor fișier *Cargo.lock* nou-generat nu va utiliza versiunea retrasă.

Pentru a retrage o versiune a unui crate, în directoriul unde crate-ul a fost publicat inițial, execută `cargo yank` și indică versiunea pe care dorești să o retragi. Spre exemplu, dacă am publicat un crate denumit `guessing_game` la versiunea 1.0.1 și dorim să o retragem, în directoriul proiectului `guessing_game` am executa:

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Prin adăugarea `--undo` la comandă, poți de asemenea anula o retragere și permite din nou proiectelor noi să depindă de acea versiune:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

O retragere *nu* implică ștergerea vreunui cod. De pildă, nu poate elimina secretele încărcate din greșeală. Dacă se întâmplă acest lucru, trebuie să resetezi imediat respectivele secrete.

[spdx]: http://spdx.org/licenses/
[semver]: http://semver.org/
