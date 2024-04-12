## Tipuri avansate

Sistemul de tipuri din Rust include unele caracteristici care ne-au fost deja prezentate, dar nu le-am explorat în profunzime până acum. Vom începe prin a discuta despre pattern-ul newtype în general, analizând motivul pentru care tipurile newtype sunt utile. Apoi vom aborda pseudonimele de tip, o funcționalitate similară cu newtype, dar cu semantici ușor diferite. De asemenea, vom discuta despre tipul `!` și despre tipurile cu dimensiune dinamică.

### Utilizarea pattern-ului newtype pentru siguranța și abstractizarea tipului

> Notă: Această secțiune presupune că ai citit secțiunea anterioară [„Utilizarea pattern-ului newtype pentru implementarea trăsăturilor externe pe tipuri externe”][using-the-newtype-pattern]<!-- ignore -->

Pattern-ul newtype este de asemenea util în alte situații decât cele pe care le-am tratat până acum, servind atât pentru a ne asigura că valorile nu sunt confundate între ele în mod static, cât și pentru a indica unitățile unei valori. Am văzut un exemplu în care tipurile newtype sunt folosite pentru a indica unitățile în Listarea 19-15: să ne amintim că structurile `Millimeters` și `Meters` conțineau valori `u32` învelite în newtype. Dacă am compune o funcție cu un parametru de tip `Millimeters`, programul nostru nu ar compila dacă am încerca să apelăm acea funcție cu o valoare de tip `Meters` sau un `u32` simplu.

În plus, putem folosi pattern-ul newtype pentru a abstractiza anumite detalii de implementare ale unui tip: noul tip poate expune o interfață API publică diferită de cea a tipului intern privat.

Newtype este folositor și pentru a masca implementarea internă. De exemplu, am putea crea un tip `People` pentru a împacheta un `HashMap<i32, String>` care păstrează ID-ul unei persoane asociat cu numele acesteia. Codul care interacționează cu `People` s-ar limita doar la interfața API publică pe care o furnizăm, cum ar fi o metodă pentru adăugarea unui șir de caractere reprezentând numele în colecția `People`; acest cod nu ar avea nevoie să cunoască faptul că intern folosim un ID de tip `i32` pentru nume. Pattern-ul newtype este o cale eficientă de a atinge încapsularea pentru a masca detalii de implementare, un aspect pe care l-am discutat în secțiunea [„Încapsularea care ascunde detaliile implementării”][encapsulation-that-hides-implementation-details]<!-- ignore --> din Capitolul 17.

### Crearea sinonimelor de tip prin aliasuri de tip

Rust ne permite să declarăm un *alias de tip* pentru a oferi unui tip existent un nume alternativ. Facem acest lucru utilizând cuvântul cheie `type`. De exemplu, putem crea aliasul `Kilometers` pentru `i32` astfel:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Aliasul `Kilometers` acum funcționează ca un *sinonim* pentru `i32`; în contrast cu tipurile `Millimeters` și `Meters` pe care le-am creat anterior, în Listarea 19-15, `Kilometers` nu constituie un tip nou și separat. Valorile de tipul `Kilometers` vor fi tratate identic cu valorile de tip `i32`:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

Fiindcă `Kilometers` și `i32` sunt de fapt același tip, putem adăuga împreună valori de ambele tipuri și putem transmite valori de tip `Kilometers` către funcții care acceptă parametrii de tip `i32`. Cu toate acestea, această metodă nu ne conferă avantajele verificării de tipuri pe care le avem cu modelul newtype, despre care am discutat anterior. Adică, dacă am confunda valorile `Kilometers` cu cele `i32` într-un anumit loc, compilatorul nu va genera eroare.

Utilizarea principală a sinonimelor de tip este de a diminua repetiția. De pildă, putem să ne confruntăm cu un tip complicat ca acesta:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Scriind acest tip extins în semnăturile funcțiilor și ca adnotări de tip de-a lungul codului poate fi anevoios și susceptibil de erori. Imaginează-ți un proiect plin cu cod similar acelui din Listarea 19-24.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-24/src/main.rs:here}}
```

<span class="caption">Listarea 19-24: Folosirea unui tip lung în numeroase locuri</span>

Un alias de tip simplifică gestionarea acestui cod prin reducerea frecvenței repetării. În Listarea 19-25, am introdus aliasul `Thunk` pentru tipul lung și acum putem înlocui toate aparițiile acelui tip cu aliasul mai concis `Thunk`.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-25/src/main.rs:here}}
```

<span class="caption">Listarea 19-25: Introducerea unui alias de tip `Thunk` pentru diminuarea repetiției</span>

Acest cod este mult mai simplu de citit și scris! Selectarea unui nume sugestiv pentru un alias de tip poate contribui și la transmiterea intenției noastre (termenul *thunk* se referă la codul care va fi evaluat ulterior, deci este un nume adecvat pentru o închidere care este păstrată încă neevaluată).

Aliasurile de tip sunt frecvent utilizate și împreună cu tipul `Result<T, E>` pentru a diminua repetiția. Să luăm ca exemplu modulul `std::io` din biblioteca standard. Operațiile de I/O returnează de obicei un `Result<T, E>` pentru a aborda cazurile când aceste operații eșuează. Biblioteca conține structura `std::io::Error`, care reprezintă toate erorile de I/O posibile. Multe dintre funcțiile din `std::io` vor returna un `Result<T, E>` unde `E` este `std::io::Error`, cum ar fi funcțiile din trăsătura `Write`:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

Expresia `Result<..., Error>` se repetă deseori. Drept urmare, `std::io` declară acest alias de tip:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Fiindcă această declarație se află în modulul `std::io`, putem utiliza aliasul complet specificat `std::io::Result<T>`; adică, un `Result<T, E>` unde `E` este specificat drept `std::io::Error`. Funcțiile din semnăturile trăsăturii `Write` capătă în cele din urmă următoarea înfățișare:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

Aliasul de tip ne ajută în două feluri: îmbunătățește concizia codului *și* oferă o interfață coerentă întregului modul `std::io`. Deoarece este un alias, rămâne pur și simplu un alt `Result<T, E>`, ceea ce înseamnă că putem aplica asupra lui orice metodă compatibilă cu `Result<T, E>`, inclusiv folosirea de sintaxă specială, cum ar fi operatorul `?`.

### Tipul `never`` care nu returnează niciodată

Rust dispune de un tip special numit `!`, cunoscut în terminologia teoriei tipurilor ca *tipul gol* deoarece nu posedă nicio valoare. Totuși, preferăm să-l numim *tipul never* deoarece este utilizat în locul tipului de retur când o funcție nu va returna niciodată. Iată un exemplu:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Acest cod se citește ca „funcția `bar` returnează never.” Funcțiile care returnează never sunt numite *funcții divergente*. Imposibilitatea creării de valori de tipul `!` implică faptul că `bar` nu va putea returna niciodată.

Dar care este utilitatea unui tip pentru care nu putem crea valori? Vom reaminti codul din Listarea 2-5, care face parte din jocul de ghicit numerele; iată o reproducere parțială în Listarea 19-26:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

<span class="caption">Listarea 19-26: Un `match` cu o ramură care se încheie cu `continue`</span>

Pe atunci, am omis detalii importante ale acestui cod. În Capitolul 6, secțiunea [„Operatorul de control al fluxului `match`”][the-match-control-flow-operator]<!-- ignore -->, am explicat că toate ramurile `match` trebuie să returneze același tip. Astfel, codul următor nu este funcțional:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

Tipul lui `guess` de aici ar trebui să fie atât integer cât și string, dar Rust afirmă că `guess` trebuie să aibă un singur tip definit. Atunci, ce returnează `continue`? Cum a fost posibil să returnăm `u32` dintr-o ramură și o altă ramură să se încheie cu `continue` în Listarea 19-26?

După cum probabil ai dedus, `continue` are valoarea `!`. Adică, când Rust determină tipul lui `guess`, analizează ambele ramuri ale `match`-ului, prima cu o valoare `u32` și a doua cu valoarea `!`. Din moment ce `!` nu poate avea vreo valoare, Rust decide că tipul lui `guess` este `u32`.

Descrierea formală a acestui comportament este că expresiile de tip `!` pot fi transformate în orice alt tip. Este permis să finalizăm această ramură de `match` cu `continue` deoarece `continue` nu produce o valoare; în schimb, redirecționează controlul la începutul buclei, așadar în cazul `Err`, `guess` nu primește nicio valoare.

Tipul never este de asemenea valoros în contextul macro-ului `panic!`. Să ne amintim de funcția `unwrap` pe care o utilizăm pe valori de tip `Option<T>` pentru a obține o valoare sau a genera panică; iată definiția ei:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

Aici, se întâmplă exact ca în cazul `match`-ului din Listarea 19-26: Rust observă că `val` are tipul `T` și `panic!` are tipul `!`, astfel rezultatul întregii expresii `match` este `T`. Acest cod este valid pentru că `panic!` nu generează o valoare; el încheie execuția programului. În cazul `None`, nu vom returna o valoare din `unwrap`, deci acest fragment de cod este corect.

O ultimă expresie care are valoarea `!` este `loop`:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

În acest caz, bucla nu se finalizează niciodată, prin urmare `!` este valoarea expresiei. Totuși, aceasta nu ar fi adevărat dacă am adăuga un `break`, întrucât bucla s-ar opri odată ce s-ar ajunge la `break`.

### Tipuri dinamic dimensionate și trăsătura `Sized`

Rust trebuie să cunoască detalii specifice despre tipurile sale, cum ar fi cantitatea de spațiu necesară alocării pentru o valoare de un anumit tip. Acest lucru crează o anumită confuzie în sistemul de tipuri, la prima vedere: conceptul de *tipuri dinamic dimensionate* (dynamically sized types, DST) sau *tipuri fără mărime fixă*. Aceste tipuri ne permit să scriem cod folosind valori a căror mărime o putem determina doar în timpul execuției.

Să explorăm în detaliu un tip dinamic dimensionat numit `str`, pe care l-am utilizat constant în această carte. Da, nu `&str`, ci `str` în sine este un DST. Nu putem determina lungimea string-ului decât în timpul execuției, ceea ce înseamnă că nu putem crea o variabilă de tip `str`, nici nu putem accepta un argument de acest tip. Să explorăm următorul exemplu de cod, care nu va compila:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust necesită să știe de câtă memorie are nevoie fiecare valoare a unui tip specific, și fiecare valoare a acelui tip trebuie să folosească aceeași cantitate de memorie. Dacă Rust ar permite acest cod să fie scris, cele două valori `str` ar trebui să ocupe același spațiu. Dar ele au lungimi diferite: `s1` necesită 12 octeți de stocare, `s2` - 15. Acest motiv face imposibilă crearea unei variabile de tip dinamic dimensionat.

Deci, ce putem face? Deja știi soluția: tipizăm `s1` și `s2` ca `&str` în loc de `str`. Așa cum am discutat în secțiunea [“Secțiuni de string”][string-slices]<!-- ignore --> din Capitolul 4, o structură de tip secțiune stochează doar poziția de start și lungimea secțiunii. De aceea, în timp ce un `&T` este o valoare care menține adresa de memorie unde `T` se află, un `&str` este format din *două* valori: adresa 'str'-ului și lungimea acestuia. Astfel, noi putem cunoaște mărimea unei valori `&str` în momentul compilării: este de două ori mărimea unui `usize`. Asta înseamnă că știm întotdeauna mărimea unui `&str`, indiferent cât de lung este string-ul referențiat. În mod obișnuit, în Rust, DST-urile se folosesc astfel: conțin o porțiune suplimentară de metadate care stochează mărimea informației dinamice. Regula fundamentală a tipurilor dinamic dimensionate este aceea că valorile lor trebuie să fie întotdeauna plasate în spatele unui tip de pointer.

Putem să combinăm `str` cu diverse tipuri de pointeri, cum ar fi `Box<str>` sau `Rc<str>`. De fapt, te-ai întâlnit deja cu acest concept, dar în cazul unui alt tip dinamic dimensionat: trăsăturile. Fiecare trăsătură este un DST la care ne putem referi folosind numele trăsăturii. În Capitolul 17, în secțiunea [“Utilizând obiecte trăsătură care termit valori de diferite tipuri”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore -->, am menționat că pentru a folosi trăsăturile ca obiecte trăsătură, trebuie să le punem în spatele unui pointer, cum ar fi `&dyn Trait` sau `Box<dyn Trait>`; varianta `Rc<dyn Trait>` este, de asemenea, validă.

Pentru a lucra cu DST-uri, Rust folosește trăsătura `Sized`, care ne spune dacă mărimea unui tip este cunoscută la momentul compilării. Această trăsătură este automat implementată pentru toate elementele ale căror dimensiuni pot fi determinate la compilare. Mai mult, Rust adaugă implicit o constrângere `Sized` la orice funcție generică. Astfel, definiția unei funcții generice precum:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

este tratată de parcă am fi scris de fapt așa:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

Implicit, funcțiile generice vor funcționa doar cu tipuri a căror mărime este cunoscută la momentul compilării. Cu toate acestea, putem folosi următoarea sintaxă specială pentru a slăbi această restricție:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

O constrângere pe `?Sized` înseamnă că "T poate sau nu poate fi `Sized`", și această notație anulează implicitul conform căruia tipurile generice trebuie să aibă o dimensiune cunoscută la momentul compilării. Sintaxa `?Trait` cu acest înțeles este disponibilă numai pentru `Sized`, nu și pentru alte trăsături.

Notăm, de asemenea, că am schimbat tipul parametrului `t` din `T` în `&T`, deoarece tipul poate să nu fie `Sized`, și astfel, trebuie să lucrăm cu el prin intermediul unui anumit tip de pointer. În acest caz, am optat pentru o referință.

În următoare secțiune, vom aborda subiectul funcțiilor și închiderilor!

[encapsulation-that-hides-implementation-details]:
ch17-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-operator]:
ch06-02-match.html#the-match-control-flow-operator
[using-trait-objects-that-allow-for-values-of-different-types]:
ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[using-the-newtype-pattern]: ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
