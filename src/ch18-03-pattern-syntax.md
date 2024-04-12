## Sintaxa pattern-urilor

În această secțiune colectăm toată sintaxa validă în pattern-uri și discutăm de ce și când ar putea fi dorită utilizarea fiecărui tip.

### Potrivirea literalilor

După cum am văzut în Capitolul 6, este posibil să potrivim pattern-uri direct cu literali. Următorul cod oferă câteva exemple:

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

Acest cod afișează `one` deoarece valoarea în `x` este 1. Această sintaxă este folositoare atunci când dorim ca codul nostru să efectueze o acțiune dacă primește o valoare concretă specifică.

### Potrivirea variabilelor denumite

Variabilele denumite sunt pattern-uri irefutabile, care se potrivesc cu orice valoare, și le-am folosit deja de mai multe ori în această carte. Totuși, apare o complicație atunci când folosim variabile denumite în expresiile `match`. Deoarece `match` declanșează începutul unui nou domeniu de vizibilitate, variabilele declarate ca parte a unui pattern în interiorul expresiei `match` vor "umbri" acele variabile cu același nume aflate în afara construcției `match`, la fel ca în cazul tuturor variabilelor. În Listarea 18-11, declarăm o variabilă `x` cu valoarea `Some(5)` și o altă variabilă `y` cu valoarea `10`. Apoi, construim o expresie `match` bazată pe valoarea lui `x`. Observăm pattern-urile din ramurile match și `println!` de la final, și încercăm să prevedem ce va afișa codul înainte de a-l rula sau de a citi în continuare.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-11/src/main.rs:here}}
```

<span class="caption">Listarea 18-11: O expresie `match` care include un braț ce introduce o variabilă `y` umbrită</span>

Să vedem ce se întâmplă atunci când se execută expresia `match`. Pattern-ul din primul braț al `match`-ului nu corespunde cu valoarea definită a lui `x`, astfel că execuția codului continuă.

Pattern-ul din al doilea braț al `match`-ului prezintă o nouă variabilă, `y`, care va potrivi orice valoare încapsulată într-un `Some`. Deoarece acesta se află într-un domeniu de vizibilitate nou în interiorul expresiei `match`, reprezintă un nou `y`, diferit de `y`-ul pe care l-am declarat inițial cu valoarea 10. Acest `y` recent se va potrivi cu orice valoare dintr-un `Some`, exact ceea ce avem în `x`. Astfel, noul `y` se leagă de valoarea interioară a `Some` din `x`, care este `5`, iar brațul respectiv execută și afișează `Matched, y = 5`.

Dacă `x` ar fi fost `None` în loc de `Some(5)`, pattern-urile din primele două brațe nu s-ar fi potrivit, iar valoarea ar fi corespuns cu caracterul underscore. În această repetiție, nu am introdus variabila `x` în pattern-ul brațului cu underscore, deci `x` din expresie este în continuare `x`-ul extern care nu a fost umbrit. În acest scenariu ipotetic, `match`-ul ar fi afișat `Default case, x = None`.

După finalizarea expresiei `match`, domeniul de vizibilitate a acesteia se încheie, iar `y`-ul intern nu mai este accesibil. Ultimul `println!` rezultă în `at the end: x = Some(5), y = 10`.

Pentru a forma o expresie `match` care să compare valorile lui `x` și `y` externe, în loc să introducem o variabilă umbrită, ar trebui să folosim o condiție suplimentară cu gardă de match. Vom discuta despre gărzile de match într-o secțiune ulterioară numită [„Condiții suplimentare cu gărzi de match”](#extra-conditionals-with-match-guards)<!-- ignore -->.

### Pattern-uri multiple

În expresiile de tip `match`, putem să corelăm mai multe pattern-uri utilizând operatorul `|`, care reprezintă pattern-ul *sau*. De exemplu, în codul de mai jos, comparăm valoarea lui `x` cu ramurile de match, iar prima dintre ele, având o opțiune *sau*, va rula codul aferent dacă `x` se potrivește cu oricare dintre valorile specificate în acea ramură:

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

Acest cod va genera afișajul `one or two`.

### Potrivirea diapazoanelor de valori cu `..=`

Prin utilizarea sintaxei `..=`, putem potrivi o secvență întreagă de valori. În exemplul următor, dacă un pattern corespunde cu oricare dintre valorile din diapazon, ramura respectivă va fi activată:

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

Dacă `x` este unul dintre numerele 1, 2, 3, 4 sau 5, prima ramură va fi selectată. Această metodă de specificare a unui diapazon este mai eficientă decât utilizarea repetată a operatorului `|`, evitând astfel o construcție de tipul `1 | 2 | 3 | 4 | 5`. Acest mod de exprimare este deosebit de util când dorim să potrivim un interval extins, precum între 1 și 1,000!

Compilatorul verifică în timpul compilării că diapazonul selectat nu este gol, iar Rust permite folosirea diapazoanelor doar pentru tipurile `char` și numeric, unde este posibil să se constate dacă intervalul este sau nu populat.

Iată cum arată utilizarea diapazoanelor pentru valori de tip `char`:

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust detectează că `'c'` se află în domeniul primei ramuri și afișează `early ASCII letter`.

### Destructurarea structurilor, enum-urilor și tuplelor

Pattern-urile sunt de asemenea utile în destructurarea structurilor, enumerărilor și tuplelor, oferindu-ne posibilitatea de a accesa diferite secțiuni ale acestor tipuri de valori. Analizăm fiecare tip în parte.

#### Destructurarea structurilor

Listarea 18-12 prezintă o structură `Point` cu două câmpuri, `x` și `y`, pe care le putem separa folosind un pattern într-o instrucțiune `let`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-12/src/main.rs}}
```

<span class="caption">Listarea 18-12: Separarea câmpurilor unei structuri în variabile diferite</span>

Acest cod generează variabilele `a` și `b` care preiau valorile câmpurilor `x` și `y` ale structurii `p`. Acest exemplu ne arată că numele variabilelor din pattern nu trebuie să corespundă cu numele câmpurilor structurii. Cu toate acestea, este comun să aliniem numele variabilelor cu numele câmpurilor pentru a facilita reținerea sursei variabilelor. Din acest motiv, și pentru că expresia `let Point { x: x, y: y } = p;` include repetiții inutile, Rust oferă o formă prescurtată pentru pattern-uri care se potrivesc cu câmpurile structurilor: e suficient să enumerăm numele câmpului structurii și variabilele rezultate din pattern vor purta aceleași nume. Listarea 18-13 funcționează la fel ca și codul din Listarea 18-12, însă variabilele create în pattern-ul `let` sunt `x` și `y`, nu `a` și `b`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-13/src/main.rs}}
```

<span class="caption">Listarea 18-13: Destructurarea câmpurilor unei structuri folosind o formă prescurtată pentru câmpuri</span>

Acest cod creează variabilele `x` și `y` care se potrivesc cu câmpurile `x` și `y` ale variabilei `p`. Rezultatul este că variabilele `x` și `y` conțin valorile din structura `p`.

De asemenea, putem folosi valori literale în cadrul pattern-ului unei structuri, în loc să generăm variabile pentru fiecare câmp. Aceasta ne permite să verificăm anumite câmpuri pentru valori specifice dar tot creând variabile pentru extragerea valorilor celorlalte câmpuri.

În Listarea 18-14, avem o expresie `match` care categorizează valorile `Point` în trei situații: puncte care se regăsesc exact pe axa `x` (când `y = 0`), pe axa `y` (`x = 0`) sau niciuna dintre acestea.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-14/src/main.rs:here}}
```

<span class="caption">Listarea 18-14: Destructurarea și potrivirea valorilor literale în același pattern</span>

Prima ramură va corespunde oricărui punct de pe axa `x` prin faptul că specifică pentru câmpul `y` să se potrivească cu valoarea literală `0`. Pattern-ul în continuare generează o variabilă `x` care poate fi folosită în cod pentru această ramură.

Similar, a doua ramură corespunde oricărui punct de pe axa `y` prin specificarea că câmpul `x` se potrivește atunci când valoarea este `0` și astfel se generează o variabilă `y` pentru valoarea câmpului `y`. A treia ramură nu definește nicio valoare literală, deci potrivește orice alt `Point` și generează variabile pentru ambele câmpuri `x` și `y`.

În acest exemplu, valoarea `p` se aliniază cu a doua ramură datorită faptului că `x` conține un 0, deci codul va afișa „On the y axis at 7“.

Amintim că o expresie `match` încetează să evalueze ramurile după ce găsește primul pattern corespunzător, astfel încât chiar și pentru `Point { x: 0, y: 0}` care se află și pe axa `x` și pe axa `y`, codul va afișa doar „On the x axis at 0“.

#### Destructurarea enumerărilor

Am destructurat enumerări în această carte (de exemplu, Listarea 6-5 din Capitolul 6), dar nu am discutat explicit faptul că pattern-ul folosit pentru a destructura un enum corespunde cu modul în care sunt definite datele stocate în acel enum. De exemplu, în Listarea 18-15 utilizăm enum-ul `Message` din Listarea 6-2 și compunem un `match` cu pattern-uri ce vor destructura fiecare valoare internă.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-15/src/main.rs}}
```

<span class="caption">Listarea 18-15: Destructurarea variantelor de enum care stochează diferite tipuri de valori</span>

Acest cod va genera afișajul `Change the color to red 0, green 160, and blue 255`. Modificați valoarea lui `msg` pentru a observa execuția codului din celelalte brațe ale `match`-ului.

Pentru variantele de enum care nu conțin date, precum `Message::Quit`, nu putem să continuăm destructurarea. Putem doar să facem match pe valoarea literală `Message::Quit`, fără variabile în acel pattern.

Pentru variantele de enum similare cu structurile, precum `Message::Move`, putem utiliza un pattern asemănător cu cel folosit pentru match pe structuri. După denumirea variantei, introducem acolade și apoi specificăm câmpurile cu variabile, permițându-ne să desfacem componentele pentru a le utiliza în codul acestui braț. Aici aplicăm forma prescurtată, așa cum am procedat în Listarea 18-13.

În cazul variantelor de enum similare cu tuple, ca `Message::Write` ce conține o tuplă cu un singur element și `Message::ChangeColor` ce conține o tuplă cu trei elemente, pattern-ul este similar cu cel utilizat pentru match pe tuple. Numărul variabilelor din pattern trebuie să coincidă cu numărul de elemente din varianta cu care facem match.

#### Destructurarea structurilor și enumerărilor imbricate

Până acum, în exemplele noastre s-a făcut potrivirea structurilor sau enumerărilor la un singur nivel de adâncime, însă potrivirea poate fi folosită și pentru elemente imbricate! De exemplu, codul din Listarea 18-15 poate fi refactorizat pentru a accepta culorile RGB și HSV în mesajul `ChangeColor`, așa cum este prezentat în Listarea 18-16.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-16/src/main.rs}}
```

<span class="caption">Listarea 18-16: Potrivire pe enum-uri imbricate</span>

Pattern-ul primei ramuri în expresia `match` se potrivește cu varianta `Message::ChangeColor` a enumerării, care conține varianta `Color::Rgb`; apoi pattern-ul face legătura cu cele trei valori interne `i32`. Pattern-ul celei de-a doua ramuri se potrivește de asemenea cu varianta `Message::ChangeColor` a enumerării, dar enumerarea internă se potrivește cu `Color::Hsv`. Putem specifica aceste condiții complexe într-o singură expresie `match`, chiar dacă sunt implicate două enumerări.

#### Destructurarea structurilor și tuplelor

Putem combina, potrivi și imbrica pattern-uri de destructurare în moduri chiar mai complexe. Următorul exemplu ilustrează o destructurare avansată în care structurile și tupletele sunt imbricate într-o tuplă și, apoi, extragem toate valorile primitive:

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

Această tehnică ne permite să destructurăm tipuri complexe în componente individuale pentru a putea folosi valorile necesare în mod separat.

Destructurarea cu ajutorul pattern-urilor este un mod convenabil de a accesa părți din valori, cum ar fi valorile din fiecare câmp al unei structuri, în mod separat.

### Ignorarea valorilor într-un pattern

Am observat că este uneori util să ignorăm anumite valori într-un pattern, cum ar fi în cazul ultimei ramuri a unui `match`, pentru a avea un caz general care, deși nu efectuează nicio acțiune concretă, acoperă toate celelalte valori posibile. Există diferite metode pentru a ignora valori întregi sau părți ale valorilor într-un pattern: utilizând pattern-ul `_` (cu care suntem deja familiari), folosind pattern-ul `_` în cadrul unui alt pattern, utilizând un nume care începe cu un underscore `_` sau folosind `..` pentru a omite părțile rămase ale unei valori. Vom explora cum și de ce să utilizăm fiecare dintre aceste tehnici de pattern-uri.

#### Ignorarea completă a valorii cu `_`

Am folosit simbolul `_` (underscore) ca un pattern wildcard care se potrivește cu orice valoare, fără a se lega de ea. Aceast pattern este util în special ca ultima ramură a unei expresii `match`, dar `_` poate fi folosit și în orice alt tip de pattern, inclusiv în parametrii funcțiilor, cum este ilustrat în Listarea 18-17.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-17/src/main.rs}}
```

<span class="caption">Listarea 18-17: Utilizarea `_` în semnătura unei funcții</span>

Codul de mai sus va ignora în totalitate valoarea `3` transmisă ca prim argument și va afișa `This code only uses the y parameter: 4`.

De obicei, atunci când nu mai este nevoie de un parametru specific într-o funcție, semnătura acesteia se modifică pentru a nu include parametrul respectiv. Totuși, ignorarea unui parametru al funcției este deosebit de utilă în situații când, de exemplu, implementezi o trăsătură care necesită o semnătură tipică, însă corpul funcției din implementarea ta nu are nevoie de unul dintre parametri. Acest lucru te ajută să eviți avertismentele de la compilator despre parametrii nefolosiți, care ar apărea dacă ai folosi un nume pentru parametru.

#### Ignorarea părților specifice ale unei Valori cu `_` imbricat

Putem utiliza `_` și în interiorul altor pattern-uri pentru a ignora anumite părți ale unei valori, de exemplu, când dorim să ne concentrăm doar pe o componentă specifică a valorii și nu avem nevoie de restul ei în codul pe care dorim să-l executăm. Listarea 18-18 prezintă un cod responsabil de gestionarea valorii unei setări. Cerințele funcționale impun ca un utilizator să nu poată rescrie o personalizare existentă a setării, dar îi permite să reseteze setarea și să-i atribuie o valoare dacă în prezent este neconfigurată.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-18/src/main.rs:here}}
```

<span class="caption">Listarea 18-18: Utilizarea `_` în cadrul pattern-urilor care potrivesc variantele `Some` și când nu este necesar să folosim valoarea conținută în `Some`</span>

Codul va afișa mesajele `Can't overwrite an existing customized value`  și după aceea `setting is Some(5)`. În prima ramură a expresiei `match`, nu avem nevoie de potrivirea sau utilizarea valorilor din variantele `Some`, însă trebuie să detectăm cazul în care `setting_value` și `new_setting_value` sunt de tip `Some`. În această situație, explicăm de ce nu se schimbă valoarea `setting_value`, care rămâne neschimbată.

În toate celelalte cazuri (când `setting_value` sau `new_setting_value` sunt `None`), exprimate prin pattern-ul `_` din cea de-a doua ramură, intenționăm să permitem ca `new_setting_value` să înlocuiască `setting_value`.

De asemenea, putem folosi `_` în diferite părți ale unui singur pattern pentru a ignora valori specifice. Listarea 18-19 demonstrează cum se ignoră valorile de pe poziția a doua și a patra într-o tuplă de cinci elemente.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-19/src/main.rs:here}}
```

<span class="caption">Listarea 18-19: Ignorarea mai multor componente ale unei tuple</span>

Codul va afișa `Some numbers: 2, 8, 32`, ignorând valorile 4 și 16.

#### Ignorarea unei variabile neutilizate prin începerea numelui acesteia cu `_`

Dacă definim o variabilă dar nu o folosim, Rust va emite de obicei un avertisment, pentru că o variabilă neutilizată poate indica prezența unui bug. Totuși, există momente când este util să definim o variabilă care nu va fi utilizată imediat, cum ar fi în timpul dezvoltării unui prototip sau la începutul unui proiect. În aceste cazuri, avem posibilitatea de a instrui Rust să nu emită avertismentul pentru variabila neutilizată dând variabilei un nume care începe cu un underscore. În Listarea 18-20, introducem două variabile neutilizate, dar când compilăm codul, ar trebui să primim avertisment doar pentru una dintre ele.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-20/src/main.rs}}
```

<span class="caption">Listarea 18-20: Începerea numelui unei variabile cu underscore pentru a evita avertismentele pentru variabile neutilizate</span>

Primim un avertisment legat de neutilizarea variabilei `y`, dar nu primim niciun avertisment pentru neutilizarea variabilei `_x`.

Este crucial să înțelegem că există o diferență fină între utilizarea doar a `_` și a unui nume care începe cu un underscore. Sintaxa `_x` încă asociază valoarea cu variabila, pe când `_` nu face nici o asociere. Pentru exemplificare, în Listarea 18-21 vom vedea că acest aspect face o diferență semnificativă.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-21/src/main.rs:here}}
```

<span class="caption">Listarea 18-21: O variabilă neutilizată începând cu un underscore tot asociază valoarea, ceea ce poate duce la transferul posesiunii valorii</span>

Vom întâlni o eroare deoarece valoarea `s` va fi transferată în variabila `_s`, preîntâmpinând astfel reutilizarea lui `s`. Pe de altă parte, utilizarea unui simplu underscore (`_`) nu va realiza nicio asociere. Conform Listării 18-22, codul va compila fără erori deoarece `s` nu este permutat.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-22/src/main.rs:here}}
```

<span class="caption">Listarea 18-22: Folosirea underscore-ului nu asociază valoarea</span>

Acest cod este funcțional pentru că `s` nu este legat de o altă entitate și, prin urmare, nu este permutat.

#### Ignorarea părților neutilizate ale unei valori cu `..`

Atunci când lucrăm cu structuri sau tuple care includ multiple elemente, este posibil să folosim sintaxa `..` pentru a selecta anumite componente și pentru a execluda restul, evitând astfel necesitatea de a insera underscore pentru fiecare element ignorat. Pattern-ul `..` ignoră acele părți ale unei valori care nu au fost explicit potrivite în restul pattern-ului. De exemplu, în listarea 18-23, avem o structură numită `Point` care conține o coordonată în spațiul tridimensional. În expresia `match`, ne dorim să acționăm doar asupra coordonatei `x`, ignorând valorile din `y` și `z`.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-23/src/main.rs:here}}
```

<span class="caption">Listarea 18-23: Ignorarea tuturor câmpurilor unui `Point`, în afara de `x`, utilizând `..`</span>

Pentru a face acest lucru, listăm valoarea pentru `x` și apoi includem pattern-ul `..`. Acest lucru este mult mai eficient decât să fie necesar să specificăm `y: _` și `z: _`, și este deosebit de util în cazul structurilor cu multiple câmpuri, când doar unul sau două sunt relevante într-un anumit context.

Sintaxa `..` se extinde automat pentru a acoperi numărul necesar de valori. Listarea 18-24 ilustrează cum `..` poate fi folosit în cazul unui tuple.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-24/src/main.rs}}
```

<span class="caption">Listarea 18-24: Potrivirea primei și ultimei valori dintr-o tuplă și ignorarea tuturor celorlalte valori intermediare</span>

În codul respectiv, valorile pentru `first` și `last` sunt potrivite, în timp ce `..` se ocupă de ignorarea celorlalte valori intermediare.

Totuși, aplicarea lui `..` trebuie să fie clară și lipsită de ambiguitate. Dacă nu este evident care valori sunt destinate potrivirii și care sunt de omis, compilatorul Rust va raporta o eroare. Listarea 18-25 ne arată un caz de utilizare ambiguă a lui `..`, care nu va permite compilarea codului respectiv.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-25/src/main.rs}}
```

<span class="caption">Listarea 18-25: Tentativa de utilizare ambiguă a sintaxei `..`</span>

Atunci când încercăm să compilăm acest exemplu, ne vom confrunta cu următoarea eroare:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-25/output.txt}}
```

Este imposibil pentru Rust să determine câte valori să ignore în tuplă înainte de potrivirea cu variabila `second` și cât de multe să fie neglijate după aceea. Codul ar putea indica dorința de a ignora valorile `2`, de a asocia variabila `second` cu `4` și apoi de a ignora valorile `8`, `16` și `32`; sau de a ignora valorile `2` și `4`, apoi de a asocia `second` cu `8` și a ignora `16` și `32`; și alte interpretări similare. Numele de variabilă `second` nu conferă nicio indicație specială pentru Rust, motiv pentru care ne confruntăm cu eroarea de compilare - folosirea lui `..` în mai multe locuri crează ambiguitate.

### Condiții adiționale cu gărzi match

Un *gardă match* (match guard) este o condiție suplimentară de tip `if`, specificată după pattern-ul dintr-un braț al instrucțiunii `match`, care de asemenea trebuie să corespundă pentru selecția acelui braț. Gărzile match sunt deosebit de utile pentru a exprima concepte mai complexe decât permite un simplu pattern.

Condiția poate accesa variabilele definite în pattern. Listarea 18-26 ilustrează o instrucțiune `match` unde primul braț are pattern-ul `Some(x)` și încorporează o gardă match `if x % 2 == 0` (sentința va fi adevărată dacă numărul este par).

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-26/src/main.rs:here}}
```

<span class="caption">Listarea 18-26: Adăugarea unei gărzi match la un pattern</span>

De exemplu, acest cod va afișa `Numărul 4 este par`. Atunci când `num` este comparat cu pattern-ul din primul braț, potrivirea este confirmată, deoarece `Some(4)` corespunde cu `Some(x)`. Ulterior, garda evaluează dacă restul împărțirii lui `x` la 2 este zero, și fiind așa, primul braț este ales.

În situația în care `num` ar fi fost `Some(5)`, garda match din primul braț nu ar fi fost îndeplinită, întrucât restul împărțirii lui 5 la 2 este 1, diferit de zero. Rust ar continua cu evaluarea brațului secund, care ar fi corespondent deoarece nu prezintă o gardă match și astfel potrivește orice variantă `Some`.

Expresia condițională `if x % 2 == 0` nu poate fi integrată într-un pattern, așa că garda match ne permite să articulăm această logica. Partea negativă a acestei capacitați suplimentare de exprimare este că, atunci când sunt folosite expresiile că gărzi match, compilatorul nu mai verifică exhaustivitatea.

În Listarea 18-11, am menționat că putem folosi o gardă match pentru a soluționa problema umbririi pattern-urilor. Reamintim că am creat o variabilă nouă în cadrul pattern-ului din expresia `match`, în loc să utilizăm variabila din afara `match`-ului. Această variabilă nouă a făcut imposibilă testarea valorii variabilei externe. Listarea 18-27 ne arată cum se poate folosi o gardă match pentru a corecta această problemă.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-27/src/main.rs}}
```

<span class="caption">Listarea 18-27: Utilizarea unei gărzi match pentru a verifica egalitatea cu o variabilă externă</span>

Acum, codul va afișa `Cazul implicit, x = Some(5)`. Pattern-ul în a doua ramură a match-ului nu introduce o nouă variabilă `y`, care ar umbri `y`-ul extern, ce ne permite să folosim `y`-ul extern în garda match. În loc de `Some(y)` formulăm `Some(n)`. Aceasta inițializează o nouă variabilă `n` care nu creează umbrire, deoarece nu există o variabilă `n` în afara contextului `match`.

Garda match `if n == y` nu constituie un pattern, așadar nu introduce variabile noi. Acest `y` este chiar `y`-ul extern, nu un nou `y` umbrit, iar noi putem căuta o valoare care să fie egală cu `y`-ul extern simplu comparând `n` cu `y`.

De asemenea, putem utiliza operatorul *sau* `|` în garda match pentru a defini mai multe pattern-uri; condițiile gărzii match se vor aplica tuturor pattern-urilor. Listarea 18-28 demonstrează cum se aplică precedența atunci când combinăm un pattern ce folosește `|` cu o gardă match. Aspectul important din acest exemplu este că garda match `if y` este aplicabilă atât la `4`, `5`, cât și la `6`, chiar dacă ar părea că `if y` este relevant doar pentru `6`.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-28/src/main.rs:here}}
```

<span class="caption">Listarea 18-28: Combinarea mai multor pattern-uri cu o gardă match</span>

Condiția de match arată că ramura se potrivește doar dacă valoarea lui `x` este egală cu `4`, `5`, sau `6` și numai dacă `y` este `true`. Când codul este executat, pattern-ul din prima ramură corespunde deoarece `x` este `4`, dar garda match `if y` este falsă, astfel prima ramură nu este selectată. Codul avansează la a doua ramură, care este potrivită, iar programul afișează `no`. Acest lucru se întâmplă pentru că condiția `if` se aplică întregului pattern `4 | 5 | 6`, nu exclusiv ultimei valori `6`. Astfel, precedența unei gărzi match față de un pattern este următoarea:

```text
(4 | 5 | 6) if y => ...
```

nu:

```text
4 | 5 | (6 if y) => ...
```

După execuția codului comportamentul precedenței devine evident: dacă garda match s-ar fi aplicat doar la ultima valoare din secvența valorilor specificate prin operatorul `|`, ramura s-ar fi potrivit și programul ar fi afișat `yes`.

### Legătura cu `@`

Operatorul *at* `@` ne permite să inițializăm o variabilă care păstrează o valoare în același timp când verificăm acea valoare pentru o potrivire de pattern. În Listarea 18-29, intenționăm să testăm dacă un câmp `id` din `Message::Hello` se încadrează în diapazonul `3..=7`. Vrem și să legăm valoarea la variabila `id_variable` pentru a o putea folosi în codul asociat acestei ramuri a pattern-ului. Am putea să-i dăm acestei variabile numele `id`, ca și câmpul, dar pentru acest exemplu am ales un nume diferit.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-29/src/main.rs:here}}
```

<span class="caption">Listarea 18-29: Utilizând `@` pentru a lega o valoare de un pattern în timp ce de asemenea o verificăm</span>

Acest exemplu va afișa `Found an id in range: 5`. Prin specificarea `id_variable @` înainte de diapazonul `3..=7`, capturăm orice valoare care se potrivește cu diapazonul, dar o verificăm și că se încadrează în pattern-ul diapazonului.

În cea de-a doua ramură, unde pattern-ul conține doar un diapazon, codul asociat cu ramura nu are la dispoziție o variabilă ce conține valoarea efectivă a câmpului `id`. Valoarea câmpului `id` ar fi putut fi 10, 11 sau 12, însă codul corespunzător nu cunoaște care anume este. Astfel, codul pattern-ului nu poate folosi valoarea din câmpul `id`, deoarece nu am stocat valoarea `id` într-o variabilă.

În ultima ramură, unde avem definită o variabilă fără diapazon, valoarea ne este disponibilă pentru utilizare în codul ramurii, stocată într-o variabilă numită `id`. Acest lucru se datorează utilizării sintaxei prescurtate a câmpurilor structurii. Totuși, nu am efectuat nicio verificare a valorii câmpului `id` în această ramură, spre deosebire de primele două: orice valoare se potrivește cu acest pattern.

Utilizarea `@` ne oferă posibilitatea de a testa o valoare și de a o stoca într-o variabilă în cadrul aceluiași pattern.

## Sumar

Pattern-urile din Rust sunt extrem de utile în distingerea între diferitele tipuri de date. Atunci când sunt aplicate în contextul expresiilor `match`, Rust asigură că pattern-urile acoperă fiecare valoare posibilă - în caz contrar, programul nu va fi compilat. Implementarea pattern-urilor în instrucțiunile `let` și în parametrii funcțiilor rezultă în construcții mai eficiente, facilitând astfel destructurarea valorilor în componente mai mici concomitent asignându-le variabilelor. Putem crea pattern-uri de la cele mai simple la cele mai complexe, conform necesităților specifice.

În capitolul care urmează, penultimul al acestei cărți, ne vom aprofunda cunoștințele despre unele dintre aspectele avansate ce caracterizează diversele funcționalități Rust.
