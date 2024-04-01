## Salut, Cargo!

Cargo este sistemul de construcție și managerul de pachete al lui Rust. Majoritatea programatorilor Rust folosesc acest instrument pentru a-și gestiona proiectele Rust deoarece Cargo se ocupă de o mulțime de sarcini pentru tine, precum compilarea codului, descărcarea librăriilor de care depinde el și compilarea acestor librării. (Noi, de regulă, numim librăriile *dependențe*.)

Cele mai simple programe Rust, ca cel pe care l-am scris până acum, nu au nicio dependență. Dacă am fi compilat proiectul „Salut, lume!” cu Cargo, atunci doar partea responsabilă de compilare din Cargo ar fi fost utilizată. Pe măsură ce proiectele tale Rust devin mai complexe, este probabil să adaugi mai multe dependențe și dacă începi un proiect folosind Cargo, adăugarea dependențelor va fi mult mai ușor de făcut.

Deoarece majoritatea covârșitoare a proiectelor Rust folosesc Cargo, în restul acestei cărți vom presupune că și tu folosești Cargo. Cargo vine instalat cu Rust dacă ai folosit instalatoarele oficiale discutate în secțiunea [„Instalare”][installation]<!-- ignore -->. Dacă ai instalat Rust prin alte mijloace, poți verifica dacă Cargo este instalat prin introducerea următoarei comenzi în terminal:

```console
$ cargo --version
```

Dacă vezi un număr de versiune, îl ai! Dacă vezi o eroare, cum ar fi `command not found`, uită-te la documentația metodei tale de instalare pentru a determina cum să instalezi Cargo separat.

### Crearea unui proiect cu Cargo

Să creăm un proiect nou folosind Cargo și să vedem cum se deosebește de proiectul nostru original „Salut, lume!”. Navighează înapoi în directoriul tău *proiecte* (sau oriunde ai ales să-ți stochezi codul). Apoi, pe orice sistem de operare, rulează următoarele:

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

Prima comandă creează un directoriu și un proiect nou numit *hello_cargo*. Noi am numit proiectul nostru *hello_cargo*, iar Cargo creează fișierele sale într-un directoriu cu același nume.

Du-te în directoriul *hello_cargo* și listează fișierele. Vei vedea că Cargo a generat două fișiere și un directoriu: un fișier *Cargo.toml* și un directoriu *src* cu un fișier *main.rs* în interior.

A creat de asemenea un nou repozitoriu Git împreună cu un fișier *.gitignore*. Cargo nu va iniția un nou repozitoriu Git dacă comanda `cargo new` este rulată în interiorul unui repozitoriu Git existent; poți anula această comportare folosind `cargo new --vcs=git`.

> Notă: Git este un sistem popular de control al versiunilor. Poți modifica
> `cargo new` pentru a folosi un alt sistem de control al versiunilor sau
> niciun sistem de control al versiunilor folosind comutația `--vcs`. Rulează
> `cargo new --help` pentru a vedea opțiunile disponibile.

Deschide *Cargo.toml* în editorul tău de text preferat. Ar trebui să arate similar cu codul din Listarea 1-2.

<span class="filename">Numele fișierului: Cargo.toml</span>

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

<span class="caption">Listarea 1-2: Conținutul lui *Cargo.toml* generat de `cargo new`</span>

Acest fișier este în formatul [*TOML*][toml]<!-- ignore --> (*Tom’s Obvious, Minimal Language*), care este formatul de configurare al lui Cargo.

Prima linie, `[package]`, este o rubrică de secțiune care indică faptul că declarațiile următoare configurează un pachet. Pe măsură ce adăugăm mai multe informații la acest fișier, vom adăuga și alte secțiuni.

Următoarele trei linii setează informațiile de configurare de care Cargo are nevoie pentru a compila programul tău: numele, versiunea și ediția Rust pe care să o folosească. Vom vorbi despre cheia `edition` în [Anexa E][appendix-e]<!-- ignore -->.

Ultima linie, `[dependencies]`, este începutul unei secțiuni în care poți enumera orice dependențe ale proiectului tău. În Rust, pachetele de cod sunt numite *crates*. Nu vom avea nevoie de alte crate-uri pentru acest proiect, dar vom avea în primul proiect din Capitolul 2, deci vom folosi această secțiune de dependențe atunci.

Acum deschide *src/main.rs* și aruncă o privire:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
fn main() {
    println!("Salut, lume!");
}
```

Cargo tocmai a generat un program „Salut, lume!”, identic cu cel pe care l-am realizat noi în Listarea 1-1! Până acum, principalele diferențe dintre proiectul nostru și cel creat de Cargo sunt așezarea codului în directoriul *src* și existența unui fișier de configurare *Cargo.toml* în directoriul rădăcină.

Cargo așteaptă ca fișierele sursă să fie poziționate în interiorul directoriului *src*. Directoriul rădăcină al proiectului este destinat doar pentru fișierele README, informațiile despre licență, fișiere de configurare și orice altceva care nu este direct legat de cod. Utilizarea Cargo contribuie la organizarea eficientă a proiectelor tale, având un loc bine definit pentru fiecare componentă și asigurându-se că toate componentele sunt la locul potrivit.

Dacă ai început un proiect fără să folosești Cargo, așa cum am procedat noi cu proiectul „Salut, lume!”, poți să îl transformi într-un proiect care utilizează Cargo. Trebuie să muti codul în directoriul *src* și să creezi un fișier *Cargo.toml* adecvat.

### Compilarea și rularea unui proiect Cargo

Acum să ne uităm la ce e diferit când compilăm și rulăm programul „Salut, lume!” cu Cargo! Din directoriul tău *hello_cargo*, compilează proiectul prin introducerea următoarei comenzi:

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Această comandă creează un fișier executabil în *target/debug/hello_cargo* (sau
*target\debug\hello_cargo.exe* pe Windows) mai degrabă decât în directoriul tău curent. Pentru că build-ul implicit este un build de depanare, Cargo pune binarul într-un directoriu numit *debug*. Poți rula executabilul cu această comandă:

```console
$ ./target/debug/hello_cargo # sau .\target\debug\hello_cargo.exe pe Windows
Salut, lume!
```

Dacă totul merge conform planului, `Hello, world!` ar trebui să fie afișat în terminal. Folosirea comenzii `cargo build` pentru prima oară, de asemenea, determină Cargo să creeze un fișier nou la nivelul cel mai de sus: *Cargo.lock*. Acest fișier ține evidența versiunilor exacte ale dependențelor din proiectul tău. Proiectul acesta nu are dependențe, așadar fișierul este destul de simplu. Nu va fi necesar să intervenim manual asupra acestui fișier; Cargo va gestiona conținutul pentru noi.

Am compilat un proiect folosind `cargo build` și l-am executat cu `./target/debug/hello_cargo`, însă putem de asemenea să utilizăm comanda `cargo run` pentru a compila codul și apoi a executa fișierul rezultat, totul printr-o singură comandă:


```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Salut, lume!
```

Utilizarea `cargo run` este mai convenabilă decât să trebuiască să ne amintim să executăm `cargo build` și apoi să accedem toată calea către executabil, așa că majoritatea dezvoltatorilor preferă `cargo run`.

Observă că de această dată nu am văzut un afișaj care să indice dacă Cargo a compilat `hello_cargo`. Cargo a înțeles că fișierele nu au suferit modificări, și prin urmare, nu a necesitat o recompilare ci doar a executat binarul. Dacă ai modificat codul sursă, Cargo ar fi recompilat proiectul înainte de a-l executa, și ai fi observat acest afișaj:

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Salut, lume!
```

Cargo oferă de asemenea o comandă numită `cargo check`. Această comandă verifică rapid codul tău pentru a se asigura că se compilează, dar nu produce un executabil:

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

De ce nu ai vrea un executabil? Adesea, `cargo check` este mult mai rapidă decât `cargo build` pentru că omite pasul de producere a unui executabil. Dacă verifici în continuu ce lucrezi în timp ce scrii codul, folosirea `cargo check` va accelera procesul de a te informa dacă proiectul tău încă se compilează! Astfel, mulți Rustaceani rulează `cargo check` periodic în timp ce scriu programul lor pentru a se asigura că se compilează. Apoi ei rulează `cargo build` când sunt gata să folosească executabilul.

Să recapitulăm ce am învățat până acum despre Cargo:

* Putem crea un proiect folosind `cargo new`.
* Putem compila un proiect folosind `cargo build`.
* Putem compila și rula un proiect într-un singur pas folosind `cargo run`.
* Putem compila un proiect fără a produce un binar pentru a verifica erorile folosind `cargo check`.
* În loc sa salvăm rezultatul build-ului in același directoriu cu codul nostru, Cargo il stocheaza in directoriul *target/debug*.

Un avantaj suplimentar al folosirii lui Cargo este că comenzile sunt la fel, indiferent de sistemul de operare pe care lucrezi. Așadar, de la acest punct nu vom mai oferi instrucțiuni specifice pentru Linux și macOS versus Windows.

### Compilarea pentru lansare

Când proiectul tău este în sfârșit gata de lansare, poți utiliza `cargo build --release` pentru a-l compila cu optimizări. Această comandă va crea un executabil în *target/release* în loc de *target/debug*. Optimizările fac codul tău Rust să ruleze mai rapid, dar activarea lor mărește timpul necesar pentru compilarea programului. Din acest motiv există două profiluri diferite: unul pentru dezvoltare, când vrei să recompilezi repede și des, și altul pentru compilarea programului final pe care îl vei oferi unui utilizator care nu va fi recompilat în mod repetat și care va rula cât mai rapid posibil. Dacă îți evaluezi timpul de rulare al codului, nu uita să folosești comanda `cargo build --release` și să realizezi evaluarea cu executabilul din *target/release*

### Cargo în calitate de convenție

Pentru proiecte simple, Cargo nu oferă multă valoare peste utilizarea directă a lui `rustc`, dar își va demonstra valoarea pe măsură ce programele tale devin mai complexe. Odată ce programele cresc la mai multe fișiere sau au nevoie de o dependență, este mult mai ușor să lași Cargo să coordoneze compilarea.

Deși proiectul `hello_cargo` este simplu, acum folosește o mare parte din instrumentele reale pe care le vei folosi în restul carierei tale Rust. De fapt, pentru a lucra la orice proiecte existente, poți folosi următoarele comenzi pentru a verifica codul folosind Git, pentru a schimba directoriul către acel proiect și pentru a compila:

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Pentru mai multe informații despre Cargo, vezi [documentația sa][cargo].

## Sumar

Ești deja pe drumul cel bun în călătoria ta cu Rust! În acest capitol, ai învățat să:

* Instalezi cea mai recentă versiune stabilă a Rust folosind `rustup`
* Actualizezi la o versiune mai nouă a Rust
* Deschizi documentația instalată local
* Scrii și rulezi un program „Salut, lume!” folosind `rustc` direct
* Creezi și rulezi un proiect nou folosind convențiile Cargo

Acesta este momentul perfect pentru a crea un program mai substanțial pentru a te familiariza cu citirea și scrierea codului Rust. Așadar, în Capitolul 2, vom scrie un program de joc de ghicit. Dacă ai prefera mai degrabă să începi prin a învăța cum funcționează conceptele comune de programare în Rust, vezi Capitolul 3 și apoi întoarce-te la Capitolul 2.

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
