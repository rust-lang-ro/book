## Traits: definirea comportamentului partajat

O *trăsătură* (trait) definește funcționalitățile pe care un tip le posedă și care pot fi partajate cu alte tipuri. Noi utilizăm trăsăturile pentru a stabili în mod abstract comportamentul partajat. Totodată, prin *limitele trăsăturii* specificăm că un tip generic poate fi orice tip care manifestă anumite comportamente.

> Notă: Trăsăturile sunt similare cu o funcționalitate frecvent întâlnită în
> alte limbaje de programare, des numită *interfețe*, deși există diferențe
> notabile.

### Definirea unei trăsături

Comportamentul unui tip este caracterizat de metodele care pot fi invocate pe acel tip. Diverse tipuri au un comportament comun dacă este posibil să apelăm aceleași metode pe fiecare dintre ele. Definițiile trăsăturilor reprezintă o metodă de a grupa semnături de metode pentru a contura un set de comportamente necesare pentru îndeplinirea unui obiectiv anume.

Să presupunem, de exemplu, că dispunem de structuri multiple care stochează diferite cantități și tipuri de text: o structură `NewsArticle` care conține o știre legată de o locație specifică și un `Tweet` care poate avea până la 280 de caractere, împreună cu metadate care precizează dacă acesta este un tweet nou, un retweet sau un răspuns la un alt tweet.

Ne propunem să construim o crate de bibliotecă de agregare ale articolelor media numită `aggregator`, capabilă să prezinte sumare ale datelor care pot fi conținute de instanțe ale `NewsArticle` sau `Tweet`. Pentru acest lucru, avem nevoie de un rezumat din partea fiecărui tip, pe care îl solicităm prin apelarea metodei `summarize` pe o instanță respectivă. Listarea 10-12 ilustrează definiția unei trăsături publice `Summary` ce exprimă acest comportament.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

<span class="caption">Listarea 10-12: O trăsătură `Summary` ce constă din comportamentul oferit de metoda `summarize`</span>

În acest caz, introducem o trăsătură utilizând cuvântul cheie `trait` urmat de numele trăsăturii, care este `Summary`. Am declarat trăsătura ca fiind publică (`pub`), astfel încât alte crate-uri care depind de acesta să aibă posibilitatea de a o folosi, așa cum vom observa în unele exemple ulterioare. În spațiul dintre acolade, sunt prezentate semnăturile metodelor care definesc comportamentele asumate de tipurile care implementează această trăsătură; în cazul de față fiind `fn summarize(&self) -> String`.

În loc să furnizăm o implementare a metodei între acolade, punem un punct și virgulă după semnătură. Fiecărui tip care realizează această trăsătură îi revine sarcina de a implementa propriul comportament pentru corpul metodei. Compilatorul va asigura că orice tip cu trăsătura `Summary` va avea definită metoda `summarize` cu anume această semnătură exactă.

O trăsătură poate include în corpul său mai multe metode: semnăturile acestora sunt enumerate independent, pe rânduri separate, iar fiecare rând se încheie cu un punct și virgulă.

### Implementarea unei trăsături pentru un tip

Noi am definit semnăturile dorite ale metodelor trăsăturii `Summary` și acum e timpul să le implementăm pentru tipurile din agregatorul nostru de media. Listarea 10-13 ilustrează implementarea trăsăturii `Summary` pentru structura `NewsArticle`, utilizând titlul, autorul și locația pentru a crea valoarea de retur a metodei `summarize`. În cazul structurii `Tweet`, metoda `summarize` este definită ca numele de utilizator urmat de întregul text al tweet-ului, având în vedere că conținutul tweet-ului este limitat la 280 de caractere.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

<span class="caption">Listarea 10-13: Implementarea trăsăturii `Summary` pentru tipurile `NewsArticle` și `Tweet`</span>

Implementarea unei trăsături pentru un tip este similară cu implementarea metodelor obișnuite. Diferența constă în faptul că, după `impl`, specificăm numele trăsăturii pe care dorim să o realizăm, folosim cuvântul cheie `for`, și apoi numele tipului pentru care implementăm trăsătura. În blocul `impl`, includem semnăturile metodelor definite de trăsătura respectivă. În loc să finalizăm fiecare semnătură cu un punct și virgulă, adăugăm acolade și detaliem corpul fiecărei metode pentru a defini comportamentul specific pe care îl dorim de la metodele trăsăturii pentru tipul respectiv.

Odată ce biblioteca noastră a implementat trăsătura `Summary` pentru `NewsArticle` și `Tweet`, utilizatori crate-ului pot folosi metodele trăsăturii pe instanțe de `NewsArticle` și `Tweet`, similar cu modul în care sunt apelate metodele obișnuite. Singura diferență este că utilizatorii trebuie să includă atât trăsătura cât și tipurile în domeniul de vizibilitate. Aici este un exemplu de cum ar putea fi utilizată biblioteca noastră `aggregator` în cadrul unui crate binar:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Acest exemplu de cod va afișa `1 new tweet: horse_ebooks: of course, as you probably already know, people`.

Alte crate-uri care utilizează crate-ul `aggregator` pot de asemenea să își aducă trăsătura `Summary` în domeniul de vizibilitate pentru a o implementa pe tipurile proprii. O restricție de reținut este că putem implementa o trăsătură pe un tip doar în cazul în care cel puțin unul dintre elemente - trăsătura sau tipul - este definit local în propriul nostru crate. De exemplu, putem implementa trăsături standard ale bibliotecii, cum ar fi `Display`, pentru un tip nativ crate-ului nostru ca `Tweet`, ca parte a funcționalității `aggregator`. De asemenea, putem implementa `Summary` pentru `Vec<T>` în cadrul `aggregator`, din moment ce trăsătura `Summary` este locală crate-ului nostru.

Cu toate acestea, nu avem posibilitatea să implementăm trăsături externe pentru tipuri externe. Spre exemplu, nu putem să realizăm implementarea trăsăturii `Display` pentru `Vec<T>` în cadrul crate-ului `aggregator`, pentru că atât `Display` cât și `Vec<T>` sunt parte din biblioteca standard și nu sunt locale lui `aggregator`. Această limitare este o parte din principiul de *coerență*, mai precis din *regula orfanilor*. Această regulă asigură că codul altor persoane nu poate interfera cu al tău și în același timp protejează codul tău de a fi afectat de alții. Fără această regulă, ar exista posibilitatea ca două crate-uri diferite să implementeze aceeași trăsătură pentru același tip, ceea ce ar crea confuzie pentru Rust cu privire la care implementare ar trebui utilizată.

### Implementări implicite

Câteodată e util să avem un comportament implicit pentru unele sau pentru toate metodele unei trăsături în loc să necesităm implementări pentru toate metodele pe fiecare tip. Prin urmare, atunci când implementăm trăsătura pe un anumit tip, putem menține sau suprascrie comportamentul implicit al fiecărei metode.

În Listarea 10-14, am specificat un string implicit pentru metoda `summarize` a trăsăturii `Summary` în loc doar să definim semnătura metodei, cum am făcut în Listarea 10-12.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

<span class="caption">Listarea 10-14: Definirea trăsăturii `Summary` cu o implementare implicită a metodei `summarize`</span>

Pentru a folosi o implementare implicită în rezumarea instanțelor de `NewsArticle`, specificăm un bloc `impl` gol cu `impl Summary for NewsArticle {}`.

Deși nu mai definim metoda `summarize` în mod direct pe `NewsArticle`, am furnizat o implementare implicită și am specificat că `NewsArticle` implementează trăsătura `Summary`. Drept rezultat, putem totuși apela metoda `summarize` pe o instanță de `NewsArticle`, astfel:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

Acest cod afișează `New article available! (Read more...)`.

Crearea unei implementări implicite nu ne obligă să modificăm nimic în ceea ce privește implementarea `Summary` pentru `Tweet`, conform Listării 10-13. Acest lucru este datorat faptului că sintaxa pentru a suprascrie o implementare implicită este identică cu sintaxa pentru implementarea unei metode de trăsătură care nu are nicio implementare implicită.

Implementările implicite pot apela alte metode din aceeași trăsătură, chiar dacă aceste alte metode nu dispun de implementări implicite. Astfel, o trăsătură poate oferi multe funcționalități utile și poate cere implementatorilor să definească doar o mică parte din acelea. De exemplu, am putea defini trăsătura `Summary` să includă o metodă `summarize_author` ce necesită implementare, și apoi să definim o metodă `summarize` care are o implementare implicită ce invocă `summarize_author`:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

Când folosim această versiune de `Summary`, trebuie să definim `summarize_author` doar când implementăm trăsătura pentru un tip:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

După ce definim `summarize_author`, putem invoca `summarize` pe instanțele structurii `Tweet`, iar implementarea implicită a `summarize` va utiliza definiția de `summarize_author` pe care am furnizat-o. Întrucât am implementat `summarize_author`, trăsătura `Summary` ne oferă funcționalitatea metodei `summarize` fără să fie nevoie să scriem cod suplimentar.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

Acest cod afișează `1 new tweet: (Read more from @horse_ebooks...)`.

Notă: nu este posibil să apelăm implementarea implicită din cadrul unei implementări care o suprascrie pe aceeași metodă.

### Utilizarea trăsăturilor drept parametri

Înarmat cu abilitatea de a defini și implementa trăsături, ești acum pregătit să descoperi cum să folosești trăsături pentru a concepe funcții care să accepte o varietate de tipuri diferite. Vom apela la trăsătura `Summary`, implementată pentru `NewsArticle` și `Tweet` în Listarea 10-13, pentru a defini funcția `notify` care invocă metoda `summarize` pe parametrul său `item`. Acest `item` trebuie să fie de un tip care implementează trăsătura `Summary`. Procedăm astfel folosind sintaxa `impl Trait`, în modul următor:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

În loc de un tip explicit pentru parametrul `item`, specificăm cuvântul cheie `impl` și numele trăsăturii. Acest parametru va accepta orice tip care implementează trăsătura indicată. În cadrul funcției `notify`, avem posibilitatea să apelăm orice metode asociate cu `item` ce derivă din trăsătura `Summary`, precum `summarize`. Funcția `notify` poate fi apelată utilizând orice exemplar de `NewsArticle` sau `Tweet`. Încercarea de a folosi funcția cu tipuri care nu implementează `Summary`, cum ar fi `String` sau `i32`, nu va fi compilată, deoarece aceste tipuri nu îndeplinesc cerința trăsăturii `Summary`.

<!-- Old headings. Do not remove or links may break. -->
<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Sintaxa delimitării de trăsături

Sintaxa `impl Trait` este practică în situații simple, dar de fapt reprezintă o formă abreviată pentru o formă mai extinsă, cunoscută ca *delimitare de trăsături* (trait bound); aceasta se prezintă astfel:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Această variantă completă este identică cu exemplul din secțiunea anterioară, dar mai explicită. Plasăm delimitările de trăsături împreună cu declararea parametrului de tip generic după două puncte (:) și în interiorul parantezelor unghiulare.

Sintaxa `impl Trait` oferă concizie în cazurile simple, în timp ce sintaxa extinsă a delimitărilor de trăsături poate să exprime mai multă complexitate în alte situații. De pildă, putem avea doi parametri ce implementează `Summary`. Aplicarea sintaxei `impl Trait` apare în felul următor:

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

Utilizarea `impl Trait` este adecvată dacă dorim ca funcția să permită lui `item1` și `item2` să fie de tipuri diferite (atât timp cât ambele tipuri implementează `Summary`). Totuși, dacă vrem să constrângem ambii parametri să fie de același tip, atunci trebuie să apelăm la delimitarea de trăsături, ca în exemplul următor:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

Tipul generic `T`, specificat ca tip pentru parametrii `item1` și `item2`, restricționează funcția astfel încât tipul concret al valorilor transmise ca argumente pentru `item1` și `item2` trebuie să fie același.

#### Indicarea mai multor delimitări de trăsătură folosind sintaxa `+`

Este posibil să specificăm concurent mai multe delimitări de trăsătă. De exemplu, dacă dorim ca `notify` să utilizeze formatare prin afișare, precum și metoda `summarize` pentru `item`, trebuie să notăm în definiția `notify` că `item` trebuie să implementeze trăsăturile `Display` și `Summary`. Putem realiza acest lucru utilizând sintaxa `+`:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

Această sintaxă `+` poate fi folosită și în cazul limitelor impuse pe trăsături pentru tipuri generice:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

Odată ce ambele trăsături sunt specificate, în cadrul lui `notify` putem invoca `summarize` și folosi `{}` pentru a formata `item`, ceea ce este posibil datorită implementării trăsăturii `Display`.

#### Delimitări de trăsătură mai clare cu clauza `where`

Abundența de delimitări de trăsătură pe generici poate avea dezavantaje. Fiecare tip generic vine cu propriile sale restricții de trăsătură, astfel funcțiile cu mai mulți parametri generici pot fi supraîncărcate cu informații privind trăsăturile între numele funcției și lista parametrilor, complicând citirea semnăturii funcției. Pentru a simplifica această problemă, Rust oferă o sintaxă alternativă prin utilizarea unei clauze `where` care se plasează după semnătura funcției. Astfel, în loc de a scrie în modul următor:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

putem opta pentru utilizarea unei clauze `where`, cum ar fi:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

Semnătura funcției este astfel mai clară: numele funcției, lista parametrilor și tipul returnat sunt regrupate în proximitate, similar unei funcții simple care nu include numeroase restricții de trăsătură.

### Returnarea tipurilor ce implementează trăsături

Este posibil de asemenea să utilizăm sintaxa `impl Trait` în poziția de returnare pentru a returna o valoare de un tip care implementează o trăsătură, așa cum este arătat în exemplul de mai jos:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

Prin aplicarea `impl Summary` ca tip de retur, indicăm că funcția `returns_summarizable` va întoarce un tip ce implementează trăsătura `Summary`, fără a specifica tipul concret. În acest caz specific, `returns_summarizable` returnează un `Tweet`, dar codul care cheamă această funcție nu are nevoie să cunoască acest amănunt.

Posibilitatea de a specifica un tip de retur exclusiv prin trăsătura implementată se dovedește a fi extrem de valoroasă în contextul închiderilor (closure) și iteratorilor, teme ce vor fi explorate în Capitolul 13. Aceste închideri și iteratori produc tipuri cunoscute doar de compilator sau tipuri care ar fi oneros de specificat complet. Sintaxa `impl Trait` facilitează specificarea succintă a faptului că o funcție returnează un tip care implementează trăsătura `Iterator`, eliminând necesitatea de a descrie un tip extensiv.

Totuși, `impl Trait` poate fi folosit doar atunci când se returnează un tip unic. Spre exemplu, codul care face returnarea unui `NewsArticle` sau un `Tweet`, având tipul de retur definit ca `impl Summary`, nu va funcționa:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

Este interzisă returnarea fie a unui `NewsArticle`, fie a unui `Tweet`, pe fondul restricțiilor asociate modului de implementare a sintaxei `impl Trait` în compilator. Metodologia de scriere a unei funcții cu acest comportament va fi detaliată în secțiunea [„Utilizând obiecte de trăsătură ce permit valori pentru tipuri diverse”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore --> din Capitolul 17.

### Folosirea delimitărilor de trăsături pentru implementarea condiționată a metodelor

Prin utilizarea unei delimitări de trăsături într-un bloc `impl` ce include parametri de tip generic, noi putem să implementăm metode în mod condiționat pentru tipurile care implementează trăsăturile specificate. Spre exemplu, tipul `Pair<T>` din Listarea 10-15 implementează constant funcția `new` pentru a crea o nouă instanță de `Pair<T>` (reîmprospătăm aici din secțiunea [“Definirea Metodelor”][methods]<!-- ignore --> a Capitolului 5 că `Self` este un sinonim pentru tipul blocului `impl`, care în acest caz este `Pair<T>`). Totuși, în următorul bloc `impl`, `Pair<T>` implementează metoda `cmp_display` doar dacă tipul său intern `T` aderă atât la trăsătura `PartialOrd` ce face posibilă compararea cât *și* la trăsătura `Display` ce permite afișarea.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

<span class="caption">Listarea 10-15: Implementarea condiționată a metodelor pe un tip generic, în dependență de delimitările de trăsătură</span>

De asemenea, putem implementa condiționat o trăsătură pentru orice tip care implementează o altă trăsătură. Aceste implementări, realizate pentru orice tip care îndeplinește delimitările de trăsătură, poartă numele de *implementări generalizate* (blanket implementations) și sunt utilizate extensiv în biblioteca standard Rust. De exemplu, biblioteca standard oferă implementarea trăsăturii `ToString` pentru orice tip care implementează trăsătura `Display`. Blocul `impl` din biblioteca standard este similar cu acest fragment de cod:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Datorită acestei implementări generalizate prezente în biblioteca standard, noi putem invoca metoda `to_string`, definită de trăsătura `ToString`, pe orice tip ce implementează trăsătura `Display`. Ca urmare, avem posibilitatea de a transforma numere întregi în echivalentele lor `String` pentru că întregii implementează `Display`:

```rust
let s = 3.to_string();
```

Implementările generalizate pot fi găsite în documentația referitoare la trăsătura respectivă, în secțiunea “Implementors”.

Trăsăturile și delimitările de trăsături ne permit să concepem cod care utilizează parametrii de tip generic pentru a reduce dublarea, dar ne și permit să indicăm compilatorului că dorim ca tipul generic să prezinte un anumit comportament. Compilatorul, folosind informațiile delimitărilor de trăsături, poate verifica dacă toate tipurile concrete folosite în codul nostru corespund comportamentului cerut. În limbajele de programare cu tipizare dinamică, erorile legate de apeluri ale unor metode nedefinite pe un tip apar la runtime, pe când Rust transferă aceste erori la timpul de compilare, obligându-ne să rezolvăm problemele înainte ca programul nostru să fie capabil să ruleze. Mai mult, evităm necesitatea de a scrie cod care să verifice comportamentul la runtime deoarece verificările au loc în timpul compilării. Acest proces îmbunătățește performanța fără a renunța la flexibilitatea oferită de utilizarea genericilor.

[using-trait-objects-that-allow-for-values-of-different-types]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[methods]: ch05-03-method-syntax.html#defining-methods
