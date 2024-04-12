## Gestionarea erorilor irecuperabile cu `panic!`

În codul tău pot apărea situații neașteptate și iremediabile. Pentru aceste cazuri, Rust oferă macro-ul `panic!`. Poți declanșa o panică în două moduri: fie realizând o acțiune care duce la panică în codul nostru - de exemplu, accesând un array dincolo de capacitatea sa, fie apelând direct macro-ul `panic!`. Ambele variante vor provoca o panică în programul nostru care, implicit, va afișa un mesaj de eroare, va desface stiva, va efectua curățarea acesteia și va încheia execuția programului. Prin setarea unei variabile de mediu, poți de asemenea instrui Rust să afișeze stiva de apeluri în momentul unei panici, facilitând astfel identificarea cauzei problemei.

> ### Gestionarea panicii: Dezactivarea stivei sau abandonarea programului 
>
> Implicit, când se declanșează o panică, programul începe procesul de
> *dezactivare a stivei* (unwinding): Rust urcă stiva invers și eliberează
> memoria ocupată de datele din fiecare funcție întâlnită. Totuși, acest
> procedeu este destul de laborios. De aceea, Rust îți oferă opțiunea de
> *abandonare*, care oprește programul direct, fără a elibera resursele
> utilizate.
> În acest caz, sistemul de operare va trebui să curețe memoria folosită de
> program. Dacă ai nevoie să minimizezi dimensiunea executabilului în
> proiectul tău, poți opta pentru încheierea abruptă a programului în caz de
> panică prin adăugarea liniei `panic = 'abort'` în secțiunile `[profile]`
> relevante din fișierul *Cargo.toml*. De exemplu, pentru a seta această
> opțiune în modul Release, inserează:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Să folosim `panic!` într-un program simplu:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

Când executăm programul, vom observa un mesaj similar cu acesta:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

Utilizarea `panic!` generează mesajul de eroare afișat în ultimele două rânduri. Primul rând îți prezintă mesajul nostru de panică, precum și locul din sursa noastră unde panica a avut loc: *src/main.rs:2:5* ne indică faptul că problema se află la linia a doua, caracterul cinci din fișierul nostru *src/main.rs*.

În exemplul nostru, linia indicată ne duce direct la codul scris de noi, unde găsim apelul macro-ului `panic!`. Cu toate acestea, uneori apelul `panic!` s-ar putea să fie în cod la care codul nostru face referire și atunci, numele fișierului și numărul liniei raportate de mesajul de eroare vor semnala locația din codul altei persoane unde a fost invocat `panic!`, nu punctul din codul nostru care a cauzat în ultimă instanță invocarea `panic!`. Putem să folosim backtrace-ul pentru a urmări din ce funcții vine apelul `panic!` și să identificăm astfel partea din codul nostru responsabilă pentru eroare. Vom discuta backtrace-urile mai în detaliu în curând.

### Urmărirea stivei cu `panic!`

Să analizăm un exemplu în care `panic!` este declanșat de o bibliotecă din pricina unei greșeli în codul nostru, în loc de a proveni direct din codul care invocă macrocomanda. Listarea 9-1 prezintă cod care încearcă să acceseze un index inexistent într-un vector, dincolo de limita indexurilor valide.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

<span class="caption">Listarea 9-1: Tentativa de accesare a unui element în afara limitelor unui vector, ce va declanșa un apel la `panic!`</span>

În acest caz, încercăm să accesăm elementul cu indexul 100 din vectorul nostru (care ar fi de fapt la indexul 99, deoarece numerotarea începe de la zero), însă vectorul nostru conține doar 3 elemente. In această situație, Rust va apela panica. Folosirea operatorului `[]` presupune a extrage un element, dar când indexul este incorect, Rust nu poate furniza un răspuns adecvat.

În limbajul C, citirea datelor dincolo de capătul unei structuri de date este un comportament nedefinit. S-ar putea să primești date aleatoare din memoria ce corespunde acelui index în structura de date, chiar dacă acea porțiune de memorie nu face parte din structură. Acest fenomen, cunoscut sub numele de *supracitire a buffer-ului*, poate crea vulnerabilități de securitate dacă un atacator reușește să modifice indexul în așa fel încât să citească date neautorizate, plasate în memorie după structura de date.

Pentru a proteja programul tău de vulnerabilități, Rust va opri execuția dacă încerci să accesezi un element la un index inexistent, refuzând astfel să continue. Să vedem cum se întâmplă asta:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Mesajul de eroare arată că greșeala se află la linia 4 din fișierul nostru `main.rs`, unde am încercat să accesăm elementul cu indexul 99. Nota atașată ne spune că putem seta variabila de mediu `RUST_BACKTRACE` pentru a primi un backtrace detaliat, care să ne arate exact ce a cauzat eroarea. Un backtrace reprezintă o listă cu toate funcțiile invocate până în punctul erorii. Interpretarea unui backtrace în Rust se face la fel ca în alte limbaje: pornește de la începutul listei și continuă să citești până când recunoști fișiere pe care le-ai scris tu. Acolo se găsește originea problemei. Liniile care preced această marcă sunt codul apelat de codul tău; cele care urmează sunt cele care l-au apelat pe al tău. Aceasta implică posibil codul fundamental Rust, librăriile standard sau crate-urile utilizate. Să obținem un backtrace setând variabila de mediu `RUST_BACKTRACE` la orice altceva decât 0. În Listarea 9-2 vei găsi un exemplu de afișaj similar cu cel pe care îl vei întâlni.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/std/src/panicking.rs:584:5
   1: core::panicking::panic_fmt
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:142:14
   2: core::panicking::panic_bounds_check
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:84:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:242:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/alloc/src/vec/mod.rs:2591:9
   6: panic::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/ops/function.rs:248:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

<span class="caption">Listarea 9-2: Backtrace generat de `panic!`, vizibil când variabila de mediu `RUST_BACKTRACE` este activă</span>

Este o mulțime de informație afișată! Afișajul pe care îl vei vedea poate varia în funcție de sistemul de operare și versiunea de Rust utilizată. Pentru a obține backtrace-uri detaliate ca acestea, este necesar ca simbolurile de depanare să fie activate. Aceste simboluri sunt activate automat când folosești `cargo build` sau `cargo run` fară opțiunea `--release`, cum este cazul aici.

În afișajul din Listarea 9-2, linia 6 din backtrace ne conduce direct la linia din proiectul nostru care generează problema: linia 4 din *src/main.rs*. Dacă dorim să evităm panica în program, ar trebui să începem analiza acolo unde prima linie indică un fișier creat de noi. De exemplu, în Listarea 9-1, unde codul a fost scris intenționat pentru a genera o panică, soluția pentru a evita aceasta este să nu accesăm un element dintr-un vector ce depășește limitele indicilor acestuia. Când codul tău generează o panică în viitor, va trebui să identifici care acțiune și valori conduc la acea panică și ce ar trebui de fapt să facă codul tău.

Vom reveni la discuția despre `panic!`, când ar trebui și când nu ar trebui să folosim `panic!` pentru a trata erorile, în secțiunea [„Când să folosim `panic!` și când nu”][to-panic-or-not-to-panic] mai târziu în acest capitol. În continuare, vom explora cum să gestionăm recuperarea dintr-o eroare folosind `Result`.

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
