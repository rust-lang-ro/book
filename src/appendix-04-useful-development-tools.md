## Anexa D - Unelte utile de dezvoltare

În această anexă, vorbim despre unele unelte utile de dezvoltare pe care proiectul Rust le oferă. Vom analiza formatarea automată, modalități rapide de a aplica corecțiile de avertizare, un linter și integrare cu IDE-urile.

### Formatare automată cu `rustfmt`

Instrumentul `rustfmt` reformatează codul tău conform stilului de codare al comunității. Multe proiecte colaborative utilizează `rustfmt` pentru a preveni dispute cu privire la stilul de utilizat atunci când se scrie în Rust: toată lumea formatează codul lor folosind acest instrument.

Pentru a instala `rustfmt`, introdu următorul text:

```console
$ rustup component add rustfmt
```

Această comandă îți oferă `rustfmt` și `cargo-fmt`, similar cum Rust îți oferă atât `rustc` cât și `cargo`. Pentru a formata orice proiect Cargo, introduce următorare comandă:

```console
$ cargo fmt
```

Rularea acestei comenzi reformatează tot codul Rust din crate-ul curent. Aceasta ar trebui să schimbe doar stilul codului, nu și semantica acestuia. Pentru mai multe informații despre `rustfmt`, consultați [documentația sa][rustfmt].

[rustfmt]: https://github.com/rust-lang/rustfmt

### Corectează-ți codul cu `rustfix`

Instrumentul rustfix este inclus în instalările Rust și poate corecta în mod automat avertismentele compilatorului care au o modalitate clară de corectare, care și este probabil ceea ce vrei. Este posibil să fi văzut avertismente de compilator înainte. De exemplu, ia în considerare acest cod:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
fn do_something() {}

fn main() {
    for i in 0..100 {
        do_something();
    }
}
```

Aici, apelăm funcția `do_something` de 100 de ori, dar nu folosim niciodată variabila `i` în corpul buclei `for`. Rust ne avertizează despre acest lucru:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: unused variable: `i`
 --> src/main.rs:4:9
  |
4 |     for i in 0..100 {
  |         ^ help: consider using `_i` instead
  |
  = note: #[warn(unused_variables)] on by default

    Finished dev [unoptimized + debuginfo] target(s) in 0.50s
```

Avertismentul sugerează să folosim `_i` ca nume în schimb: linia de subliniere indică faptul că intenționăm ca această variabilă să rămână nefolosită. Putem aplica automat această sugestie folosind instrumentul `rustfix` rulând comanda `cargo fix`:

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Când ne uităm din nou la *src/main.rs*, vedem că `cargo fix` a modificat codul:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
fn do_something() {}

fn main() {
    for _i in 0..100 {
        do_something();
    }
}
```

Variabila buclei `for` se numește acum `_i`, iar avertismentul nu mai apare.

De asemenea, poți utiliza comanda `cargo fix` pentru a trece codul tău între diferitele ediții ale Rust. Edițiile sunt descrise în Anexa E.

### Mai multe lints cu Clippy

Instrumentul Clippy este o colecție de lints care ajută la analiza codului tău Rust. „Lints” se referă la reguli de verificare statică a codului sursă, folosite pentru a identifica erori, inconsecvențe de stil și posibile probleme de performanță sau securitate. Folosind Clippy, poți depista și remedia eficient aceste probleme comune, sporind astfel calitatea codului tău Rust.

Pentru a instala Clippy, introdu următoarea comandă:

```console
$ rustup component add clippy
```

Pentru a rula lints de la Clippy pe orice proiect Cargo, introdu următoarea comandă:

```console
$ cargo clippy
```

De exemplu, presupunem că scrii un program care folosește o aproximare a unei constante matematice, precum pi, așa cum face acest program:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

Rularea `cargo clippy` pe acest proiect duce la această eroare:

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

Aceasta eroare îți arată că Rust are deja o constantă `PI` mai precisă definită și că programul tău ar fi mai corect dacă ai folosi constanta respectivă. Atunci ai schimba codul tău pentru a utiliza constanta `PI`. Următorul cod nu duce la nicio eroare sau avertisment de la Clippy:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

Pentru mai multe informații despre Clippy, vezi [documentația sa][clippy].

[clippy]: https://github.com/rust-lang/rust-clippy

### Integrarea cu IDE folosind `rust-analyzer`

Pentru a facilita integrarea cu IDE, comunitatea Rust recomandă utilizarea
[`rust-analyzer`][rust-analyzer]<!-- ignore -->. Acest instrument este un set de
utilitare centrate pe compilator care folosesc protocolul [Language Server Protocol][lsp]<!-- ignore -->, care este o specificație pentru ca IDE-urile și limbajele de programare să comunice între ele. Diferiți clienți pot folosi `rust-analyzer`, cum ar fi
[plugin-ul Rust analyzer pentru Visual Studio Code][vscode].

[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer

Vizitează pagina de [acasa][rust-analyzer] a proiectului `rust-analyzer` <!-- ignore -->
pentru instrucțiuni de instalare, apoi instalează suportul pentru serverul de limbaj în propriul tău IDE. IDE-ul tău va obține abilități precum autocompletare, sări la definiție, și erorile inline.

[rust-analyzer]: https://rust-analyzer.github.io
