## Macrocomenzi

Pe parcursul acestei cărți, am utilizat macrocomenzi precum `println!`, dar nu am examinat îndeaproape ce reprezintă o macrocomandă și cum funcționează. Termenul *macrocomandă* se referă la o clasă de caracteristici în Rust: macrocomenzi *declarative* cu `macro_rules!` și trei tipuri de macrocomenzi *procedurale*:

* Macrocomenzi `#[derive]` personalizate care specifică cod adăugat cu atributul `derive` utilizat la structuri și enumerări
* Macrocomenzi similare atributelor ce definesc atribute personalizate aplicabile pe orice element
* Macrocomenzi ce semănă cu funcțiile, care par a fi apeluri de funcții, dar operează pe token-uri specificate ca argumente

Vom discuta despre fiecare în parte, dar înainte de asta, să explorăm motivul pentru care avem nevoie de macrocomenzi, chiar dacă dispunem deja de funcții.

### Diferența dintre macrocomenzi și funcții

La baza lor, macrocomenzile sunt un mod de a scrie cod ce generează alt cod, proces cunoscut ca *metaprogramare*. În Anexa C, discutăm atributul `derive`, care produce implementări automatizate pentru diferite trăsături. De asemenea, am folosit macrocomenzile `println!` și `vec!` prin întreaga carte. Toate aceste macrocomenzi *se desfășoară* pentru a crea mai mult cod decât cel scris de tine manual.

Metaprogramarea este eficientă în reducerea cantității de cod pe care trebuie să o scrii și să o menții, un rol pe care îl joacă și funcțiile. Totuși, macrocomenzile au anumite capabilități suplimentare față de funcții.

Semnătura unei funcții trebuie să indice numărul și tipurile de parametrii pe care funcția îi acceptă. Macrocomenzile, în schimb, pot accepta un număr variabil de parametri: putem invoca `println!("hello")` cu un singur argument sau `println!("hello {}", name)` cu două argumente. Mai mult, macrocomenzile sunt desfășurate înaintea interpretării codului de către compilator, așa că o macrocomandă poate să implementeze o trăsătură pentru un anumit tip, spre deosebire de funcții, care sunt apelate în timpul execuției și nu pot implementa trăsături, ce trebuie definite în timpul compilării.

Un dezavantaj al utilizării macrocomenzilor în locul funcțiilor este că definițiile de macrocomenzi pot fi mai complexe decât cele ale funcțiilor, deoarece scrim cod Rust care generează cod Rust. Acest nivel suplimentar de abstractizare face ca definițiile de macrocomenzi să fie mai greu de citit, de înțeles și de întreținut decât cele ale funcțiilor.

O altă diferență semnificativă între macrocomenzi și funcții este că macrocomenzile trebuie definite sau aduse în domeniul de vizibilitate *înaintea* utilizării lor într-un fișier, spre deosebire de funcții, care pot fi definite oriunde și apelate în orice loc din cod.

### Macrocomenzi declarative cu `macro_rules!` pentru metaprogramare generală

Forma de macrocomandă cea mai des utilizată în Rust este *macrocomanda declarativă*. Acestea sunt cunoscute și ca „macrocomenzi prin exemplu”, „macrocomenzi `macro_rules!`”, sau simplu, „macrocomenzi”. La baza lor, macrocomenzile declarative ne permit să scrim ceva similar unei expresii Rust `match`. După cum am discutat în Capitolul 6, expresiile `match` sunt structuri de control care evaluează o expresie, compară valoarea rezultată cu pattern-uri, și execută codul asociat cu pattern-ul corespunzător. Macrocomenzile compară, de asemenea, o valoare cu pattern-uri ce sunt asociate cu anumite secțiuni de cod: în această situație, valoarea este codul sursă Rust literal transmis macrocomenzii; pattern-urile sunt comparate cu structura acelui cod sursă, iar codul asociat fiecărui pattern corespunzător înlocuiește codul inițial transmis macrocomenzii. Tot acest proces are loc în timpul compilării.

Pentru a defini o macrocomandă, folosim construcția `macro_rules!`. Să explorăm cum se utilizează `macro_rules!`, prin a examina cum este definită macrocomanda `vec!`. Capitolul 8 a discutat cum putem folosi macrocomanda `vec!` pentru a inițializa un vector nou cu valori specifice. Spre exemplu, macrocomanda următoare creează un vector nou ce conține trei numere întregi de tip `u32`:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Macrocomanda `vec!` poate fi folosită de asemenea pentru a crea un vector cu două numere întregi sau un vector cu cinci secțiuni de string. Nu am putea realiza aceeași funcționalitate folosind o funcție, deoarece nu am ști numărul sau tipurile valorilor dinainte.

Listarea 19-28 prezintă o variantă simplificată a definiției macrocomenzii `vec!`.
<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-28/src/lib.rs}}
```

<span class="caption">Listarea 19-28: O versiune simplificată a definiției macrocomenzii `vec!`</span>

Adnotarea `#[macro_export]` indică faptul că această macrocomandă trebuie să fie disponibilă de îndată ce crate-ul în care a fost definită este adus în domeniul de vizibilitate. În lipsa acestei adnotări, macrocomanda nu ar putea fi adusă în domeniul de vizibilitate.

Inițiem definiția de macrocomandă cu `macro_rules!` și numele macrocomenzii care se definește *fără* semnul de exclamare. În exemplul nostru, numele este `vec`, și este urmat de acolade care trasează granițele corpului definiției macrocomenzii.

Structura din corpul `vec!` seamănă cu cea a unei expresii `match`. În acest caz, avem o singură ramură cu pattern-ul `( $( $x:expr ),* )`, urmat de `=>` și blocul de cod asociat acestui pattern. Când pattern-ul corespunde, blocul de cod corespondent este generat. Deoarece acesta este singurul pattern din macrocomandă, aceasta este singura formă de pattern acceptată; orice altă formă va cauza o eroare. Macrocomenzile mai complexe pot avea mai multe ramuri.

Sintaxa pattern-urilor valide în definiţiile macrocomenzilor este diferită de sintaxa pattern-urilor discutate în Capitolul 18, deoarece pattern-urile macro se potrivesc cu structura codului Rust, nu cu valorile. Să explorăm semnificația componentelor pattern-ului din Listarea 19-28; pentru ințelegerea completa a sintaxei pattern-urilor de macrocomandă, consultați [Referința Rust][ref].

Mai întâi, folosim un set de paranteze pentru a înconjura întreg pattern-ul. Utilizăm un semn dolar (`$`) pentru a declara o variabilă în sistemul de macrocomenzi care va conține codul Rust ce corespunde pattern-ului. Semnul dolar face clară distincția că aceasta este o variabilă de macrocomandă și nu una Rust obișnuită. Apoi vine un set de paranteze care captează valorile ce se potrivesc cu pattern-ul din interiorul parantezelor pentru utilizare în codul substituit. În cadrul `$()` este `$x:expr`, care se potrivește cu orice expresie Rust și îi atribuie expresiei numele `$x`.

Virgula de după `$()` indică faptul că un caracter separator literal de virgulă poate apărea opțional după codul care se potrivește în `$()`. Simbolul `*` indică că pattern-ul se aplică de zero sau mai multe ori pentru orice precede `*`.

Când apelăm această macrocomandă folosind `vec![1, 2, 3];`, pattern-ul `$x` se aplică de trei ori pentru cele trei expresii `1`, `2` și `3`.

Acum să analizăm pattern-ul din corpul codului asociat cu acest segment: `temp_vec.push()` generat în `$()*` se produce pentru fiecare element care corespunde cu `$()` în pattern, de zero ori sau mai multe, în funcție de numărul de potriviri ale pattern-ului. Variabila `$x` este înlocuită cu fiecare expresie potrivită. Apelarea acestei macrocomenzi cu `vec![1, 2, 3];`, generează următorul cod care substituie apelul macrocomenzii:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Am definit o macrocomandă care poate accepta orice număr de argumente de orice tip și poate genera cod pentru crearea unui vector care conține elementele indicate.

Pentru a învăța mai multe despre cum se scriu macrocomenzile, consultă documentația online sau alte resurse, precum [„The Little Book of Rust Macros”][tlborm] inițiat de Daniel Keep și continuat de Lukas Wirth.

### Macrocomenzi procedurale pentru generarea de cod din atribute

A doua categorie de macrocomenzi este *macrocomanda procedurală*, care funcționează mai mult ca o funcție (fiind un tip de procedură). Macrocomenzile procedurale primesc un cod ca intrare, îl prelucrează și generează cod ca ieșire, spre deosebire de macrocomenzile declarative, care se bazează pe potrivirea de pattern-uri și înlocuirea codului cu altul. Există trei tipuri de macrocomenzi procedurale: "custom derive", "attribute-like" și "function-like", toate având o modalitate de operare asemănătoare.

Când dezvoltăm macrocomenzi procedurale, acestea trebuie să fie plasate în propriul lor crate, de un tip special. Acest lucru se datorează unor motive tehnice complexe care sperăm să fie rezolvate în viitor. În Exemplul 19-29, putem vedea cum se definește o macrocomandă procedurală, unde `some_attribute` servește drept loc pentru un anumit tip de macrocomandă.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

<span class="caption">Exemplu 19-29: Definirea unei macrocomenzi procedurale</span>

Funcția care definește o macrocomandă procedurală primește un `TokenStream` ca intrare și returnează un `TokenStream`. Tipul `TokenStream` este definit de crate-ul `proc_macro`, inclus în Rust, și reprezintă o secvență de token-uri. Acesta este nucleul macrocomenzii: codul sursă asupra căruia operează macrocomanda constituie `TokenStream`-ul de intrare, iar codul generat de macrocomandă este `TokenStream`-ul de ieșire. Funcția are, de asemenea, un atribut atașat care specifică ce fel de macrocomandă procedurală este creată. Putem defini mai multiple varietăți de macrocomenzi procedurale în același crate.

Să explorăm diferitele feluri de macrocomenzi procedurale. Începem cu macrocomanda de tip "custom derive" și apoi vom clarifica subtilele diferențe care disting celelalte forme.

### Cum să scrii o macrocomandă `derive` personalizată

Să creăm un crate numit `hello_macro` care definește o trăsătură numită `HelloMacro` cu o funcție asociată numită `hello_macro`. În loc să-i cerem fiecărui utilizator să implementeze trăsătura `HelloMacro` pentru tipurile lor, vom furniza o macrocomandă procedurală care permite utilizatorilor să adnoteze tipurile lor cu `#[derive(HelloMacro)]` și astfel să obțină o implementare implicită a funcției `hello_macro`. Această implementare implicită va afișa `Hello, Macro! Numele meu este TypeName!`, unde `TypeName` este numele tipului pentru care a fost definită trăsătura. Cu alte cuvinte, scriem un crate care permite unui alt programator să scrie codul prezentat în Listarea 19-30 folosind crate-ul nostru.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-30/src/main.rs}}
...
```

<span class="caption">Listarea 19-30: Codul pe care un utilizator al crate-ului nostru îl poate scrie utilizând macrocomanda noastră procedurală</span>

Acest cod va afișa `Hello, Macro! My name is Pancakes!` când încheiem. Primul pas este să creăm un nou crate de tip bibliotecă, așa:

```console
$ cargo new hello_macro --lib
```

Apoi, vom defini trăsătura `HelloMacro` și funcția sa asociată:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/hello_macro/src/lib.rs}}
```

Avem o trăsătură și funcția ei. În acest moment, utilizatorul nostru ar putea să implementeze trăsătura pentru a obține funcționalitatea dorită, în următorul mod:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/pancakes/src/main.rs}}
```

Însă, ar trebui să redacteze blocul de implementare pentru fiecare tip pe care doresc să-l utilizeze împreună cu `hello_macro`; scopul nostru este să-i scutim de acest efort.

În plus, în momentul de față nu putem oferi funcției `hello_macro` o implementare implicită care să afișeze numele tipului pe care este implementată trăsătura: Rust nu dispune de facilități de reflecție, deci nu poate determina numele tipului în timpul rulării. Avem nevoie de o macrocomandă care să genereze cod în timpul compilării.

Următorul pas este să definim macrocomanda procedurală. La momentul redactării acestei cărți, macrocomenzile procedurale trebuie să fie în propriul lor crate. Este posibil ca în viitor această restricție să fie eliminată. Convenția pentru crate-urile structurale și  crate-urile de macrocomenzi este următoarea: pentru un crate denumit `foo`, un crate de macrocomandă procedurală custom derive este denumit `foo_derive`. Să începem un nou crate numit `hello_macro_derive` în interiorul proiectului nostru `hello_macro`:

```console
$ cargo new hello_macro_derive --lib
```

Cele două crate-uri sunt strâns legate, așadar creăm crate-ul de macrocomandă procedurală în directoriul crate-ului nostru `hello_macro`. Dacă modificăm definiția trăsăturii în `hello_macro`, va trebui să ajustăm și implementarea macrocomenzii procedurale în `hello_macro_derive`. Cele două crate-uri trebuie publicate separat, iar programatorii care utilizează aceste crate-uri trebuie să le adauge pe amândouă ca dependențe și să le includă în domeniul de vizibilitate. Totuși, am putea face ca `hello_macro` să folosească `hello_macro_derive` ca dependență și să reexporte codul macrocomenzii procedurale. Cu toate acestea, modul în care am structurat proiectul permite programatorilor să folosească `hello_macro` chiar și dacă nu doresc funcționalitatea `derive`.

Este necesar să declarăm crate-ul `hello_macro_derive` drept un crate de macrocomandă procedurală. De asemenea, avem nevoie de funcționalități din crate-urile `syn` și `quote`, așa cum vom vedea în curând, deci trebuie să le aducem ca dependențe. Adăugăm următoarele în fișierul *Cargo.toml* pentru `hello_macro_derive`:

<span class="filename">Numele fișierului: hello_macro_derive/Cargo.toml</span>

```toml
{{#include ../listings/ch19-advanced-features/listing-19-31/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

Pentru a începe definirea macrocomenzii procedurale, introducem codul din Listarea 19-31 în fișierul nostru *src/lib.rs* pentru crate-ul `hello_macro_derive`. Acest cod nu va compila până când nu adăugăm o definiție pentru funcția `impl_hello_macro`.

<span class="filename">Filename: hello_macro_derive/src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-31/hello_macro/hello_macro_derive/src/lib.rs}}
```

<span class="caption">Listarea 19-31: Codul necesar majorității crate-urilor de macrocomenzi procedurale pentru a prelucra cod Rust</span>

Am observat că am divizat codul în funcția `hello_macro_derive`, care este responsabilă cu analiza `TokenStream`, și în funcția `impl_hello_macro`, care gestionează transformarea arborelui sintactic: aceasta ușurează procesul de scriere a unei macrocomenzi procedurale. Codul din funcția exterioară (`hello_macro_derive`, în acest caz) va fi același pentru majoritatea crate-urilor de macrocomenzi procedurale întâlnite sau create. Însă, codul din interiorul funcției interne (`impl_hello_macro`, în acest caz) se va modifica în funcție de scopul final al macrocomenzii procedurale respective.

Am aflat de trei crate-uri noi: `proc_macro`, [`syn`] și [`quote`]. Crate-ul `proc_macro` este furnizat odată cu Rust, așadar nu a fost necesar să îl adăugăm în *Cargo.toml* la dependențe. Crate-ul `proc_macro` constituie API-ul compilatorului care ne permite să citim și să modificăm codul Rust din cadrul codului nostru.

Crate-ul `syn` transformă codul Rust dintr-un string într-o structură de date pe care putem aplica operațiuni. Crate-ul `quote` reconstruiește structuri de date `syn` în cod Rust. Utilizarea acestor crate-uri simplifică semnificativ procesul de parsare a diferitelor tipuri de cod Rust pe care dorim să le gestionăm: construirea unui parser complet pentru Rust reprezintă o provocare considerabilă.

Funcția `hello_macro_derive` este activată când un utilizator al bibliotecii noastre adaugă `#[derive(HelloMacro)]` la un tip. Posibilitatea aceasta se datorează faptului că am marcat funcția `hello_macro_derive` cu `proc_macro_derive` și i-am atribuit numele `HelloMacro`, concordând cu numele trăsăturii noastre, urmând astfel convenția adoptată de majoritatea macrocomenzilor procedurale.

Funcția `hello_macro_derive` începe prin a converti `input`-ul dintr-un `TokenStream` într-o structură de date care ulterior poate fi interpretată și manipulată. Acest proces implică biblioteca `syn`. Metoda `parse` de la `syn` primește un `TokenStream` și generează o structură `DeriveInput`, care reprezintă codul Rust parsat. Listarea 19-32 ne arată părțile relevante ale structurii `DeriveInput` pe care o primim atunci când analizăm șirul `struct Pancakes;`:

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

<span class="caption">Listarea 19-32: Instanța `DeriveInput` pe care o primim în urma parsării codului ce conține atributul macrocomenzii din Listarea 19-30</span>

Câmpurile acestei structuri ne indică faptul că codul Rust analizat este un struct de tip unitar cu identificatorul `ident` având numele `Pancakes`. Structura include mai multe câmpuri pentru descrierea diverselor tipuri de coduri Rust; pentru informații suplimentare consultă documentația `syn` referitoare la `DeriveInput`.

În curând, vom defini funcția `impl_hello_macro`, acolo unde vom crea noul cod Rust pe care dorim să-l integram. Însă înainte de acest pas, este important de reținut că ieșirea macrocomenzii noastre derivate este, de asemenea, un `TokenStream`. Acest `TokenStream` returnat este inclus în codul scris de utilizatorii crate-ului nostru, astfel încât, atunci când compilează crate-ul, acesta va include funcționalitățile suplimentare pe care le furnizăm noi prin intermediul `TokenStream` modificat.

Poate ai observat că utilizăm `unwrap` care va declanșa o panică în funcția `hello_macro_derive` dacă apelul la funcția `syn::parse` nu reușește. Această abordare este necesară deoarece macrocomenzile procedurale trebuie să genereze panică în caz de erori, având în vedere că funcțiile `proc_macro_derive` trebuie să returneze `TokenStream` în loc de `Result` pentru a fi conforme cu interfața API de macrocomenzi procedurale. Acest exemplu a fost simplificat prin utilizarea lui `unwrap`; într-un cod destinat producției, ar trebui să oferim mesaje de eroare mai detaliate cu privire la natura problemei folosind `panic!` sau `expect`.

Acum că avem codul pentru conversia codului Rust adnotat de la un `TokenStream` la o instanță de tip `DeriveInput`, să trecem la generarea codului care implementează trăsătura `HelloMacro` pe tipul adnotat, conform exemplului din Listarea 19-33.

<span class="filename">Numele fișierului: hello_macro_derive/src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-33/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

<span class="caption">Listarea 19-33: Implementarea trăsăturii `HelloMacro` utilizând codul Rust parsat</span>

Prin intermediul `ast.ident` obținem o instanță de tip `Ident` care include numele (identificatorul) tipului adnotat. Structura prezentată în Listarea 19-32 ne demonstrează că, atunci când executăm funcția `impl_hello_macro` asupra codului din Listarea 19-30, `ident`-ul obținut va avea câmpul `ident` cu valoarea `"Pancakes"`. Deci, variabila `name` din Listarea 19-33 va conține o structură de tip `Ident` care, când va fi afișată, va rezulta în string-ul `"Pancakes"`, numele structurii din Listarea 19-30.

Macrocomanda `quote!` ne facilitează definirea codului Rust pe care dorim să îl returnăm. Compilatorul așteaptă o formă diferită în comparație cu rezultatul direct al executării macrocomenzii `quote!`, astfel că este necesar să o convertim în `TokenStream`. Acest lucru se realizează apelând metoda `into`, care transformă această reprezentare intermediară într-o valoare de tipul necesar `TokenStream`.

Macrocomanda `quote!` oferă, de asemenea, mecanisme de șabloane extrem de utile: dacă inserăm `#name`, `quote!` va substitui acesta cu valoarea din variabila `name`. Se pot realiza chiar repetiții similare cu cea a macrocomenzilor obișnuite. Pentru o înțelegere detaliată, vezi [documentația crate-ului `quote`][quote-docs].

Dorim ca macrocomanda noastră procedurală să creeze o implementare a trăsăturii `HelloMacro` pentru tipul adnotat de utilizator, lucru pe care îl obținem utilizând `#name`. Implementarea respectivei trăsături conține funcția `hello_macro`, al cărei corp include funcționalitatea pe care dorim să o oferim: afișarea mesajului `Hello, Macro! My name is` urmat de numele tipului adnotat.

Macrocomanda `stringify!`, ce este utilizată aici, este integrată direct în Rust. Ea preia o expresie Rust, cum ar fi `1 + 2`, și o transformă într-un string literal la timpul de compilare, precum `"1 + 2"`. Aceasta diferă de macrocomenzile `format!` sau `println!`, care evaluează expresia și apoi convertesc rezultatul în `String`. Având în vedere că intrarea `#name` ar putea fi o expresie ce trebuie tipărită literal, folosim `stringify!`. Folosirea `stringify!` economisește de asemenea o alocare, convertind `#name` într-un string literal încă din timpul compilării.

În acest stadiu, comanda `cargo build` ar trebui să se execute cu succes pe crate-urile `hello_macro` și `hello_macro_derive`. Să integăm aceste crate-uri cu codul din Listarea 19-30 pentru a vedea macrocomanda procedurală în acțiune! Creează un nou proiect binar în directoriul tău *projects* folosind comanda `cargo new pancakes`. Este necesar să adăugăm `hello_macro` și `hello_macro_derive` ca dependențe în fișierul *Cargo.toml* al crate-ului `pancakes`. Dacă vei publica versiunile tale de `hello_macro` și `hello_macro_derive` pe [crates.io](https://crates.io/), acestea vor fi dependențe obișnuite; în caz contrar, poți să le specifici ca dependențe de tip `path` în modul următor:

```toml
{{#include ../listings/ch19-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:7:9}}
```

Adaugă codul din Listarea 19-30 în fișierul *src/main.rs* și execută comanda `cargo run`: ar trebui să afișeze mesajul `Hello, Macro! My name is Pancakes!` Implementarea trăsăturii `HelloMacro`, provenită din macrocomanda procedurală, a fost inclusă fără ca librăria `pancakes` să fie nevoită să o implementeze manual; atributul `#[derive(HelloMacro)]` a adăugat automat implementarea trăsăturii.

În pasul următor, vom descoperi în ce mod se diferențiază celelalte tipuri de macrocomenzi procedurale față de macrocomenzile custom derive.

### Macrocomenzile de tip atribut

Macrocomenzile tip atribut sunt similare cu macrocomenzile personalizate de derive, însă spre deosebire de generarea de cod pentru atributul `derive`, ele permit crearea de atribute noi. Acestea sunt și mai versatile: `derive` funcționează exclusiv pentru structuri și enum-uri; atributele pot fi utilizate și pentru alte elemente, cum ar fi funcții. De exemplu, să presupunem că deții un atribut denumit `route`, care adnotează funcțiile atunci când lucrezi cu un framework pentru aplicații web:

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Atributul `#[route]` ar fi definit de către framework sub formă de o macrocomandă procedurală. Semnătura funcției care definește macrocomanda va arăta astfel:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Aici avem doi parametri de tipul `TokenStream`. Primul este pentru conținutul atributului, adică partea cu `GET, "/"`. Cel de-al doilea este corpul entității căreia i s-a atașat atributul: în acest caz, `fn index() {}` și restul corpului funcției.

În rest, macrocomenzile tip atribut funcționează în aceeași manieră ca și macrocomenzile de derive personalizate: creezi un crate de tip `proc-macro` și implementezi o funcție care generează codul dorit!

### Macrocomenzi de tip funcție

Macrocomenzile de tip funcție definesc macrocomenzi care seamănă cu apelurile de funcții. Asemenea macrocomenzilor `macro_rules!`, sunt mai flexibile decât funcțiile; spre exemplu, pot accepta un număr variabil de argumente. Însă, macrocomenzile `macro_rules!` pot fi definite doar folosind sintaxa asemănătoare match-ului despre care am discutat în secțiunea [„Macrocomenzi declarative cu `macro_rules!` pentru metaprogramare generală”][decl]<!-- ignore -->. Macrocomenzile de tip funcție primesc un parametru `TokenStream` și manipulează acest `TokenStream` folosind cod Rust, asemenea celorlalte două tipuri de macrocomenzi procedurale. Un exemplu de macrocomandă de tip funcție ar fi macrocomanda `sql!`, care ar putea fi folosită în felul următor:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Această macrocomandă ar analiza instrucțiunea SQL conținută și ar verifica dacă este sintactic corectă, o complexitate de prelucrare ce depășește posibilitățile unei macrocomenzi `macro_rules!`. Macrocomanda `sql!` ar fi definită în felul următor:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Această definiție prezintă similarități cu semnătura unei macrocomenzi pentru derive personalizat: primim tokenii care sunt între paranteze și returnăm codul pe care dorim să-l generăm.

## Sumar

Ei bine! Acum în trusa ta de unelte ai câteva funcționalități Rust pe care nu le vei utiliza des, însă, în situații foarte specifice, vei ști că-ți sunt la dispoziție. Am abordat câteva subiecte complexe astfel încât, atunci când te vei întâlni cu ele în sugestiile de eroare ale mesajelor sau în codul altor persoane, să poți recunoaște aceste concepte și sintaxa aferentă. Utilizează acest capitol ca un ghid de referință care să te ajute să găsești soluții.

Următorul pas este să punem în aplicare tot ceea ce am învățat de-a lungul cărții și să lucrăm la un nou proiect!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[`syn`]: https://crates.io/crates/syn
[`quote`]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/1.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
