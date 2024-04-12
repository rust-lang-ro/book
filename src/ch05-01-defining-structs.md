## Definirea și crearea instanțelor de structuri

Structurile prezintă anumite asemănări cu tuplele, pe care le-am discutat în secțiunea [“Tipul tuplă”][tuples]<!-- ignore -->. Ambele pot deține mai multe valori direct correlate. Similar tuplelor, elementele unei structuri pot avea tipuri diferite. Cu toate acestea, în cazul unei structuri vei atribui un nume fiecărui segment de date, astfel încât să fie evident ce semnifică aceste valori. Prin adăugarea acestor denumiri, structurile devin mai flexibile decât tuplele: nu va trebui să te bazezi pe ordinea datelor pentru a specifica sau accesa valorile unei instanțe.

Pentru a defini o structură, folosim cuvântul cheie `struct` și atribuim un nume întregii structuri. Numele unei structuri ar trebui să descrie întreaga semnificație a fragmentelor de date grupate împreună. Apoi, în interiorul acoladelor, definim numele și tipurile fragmentelor de date, pe care le numim *câmpuri* (fields). De exemplu, în Listarea 5-1 prezentăm o structură care stochează informații legate de un cont de utilizator.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

<span class="caption">Listarea 5-1: Definirea structurii `User`</span>

Odată ce am definit o structură, pentru a o putea utiliza, trebuie să creăm o *instanță* a acesteia. Creăm o instanță stabilind valorile specifice pentru fiecare dintre câmpurile structurii. Acest lucru se realizează prin menționarea numelui structurii urmat de paranteze acolade, care includ perechi de tip *cheie: valoare*. 'Cheile' sunt denumirile câmpurilor, iar 'valorile' reprezintă informațiile pe care intenționăm să le stocăm în aceste câmpuri. Ordinea în care specificăm câmpurile nu trebuie să respecte neapărat ordinea în care acestea au fost declarate în structură. Cu alte cuvinte, putem privi definiția structurii ca pe un șablon general pentru tipul de date, iar instanțele completează acest șablon cu date specifice, formând astfel valori ale acelui tip. De exemplu, putem declara un utilizator specific conform exemplului din Listarea 5-2.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

<span class="caption">Listing 5-2: Creating an instance of the `User`
struct</span>

Pentru a extrage o anumită valoare dintr-o structură, apelăm la notația cu punct. De pildă, dacă dorim să accesăm adresa de email a acestui utilizator, utilizăm expresia `user1.email`. În cazul în care instanța noastră este mutabilă, avem posibilitatea de a modifica o valoare folosind aceeași notație cu punct, dar realizând o atribuire într-un câmp specific. În Listarea 5-3 este prezentată modalitatea de schimbare a valorii în câmpul `email` al unei instanțe mutabile de tip `User`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

<span class="caption">Listing 5-3: Changing the value in the `email` field of a
`User` instance</span>

<span class="caption">Listarea 5-3: Schimbarea valorii în câmpul `email` pentru o instanţă `User`</span>

Este important de notat, că toată instanța trebuie să fie mutabilă; Rust nu ne oferă posibilitatea de a defini doar anumite câmpuri ca fiind mutabile. Similar cu alte expresii, putem crea o instanță nouă a structurii în calitate de ultimă expresie în corpul funcției, astfel încât noua instanță să fie returnată în mod implicit.

Listarea 5-4 prezintă o funcție `build_user`, care returnează o instanță `User`, în funcție de email-ul și numele de utilizator specificat. Câmpul `active` ia valoarea `true`, iar `sign_in_count` primește valoarea `1`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

<span class="caption">Listarea 5-4: O funcție `build_user` care preia un email
și un nume de utilizator, returnând o instanță `User`</span>

Este firesc să folosim aceleași denumiri pentru parametrii funcției precum cele ale câmpurilor din structură. Cu toate acestea, repetarea numelor câmpurilor `email` și `username` și a variabilelor poate deveni monotonă. Dacă structura ar conține mai multe câmpuri, repetarea fiecărui nume s-ar transforma într-o sarcină mai mult decât fastidioasă. Din fericire, există o prescurtare foarte convenabilă!

<!-- Old heading. Do not remove or links may break. -->
<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Aplicarea sintaxei de inițializare abreviată a câmpurilor

Dând luare de seama că în Listarea 5-4 numele parametrilor se potrivesc perfect cu denumirile câmpurilor din structură, avem posibilitatea de a utiliza sintaxa de inițializare abreviată a câmpurilor (field init shorthand). Acest lucru ne permite să rescriem funcția `build_user` astfel încât aceasta să funcționeze identic, însă eliminând repetarea neceasră a numelor `username` și `email`. Acest proces este ilustrat în Listarea 5-5.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

<span class="caption">Listarea 5-5: O funcție `build_user` care utilizează sintaxa abreviată de inițializare a câmpurilor, având în vedere faptul că parametrii `username` și `email` corespund cu numele câmpurilor din structură</span>

În acest caz, noi generăm o nouă instanță a structurii `User`, ce include un câmp denumit `email`. Intenția noastră este de a atribui valoarea câmpului `email` cu valoarea corespunzătoare parametrului `email` din funcția `build_user`. Grație faptului că numele câmpului `email` și cel al parametrului `email` sunt identice, este suficient să scriem o singură dată `email`, în loc de `email: email`.

### Crearea de instanțe din alte instanțe utilizând sintaxa de actualizare a structurii

Adeseori este util să generezi o nouă instanță a unei structuri care păstrează majoritatea valorilor provenite din altă instanță, dar modifică câteva dintre ele. Aceasta se poate realiza utilizând *sintaxa de actualizare a structurii*.

Pentru început, în Listarea 5-6, ilustrăm cum să creăm o nouă instanță `User` în `user2`, fără a folosi sintaxa de actualizare. Atribuim o nouă valoare pentru `email`, dar pentru restul valorilor, le păstrăm pe cele din `user1`, pe care l-am creat în Listarea 5-2.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

<span class="caption">Listarea 5-6: Crearea unei noi instanțe de tip 'User', folosind o valoare din 'user1'</span>

Utilizând sintaxa de actualizare a structurii, putem atinge același rezultat cu un cod mult mai concis, așa cum se arată în Listarea 5-7. Sintaxa `..` indică faptul că toate celelalte câmpuri care nu au fost explicit stabilite trebuie să aibă valorile identice cu cele din instanța sursă.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

<span class="caption">Listing 5-7: Using struct update syntax to set a new
`email` value for a `User` instance but to use the rest of the values from
`user1`</span>

Codul prezentat în Listarea 5-7 generează de asemenea o instanță în `user2`, care are o valoare distinctă pentru `email`, dar păstrează aceleași valori pentru câmpurile `username`, `active` și `sign_in_count` ca în `user1`. Elementul `..user1` trebuie poziționat la final pentru a indica faptul că orice alt câmp rămas ar trebui să își preia valorile de la câmpurile corespondente din `user1`. În același timp, ne este permis să stabilim valorile pentru oricâte câmpuri dorim, fără a fi impusă vreo ordine, indiferent de succesiunea câmpurilor în cadrul definiției structurii.

Este important de remarcat faptul că sintaxa de actualizare a structurilor utilizează `=` la fel ca într-o atribuire; acest lucru se produce deoarece datele sunt permutate, așa cum am observat în secțiunea [“Variabile și interacționarea cu date folosind permutarea”][move]<!-- ignore -->. În acest exemplu, după ce am creat `user2`, nu mai putem folosi în totalitate `user1` deoarece string-ul din câmpul `username` al `user1` a fost permutat în `user2`. Dacă am fi atribuit `user2` noi valori de string atât pentru `email` cât și pentru `username`, utilizând astfel doar valorile `active` și `sign_in_count` de la `user1`, atunci `user1` ar fi rămas valid după crearea `user2`. Atât `active` cât și `sign_in_count` sunt tipuri care implementează trăsătura `Copy`, astfel comportamentul de care am discutat în secțiunea [“Date doar pe stivă: trăsătura Copy”][copy]<!-- ignore --> va fi aplicabil.

### Utilizarea structurilor tuplă fără câmpuri denumite pentru a genera tipuri diferite

Rust suportă de asemenea structuri care se aseamănă cu tuplele, denumite *structuri tuplă*. Acestea îmbogățesc semnificația pe care o conferă numele structurii, deși nu dispun de nume asociate câmpurilor lor; în loc, acestea sunt definite doar prin tipurile câmpurilor. Structurile tuplă sunt utile atunci când dorești să atribui un nume întregii tuple, configurându-l ca un tip distinct față de alte tuple, precum și atunci când ar fi redundant să denumești fiecare câmp, așa cum s-ar întâmpla într-o structură obișnuită.

Pentru a defini o structură tuplă, începeți cu cuvântul-cheie `struct`, urmat de numele structurii și de tipurile componente ale tuplei. De exemplu, în continuare definim și folosim două structuri tuplă, denumite `Color` și `Point`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

Observă că `black` și `origin` sunt de tipuri diferite, deoarece reprezintă instanțe ale unor structuri tuplă distincte. Fiecare structură pe care o definești în program constituie un tip inedit, chiar dacă toate câmpurile acelei structuri au același tip. De exemplu, o funcție care necesită un argument de tip `Color` nu va putea primi un `Point` ca și parametru, chiar dacă ambele tipuri sunt formate din trei valori `i32`. La fel ca și în cazul tuplă, la structurile tuplă avem posibilitatea de a le descompune în elementele individuale și putem accesa o anumită valoare folosind un `.` urmat de indexul respectiv.

### Structuri asemănătoare cu unit, fără niciun câmp

Poate vei fi surprins, dar este posibil să definești și structuri care nu includ niciun câmp! Acestea se numesc *structuri asemănătoare cu unit* (unit-like structs) deoarece comportamentul lor este similar cu cel al `()`, tipul unit pe care l-am menționat în secțiunea dedicată [`Tipul tuplă`][tuples]<!-- ignore -->. Asemenea structuri pot dovedi a fi utile când ai nevoie să implementezi o trăsătură pe un anumit tip, dar concomitent nu ai nicio dată pe care dorești să o stochezi direct în tipul respectiv. Ne vom dedica mai mult acestui concept de trăsătură în Capitolul 10. Ca exemplu, iată cum poți declara și instanția o structură asemănătoare cu unit, pe care o vom numi `AlwaysEqual`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

Pentru a defini structura `AlwaysEqual`, utilizăm cuvântul cheie `struct`, urmat de numele ales de noi și semnul punct și virgulă. Nu este necesar să folosim paranteze sau acolade! Ulterior, putem obține o instanță a structurii `AlwaysEqual` în variabila `subject` prin intermediu unui proces similar: utilizând numele definit de noi, în absența oricăror paranteze sau acolade. Concepe următorul scenariu: la un moment dat, vom implementa o funcționalitate pentru acest tip, astfel încât orice instanță a `AlwaysEqual` să fie considerată egală cu orice instanță a altui tip, aceasta urmând să fie folosită probabil pentru a obține un rezultat standard în cazul unor teste. Pentru a implementa această funcționalitate, nu ne va fi nevoie de nicio informație suplimentară! În Capitolul 10 vei afla cum se definesc trăsăturile și cum acestea pot fi implementate pe orice tip, inclusiv pe structurile asemănătoare cu unit.

> ### Proprietatea datelor din cadrul unei structuri
>
> În definiția structurii `User` prezentată în Listarea 5-1, am optat pentru
> folosirea tipului `String`, care reprezintă un string deținut, în locul
> tipului de secțiune de string `&str`. Aceasta nu este o decizie la
> întâmplare, obiectivul nostru fiind ca fiecare instanță a respectivei
> structuri să dețină întreaga sa colecție de date, dată ce vor rămâne valabile
> atât timp cât structura în sine este în vigoare.
>
> De asemenea, structurile pot stoca referințe către date deținute de alte
> elemente, dar pentru asta este necesară utilizarea *duratelor de viață*,
> o caracteristică specifică limbajului Rust pe care o vom detalia în Capitolul
> 10. Duratele de viață garantează că datele referențiate de o structură rămân
> valabile pe întreaga durată de existență a structurii. Să presupunem că
> dorești să stochezi o referință într-o structură fără a indica duratele de
> viață, exact ca în exemplul următor; vei constata că acest lucru nu este
> posibil:
>
> <span class="filename">Numele fișierului: src/main.rs</span>
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> Compilatorul va semnala eroare, solicitând specificatori pentru durata de
> viață:
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` due to 2 previous errors
> ```
>
> În Capitolul 10, vom discuta cum putem rezolva aceste erori pentru a stoca
> referințe în structuri. Până atunci însă, vom remedia astfel de erori prin
> folosirea tipurilor ce se află sub posesiunea noastră, precum `String`, în
> locul referințelor de tip `&str`.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
