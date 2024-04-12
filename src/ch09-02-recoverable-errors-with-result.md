## Gestionarea erorilor recuperabile cu `Result`

Nu toate erorile sunt atât de grave încât să necesite oprirea completă a programului. Când o funcție eșuează, uneori motivul poate fi ușor de interpretat și de abordat. Spre exemplu, dacă încerci să deschizi un fișier iar operațiunea nu reușește pentru că fișierul lipsește, ai putea să optezi pentru crearea fișierului în loc să oprești programul.

Reamintește-ți din Capitolul 2, secțiunea [„Gestionarea potențialelor eșecuri cu `Result`”][handle_failure] <!-- ignore -->. Am discutat atunci că enum-ul `Result` este definit cu două variante: `Ok` și `Err`.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` și `E` sunt parametri de tip generic. Vom discuta despre generice mai pe larg în Capitolul 10. Important de știut acum este că `T` reprezintă tipul valorii returnate în caz de succes de varianta `Ok`, iar `E` tipul erorii returnate în caz de eșec de varianta `Err`. Cu acești parametri generici, tipul `Result` poate fi utilizat în multe contexte diferite, unde valorile succesului și ale erorii pot varia.

Imaginează-ți că apelăm o funcție care returnează o valoare `Result` pentru că funcția ar putea să nu reușească. În Listarea 9-3, încercăm să deschidem un fișier.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

<span class="caption">Listarea 9-3: Deschiderea unui fișier</span>

Tipul returnat de `File::open` este `Result<T, E>`. În acest context, `T` este `std::fs::File`, adică un descriptor al fișierului, iar `E` este `std::io::Error`. Acest lucru înseamnă că apelul la `File::open` poate reuși sau eșua — fișierul să nu existe sau să nu avem permisiunea necesară. Funcția `File::open` trebuie astfel să ne poată informa despre reușită sau eșec și să ne furnizeze descriptorul fișierului sau date despre eroare. `Result` este exact mecanismul care transmite aceste informații.

Dacă `File::open` reușește, variabila `greeting_file_result` va conține o instanță `Ok` cu descriptorul fișierului. Dacă eșuează, va conține `Err` cu informații suplimentare privind eroarea survenită.

Trebuie să extindem codul din Listarea 9-3 pentru a gestiona diferite rezultate ale funcției `File::open`. Listarea 9-4 prezintă cum putem folosi expresia `match` pentru aceasta, discutată în Capitolul 6.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

<span class="caption">Listarea 9-4: Utilizarea `match` pentru gestionarea variantelor `Result`</span>

La fel ca `Option`, și `Result` și variantele sale sunt importate automat, deci nu avem nevoie să precizăm `Result::` înainte de `Ok` și `Err` în ramurile `match`.

Când rezultatul este `Ok`, codul extrage valoarea file din varianta `Ok` și o atribuie variabilei `greeting_file`. Ulterior, descriptorul fișierului poate fi folosit pentru citire sau scriere.

Ramura pentru `Err` din `match` gestionează situația eșecului de la `File::open`. În acest exemplu, optăm pentru panică, prin `panic!`. Dacă nu există un fișier denumit *hello.txt* în directoriul nostru curent atunci când rulăm codul, `panic!` ne va arăta următoarea ieșire:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Ca de obicei, această ieșire ne detaliază cu precizie problema întâmpinată.

### Diferențiind reacția la erori

Codul prezentat în Listarea 9-4 va genera o panică (`panic!`) indiferent de cauza eșecului funcției `File::open`. Totuși, noi intenționăm să abordăm diferit motivele specifice ale eșecului: dacă `File::open` nu reușește datorită inexistenței fișierului, intenționăm să creăm fișierul și să returnăm un descriptor către acesta. În schimb, dacă `File::open` eșuează din alte motive - de exemplu, lipsa permisiunilor de acces - dorim să menținem reacția inițială de panică, așa cum este ilustrat în Listarea 9-4. Pentru a gestiona acest comportament, includem o expresie `match` suplimentară, ilustrată în Listarea 9-5.

<span class="filename">Numele fișierului: src/main.rs</span>

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

<span class="caption">Listarea 9-5: Abordări diferențiate ale gestionării erorilor</span>

Tipul valorii returnate de `File::open` când întâlnește o eroare (`Err`) este `io::Error`, o structură definită în biblioteca standard. Această structură include metoda `kind`, prin care putem obține o valoare de tip `io::ErrorKind`. Enum-ul `io::ErrorKind`, de asemenea furnizat de biblioteca standard, categorizează potențialele erori ce pot apărea în timpul unei operațiuni `io`. Pentru cazul nostru, ne folosim de varianta `ErrorKind::NotFound`, care semnalizează că fișierul pe care dorim să-l deschidem nu există încă. Acest lucru ne conduce la aplicarea unui `match` pe variabila `greeting_file_result`, dar în interiorul acestuia aplicăm și un `match` pe rezultatul apelării `error.kind()`.

Ne interesează să verificăm, în cadrul `match`-ului intern, dacă valoarea întoarsă de `error.kind()` corespunde cu varianta `NotFound` a enum-ului `ErrorKind`. Dacă este așa, înaintăm cu încercarea de creare a fișierului folosind `File::create`. Însă, cum și această operațiune poate să eșueze, introducem un al doilea braț în expresia de `match` din interior. În situația în care crearea fișierului nu este posibilă, se va afișa un mesaj de eroare diferit. Cel de-al doilea braț al `match`-ului exterior rămâne neschimbat, astfel programul va genera o panică pentru orice alt tip de eroare, în afara erorii generată de absența fișierului.

> ### Metode alternative la utilizarea `match` cu `Result<T, E>`
>
> Expresia `match` este extrem de utilă, însă poate deveni încărcată în
> anumite contexte. În Capitolul 13, veți descoperi closures, care facilitează
> lucrul cu diverse metode definite pentru `Result<T, E>`. Aceste metode oferă
> abordări mai concise comparativ cu `match` pentru gestionarea valorilor
> `Result<T, E>` în cod.
>
> De pildă, vă prezentăm o altă cale de a implementa logica din Listarea 9-5,
> utilizând de această dată closures și metoda `unwrap_or_else`:
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {:?}", error);
>             })
>         } else {
>             panic!("Problem opening the file: {:?}", error);
>         }
>     });
> }
> ```
>
> Deși acest fragment de cod produce același efect ca Listarea 9-5, el nu
> conține niciun `match`, ceea ce îl face mai curat și mai lizibil. Vă sugerăm
> să reveniți la acest exemplu după parcurgerea Capitolului 13 și să
> consultați documentația metodei `unwrap_or_else` din biblioteca standard a
> Rust. Vei descoperi că există multe alte metode care pot simplifica expresii
> complexe și îmbinate de `match`, mai ales atunci când tratați erori în codul
> dvs.

### Scurtături pentru panică la eroare: `unwrap` și `expect`

Deși este destul de eficientă, utilizarea expresiei `match` poate deveni oneroasă și nu întotdeauna exprimă clar intenția programatorului. Tipul `Result<T, E>` dispune de numeroase metode auxiliare destinate efectuării de operații specifice. Una dintre aceste metode este `unwrap`, care funcționează similar cu expresia `match` pe care am detaliat-o în Listarea 9-4. Dacă `Result` este varianta `Ok`, `unwrap` extrage și returnează valoarea conținută. În schimb, dacă `Result` este varianta `Err`, `unwrap` va apela macro-ul `panic!`. Iată `unwrap` în aplicare:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

Dacă executăm acest cod fără a avea fișierul *hello.txt*, vom întâlni un mesaj de eroare generat de apelul `panic!` pe care îl face metoda `unwrap`:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Os {
code: 2, kind: NotFound, message: "No such file or directory" }',
src/main.rs:4:49
```

Metoda `expect` ne oferă posibilitatea de a alege mesajul de eroare pentru `panic!`. Prin folosirea lui `expect` în locul lui `unwrap` și prin oferirea de mesaje explicite de eroare, îți poți clarifica intenția și facilita identificarea sursei unei erori fatale. Sintaxa metodei `expect` este următoarea:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

`Expect` se utilizează la fel cum procedăm cu `unwrap`: intenționăm să extragem descriptorul fișierului sau să declanșăm macro-ul `panic!`. Cu toate acestea, mesajul de eroare dat de `expect` când apelează `panic!` va fi textul specific pe care îl pasăm către `expect`, spre deosebire de mesajul prestabilit al lui `unwrap`. Iată cum arată în practică:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at 'hello.txt should be included in this project: Os {
code: 2, kind: NotFound, message: "No such file or directory" }',
src/main.rs:5:10
```

În codul destinat producției, Rustaceanii aleg de obicei `expect` în loc de `unwrap`, furnizând mai multe detalii legate de motivul pentru care operațiunea ar trebui să reușească întotdeauna. În acest fel, dacă ipotezele tale se dovedesc a fi incorecte, vei dispune de mai multe informații utile pentru depanare.

### Propagarea erorilor

Când o funcție se confruntă cu posibilitatea unui eșec în timpul executării, poți alege să nu soluționezi eroarea în interiorul acelei funcții. În schimb, poți redirecționa eroarea către codul care a inițiat apelul funcției, permițându-i să decidă cum să procedeze. Această abordare se numește *propagarea* erorii și conferă un grad mai mare de control codului apelant, care ar putea deține informații suplimentare sau o logică specifică pentru tratamentul erorii, comparativ cu ce este disponibil în contextul funcției tale.

De exemplu, în Listarea 9-6 este prezentată o funcție ce încearcă să citească numele de utilizator dintr-un fișier. Dacă fișierul nu există sau nu poate fi accesat, această funcție va returna erorile întâmpinate direct codului ce a solicitat funcția.

<span class="filename">Numele fișierului: src/main.rs</span>

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

<span class="caption">Listarea 9-6: Funcție care gestionează erorile prin `match`</span>

Funcția prezentată poate fi scrisă într-o formă mult mai concisă. Totuși, pentru a înțelege mai bine gestionarea erorilor, vom începe cu o abordare manuală, pas cu pas. La final, vom prezenta și versiunea simplificată. Mai întâi, să ne concentrăm asupra tipului de retur: `Result<String, io::Error>`. Acesta indică faptul că funcția returnează `Result<T, E>`, unde parametrul generic `T` este specificat ca `String`, iar `E` ca `io::Error`.

Dacă funcția se execută corect, rezultatul va fi un `Ok` conținând un `String`—numele de utilizator citit din fișier. În cazul apariției unei erori, se va returna `Err` cu un `io::Error`, detaliind problema survenită. Alegem să folosim `io::Error` ca tip de eroare pentru că acesta este returnat atunci când operațiunile `File::open` sau `read_to_string` eșuează, acestea fiind funcțiile utilizate în cadrul funcției noastre.

Începem corpul funcției apelând funcția `File::open`. Gestionăm rezultatul `Result` printr-un `match` asemănător celui din Listarea 9-4. Dacă `File::open` reușește, variabila de tip șablon `file`, care acum stochează descriptorul fișierului, este asignată variabilei mutabile `username_file`, iar execuția funcției continuă. În caz de eroare `Err`, în loc să utilizăm macro-ul `panic!`, preferăm să ieșim din funcție folosind cuvântul cheie `return`, returnând direct eroarea primită de la `File::open`, acum stocată în variabila șablon `e`.

Dacă avem un descriptor de fișier valid în `username_file`, funcția trece la crearea unui nou `String` în variabila `username`. Apoi invocăm metoda `read_to_string` pe descriptorul din `username_file` pentru a citi conținutul fișierului în `username`. Metoda `read_to_string` returnează și ea un `Result`, deoarece poate eșua, chiar și când `File::open` a avut succes. Prin urmare, aplicăm un nou `match` pentru acest `Result`. Dacă `read_to_string` se finalizează cu succes, funcția noastră este și ea un succes și returnăm numele de utilizator din fișier, acum aflat în `username`, încapsulat într-un `Ok`. Dacă `read_to_string` dă greș, tratăm eroarea în mod similar cu cel din `match`-ul precedent, dar fără a mai folosi explicit cuvântul `return`, fiindcă aceasta este ultima expresie din funcție, iar valorile erorilor sunt întoarse implicit.

Codul care cheamă funcția noastră va trebui să gestioneze rezultatul: fie o valoare `Ok` ce conține un nume de utilizator, fie o valoare `Err` ce include o eroare `io::Error`. Depinde de codul apelant să decidă cum va proceda cu aceste rezultate. În cazul unei valori `Err`, codul respectiv poate alege să genereze panică folosind `panic!` și astfel să oprească execuția programului, să utilizeze un nume de utilizator prestabilit sau să caute numele de utilizator prin alte metode, care nu implică accesul la un fișier. Noi nu cunoaștem intențiile specifice ale codului apelant, prin urmare, transmitem informațiile despre succes sau eroare mai departe pentru ca acesta să le gestioneze în cel mai potrivit mod.

Această metodă de a transmite erorile este atât de răspândită în Rust, încât limbajul include operatorul `?` pentru a simplifica acest proces.

#### Propagarea erorilor cu operatorul `?`

Listarea 9-7 ne prezintă cum să folosim funcția `read_username_from_file` pentru a obține aceleași rezultate ca și în Listarea 9-6, dar de această dată folosind operatorul `?`.

<span class="filename">Numele fișierului: src/main.rs</span>

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

<span class="caption">Listarea 9-7: O funcție care returnează erori către codul apelant folosind operatorul `?`</span>

Atunci când se pune operatorul `?` după o valoare de tip `Result`, acesta funcționează similar cu expresiile `match` pe care le-am utilizat anterior pentru a gestiona valorile `Result` în Listarea 9-6. Dacă rezultatul de tip `Result` este `Ok`, conținutul lui `Ok` este returnat și execuția programului continuă. Dacă rezultatul este `Err`, atunci `Err` este returnat de întreaga funcție, ca și cum am folosi cuvântul cheie `return`, astfel propagând eroarea către codul care a invocat funcția.

Însă, există o diferență între expresiile `match` din Listarea 9-6 și operatorul `?`: erorile asupra cărora este aplicat operatorul `?` sunt trecute prin funcția `from`, definită de trăsătura `From` din biblioteca standard. Funcția `from` transformă valorile dintr-un tip în altul. Când operatorul `?` invocă `from`, tipul erorii revine convertit la tipul de eroare specificat în semnătura funcției curente. Acest aspect se dovedește a fi util atunci când o funcție trebuie să returneze un singur tip de eroare pentru a reprezenta diferitele cauze care pot conduce la eșecul acesteia. 

De exemplu, putem modifica funcția `read_username_from_file` prezentată în Listarea 9-7 astfel încât să returneze un tip propriu de eroare, denumit `OurError`, pe care îl definim noi. De asemenea, dacă implementăm `impl From<io::Error> pentru OurError` pentru a crea o instanță `OurError` dintr-un `io::Error`, apelurile operatorului `?` din funcția `read_username_from_file` vor utiliza automat `from` pentru a converti erorile, fără să mai adăugăm cod suplimentar.

În cazul prezentat în Listarea 9-7, semnul `?` de la sfârșitul apelului `File::open` va extrage valoarea dintr-un rezultat `Ok` și o va asigna variabilei `username_file`. Dacă întâmpinăm o eroare, operatorul `?` va opri execuția funcției imediat și va transmite valoarea `Err` codului care a făcut apelul. Același principiu se aplică și pentru `?` de la sfârșitul apelului `read_to_string`.

Operatorul `?` reduce semnificativ redundanța codului și facilitează simplificarea implementării funcției. Mai mult, prin înlănțuirea directă a apelurilor de metode după `?`, putem condensa codul și mai mult, aspect ilustrat în Listarea 9-8.

<span class="filename">Numele fișierului: src/main.rs</span>

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

<span class="caption">Listarea 9-8: Lănțuirea metodelor după operatorul `?`</span>

Am inițializat noul string `username` la începutul funcției, ca și înainte. În loc să declarăm o variabilă `username_file`, acum apelăm metoda `read_to_string` imediat după `File::open("hello.txt")?`. Continuăm să folosim `?` la sfârșitul lui `read_to_string` și returnăm un `Ok` care conține `username` dacă atât `File::open`, cât și `read_to_string` reușesc, evitând returnarea de erori. Practic, obținem aceeași funcționalitate ca în Listarea 9-6 și Listarea 9-7, dar printr-o scriere mai compactă și ergonomică.

Listarea 9-9 va prezenta cum să simplificăm și mai mult codul, utilizând `fs::read_to_string`.

<span class="filename">Filename: src/main.rs</span>

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

<span class="caption">Listarea 9-9: Folosirea funcției `fs::read_to_string` pentru a evita deschiderea și citirea explicită a fișierului</span>

Citirea conținutului unui fișier într-un string este o operaţie frecventă, iar biblioteca standard facilitează această operație prin funcția practică `fs::read_to_string`. Această funcție deschide fișierul, inițializează un nou `String`, citește conținutul fișierului, îl stochează în acel `String` și apoi îl returnează. Desigur, prin utilizarea `fs::read_to_string` nu putem ilustra în detaliu gestionarea erorilor, motiv pentru care am ales inițial metoda mai elaborată.

#### Unde poate fi folosit operatorul `?`

Operatorul `?` poate fi utilizat numai în funcții a căror valoare de retur este compatibilă cu tipul de valoare asupra căruia se aplică `?`. Acest lucru se datorează faptului că operatorul `?` este conceput pentru a efectua un retur prematur din funcție, similar cu expresia `match` pe care am descris-o în Listarea 9-6. În cazul Listării 9-6, `match` opera cu o valoare de tip `Result`, iar ramura de retur prematur returna o valoare `Err(e)`. Funcția trebuie să aibă ca tip de retur un `Result` pentru a fi compatibil cu acțiunea de `return`.

În Listarea 9-10 vom vedea ce eroare apare atunci când utilizăm operatorul `?` într-o funcție `main` care are un tip de retur incompatibil cu tipul valorii pentru care aplicăm `?`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

<span class="caption">Listarea 9-10: Tentativa de a folosi `?` în funcția `main` care returnează `()` nu va funcționa</span>

Acest cod încearcă să deschidă un fișier, operațiune care poate să eșueze. Operatorul `?` este aplicat valorii `Result` returnate de `File::open`, însă funcția `main` are definit ca tip de retur `()`, nu `Result`. Când încercăm să compilăm acest cod, ne întâmpină următorul mesaj de eroare:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Această eroare semnalează faptul că putem utiliza operatorul `?` exclusiv în cadrul funcțiilor care returnează `Result`, `Option`, sau alte tipuri ce implementează `FromResidual`.

Pentru a corecta eroarea, ai două opțiuni. Prima este să modifici tipul de retur al funcției tale astfel încât să corespundă cu tipul valorii peste care aplici operatorul `?`, cu condiția să nu existe restricții care te împiedică. Alternativa este folosirea unei structuri `match` sau a metodelor disponibile pentru `Result<T, E>` pentru a gestiona rezultatul `Result<T, E>` în modul cel mai potrivit pentru contextul tău.

Mesajul de eroare a subliniat, de asemenea, că operatorul `?` poate fi aplicat pe valori de tip `Option<T>`. Asemenea utilizării `?` pe `Result`, acesta poate fi folosit pe `Option` numai într-o funcție care returnează un `Option`. Atunci când `?` este utilizat pe un `Option<T>`, comportamentul său este asemănător: dacă valoarea este `None`, atunci `None` se returnează imediat, întrerupând execuția funcției. Dacă valoarea este de tip `Some`, conținutul lui `Some` devine valoarea expresiei, iar execuția funcției continuă. Următoarea listare, 9-11, conține un exemplu de funcție care identifică ultimul caracter din prima linie a unui text furnizat:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

<span class="caption">Listarea 9-11: Utilizarea operatorului `?` pentru o valoare `Option<T>`</span>

Funcția aceasta returnează `Option<char>` deoarece există posibilitatea ca un caracter să fie prezent, dar de asemenea e posibil ca acesta să lipsească. Codul ia secțiunea de string `text` ca argument și îi aplică metoda `lines`, care oferă un iterator pentru liniile din string. Pentru a examina prima linie, folosim metoda `next` pe iterator, pentru a extrage prima valoare. Dacă `text` este un string gol, `next` va returna `None`; în acest caz, operatorul `?` intervine pentru a opri execuția și a returna `None` din `last_char_of_first_line`. Dacă în schimb `text` nu este gol, `next` va returna un `Some` ce conține slice-ul primei linii din `text`.

Operatorul `?` extrage acea secțiune, permițându-ne apoi să apelăm metoda `chars` pentru a obține un iterator al caracterelor sale. Ne interesează ultimul caracter din prima linie, deci apelăm `last` pentru a obține ultimul element al iteratorului, care este tot un `Option`. Este posibil ca prima linie să fie și ea un string gol – de exemplu, dacă `text` începe cu o linie goală, urmată de alte linii cu caractere, cum ar fi `"\nhi"`. În cazul în care există un caracter final în prima linie, acesta va fi returnat într-o valoare `Some`. Folosind operatorul `?`, putem exprima această verificare concis, permițând implementarea funcției într-o singură linie. Altfel, fără operatorul `?` aplicabil pe `Option`, am fi nevoiți să reconstruim această logică prin mai multe apeluri de metode sau printr-o expresie `match`.

Trebuie să reții că operatorul `?` poate fi utilizat pentru a propaga erorile într-o funcție care returnează un `Result` atunci când lucrezi cu un `Result`, iar în cazul în care lucrezi cu un `Option`, poți utiliza operatorul `?` într-o funcție care returnează un `Option`. Însă nu este posibilă utilizarea mixtă a ambelor. Cu alte cuvinte, operatorul `?` nu realizează automat conversia între `Result` și `Option` sau invers; în acele situații, trebuie să apelezi metode specifice, cum ar fi `ok` pentru `Result` sau `ok_or` pentru `Option`, pentru a efectua conversia în mod explicit.

Până în prezent, toate funcțiile `main` pe care le-am utilizat returnau `()`. Merită să subliniem că funcția `main` este unică, fiind punctul de start și de terminare al programelor executabile. Există restricții specifice legate de tipul de retur pe care îl poate avea, astfel încât programul să funcționeze corespunzător.

Noutatea bună e că `main` poate de asemenea să returneze `Result<(), E>`. În Listarea 9-12, prezentăm codul din Listarea 9-10, dar cu tipul de retur al funcției `main` modificat în `Result<(), Box<dyn Error>>` și adăugăm la final valoarea de retur `Ok(())`. Cu aceste modificări, codul este gata de compilat:

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

<span class="caption">Listarea 9-12: Modificând funcția `main` pentru a returna `Result<(), E>`, permitem folosirea operatorului `?` pe valorile `Result`</span>

Tipul `Box<dyn Error>` este un *obiect trăsătură*, concept pe care îl vom aborda mai detaliat în secțiunea [„Utilizarea obiectelor trăsături care permit valori de diferite tipuri”][trait-objects]<!-- ignore --> din Capitolul 17. Până atunci, poți gândi la `Box<dyn Error>` ca o modalitate de a desemna „orice fel de eroare”. Utilizarea operatorului `?` pe o valoare de tip `Result` în funcția `main` este posibilă atunci când tipul erorii este `Box<dyn Error>`, pentru că aceasta acceptă returnarea anticipată a oricărei valori de eroare `Err`. Deși corpul funcției `main` va genera, în mod normal, doar erori de tip `std::io::Error`, prin definirea tipului de eroare ca fiind `Box<dyn Error>`, semnătura acestuia rămâne validă chiar și atunci când adăugăm mai mult cod care generează alte tipuri de erori în `main`.

Când funcția `main` returnează un `Result<(), E>`, aplicația va termina execuția cu valoarea `0` dacă `main` returnează `Ok(())` și va închide cu o valoare diferită de zero dacă `main` generează o eroare `Err`. Executabilele în limbajul C returnează valori întregi când se încheie execuția: programele care se termină corect returnează întregul `0`, în timp ce programele care se închid cu o eroare returnează un întreg diferit de `0`. Rust adoptă această convenție, returnând întregi de la executabile pentru compatibilitate.

Funcția `main` poate returna orice tip de date care implementează [trăsătura `std::process::Termination`][termination], care include funcția `report` rezultând într-un `ExitCode`. Consultă documentația bibliotecii standard Rust pentru mai multe detalii despre cum să implementezi trăsătura `Termination` pentru tipurile tale.

După ce am clarificat modul în care apelăm `panic!` sau returnăm `Result`, să discutăm cum alegem între aceste opțiuni în funcție de situație.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[termination]: ../std/process/trait.Termination.html
