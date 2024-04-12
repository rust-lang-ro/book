## Personalizarea build-urilor cu profile de release

În Rust, *profilele de release* (release profiles) sunt profile predefinite și customizabile care oferă programatorului control sporit asupra diverselor opțiuni de compilare. Fiecare profil este configurat independent față de celelalte.

Cargo pune la dispoziție două profile principale: profilul `dev`, activat de Cargo la executarea comenzii `cargo build`, și profilul `release`, folosit de Cargo când lansezi `cargo build --release`. Profilul `dev` este setat cu valori implicite optimizate pentru dezvoltare, iar profilul `release` are valori implicite optimizate pentru versiunile de publicare finală.

S-ar putea să recunoști aceste nume de profile din afișajele build-urilor efectuate:

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
$ cargo build --release
    Finished release [optimized] target(s) in 0.0s
```

`dev` și `release` sunt aceste profile diferite utilizate de compilatorul Rust.

Cargo are o serie de setări standard pentru fiecare profil, care intră în vigoare dacă nu există secțiuni `[profile.*]` definite explicit în fișierul *Cargo.toml*. Personalizând secțiunile `[profile.*]`, poți modifica orice subansamblu din aceste setări standard. Iată valorile implicite pentru setarea `opt-level` ale profilelor `dev` și `release`:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

Parametrul `opt-level` determină numărul de optimizări aplicate codului tău de către Rust, cu o gamă de la 0 la 3. Aplicarea mai multor optimizări prelungește timpul de compilare, de aceea, în faza de dezvoltare unde codul este compilat frecvent, este de preferat să folosești mai puține optimizări pentru a avea un timp de compilare scurt, chiar dacă codul executabil va fi mai lent. Prin urmare, valoarea implicită a `opt-level` pentru profilul `dev` este `0`. În schimb, când ești pregătit să lansezi codul, este recomandat să aloci mai mult timp compilării. Codul în modul release va fi compilat o singură dată, însă va fi rulat de nenumărate ori, preferându-se astfel un timp de compilare mai mare pentru a obține un cod executabil mai rapid. Din acest motiv, valoarea implicită pentru `opt-level` în profilul `release` este `3`.

Poți modifica o setare implicită adăugând o valoare diferită în *Cargo.toml*. Să zicem că dorim să utilizăm nivelul de optimizare 1 în profilul de dezvoltare. În acest caz, putem adăuga următoarele două linii în fișierul *Cargo.toml* al proiectului:

<span class="filename">Numele fișierului: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

Codul de mai sus suprascrie setarea implicită de `0`. Acum, la executarea comenzii `cargo build`, Cargo va implementa valorile implicite ale profilului `dev` împreună cu ajustarea făcută de noi la `opt-level`. Setând `opt-level` la valoarea `1`, Cargo va aplica mai multe optimizări decât cele implicite, dar mai puține decât într-o compilare destinată profilului release.

Pentru lista completă a opțiunilor de configurare și a valorilor implicite pentru fiecare profil, consultă [documentația Cargo](https://doc.rust-lang.org/cargo/reference/profiles.html).
