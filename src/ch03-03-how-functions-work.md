## Funcții

Funcțiile sunt omniprezente în codul Rust. Ai văzut deja una dintre cele mai importante funcții din limbaj: funcția `main`, care este punctul de intrare în multe programe. Ai văzut deja și cuvântul cheie `fn`, care îți permite să declari funcții noi.

Codul Rust folosește *snake case* ca stil convențional pentru numele funcțiilor și variabilelor, în care toate literele sunt minuscule și cuvintele sunt separate de caracterele de underscore. Iată un program care conține o definiție exemplară a unei funcții:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Definim o funcție în Rust prin introducerea `fn` urmată de un nume de funcție și un set de paranteze. Parantezele acolade indică compilatorului unde începe și se termină corpul funcției.

Putem apela orice funcție pe care am definit-o introducând numele său urmat de un set de paranteze. Deoarece `another_function` este definită în program, ea poate fi apelată din interiorul funcției `main`. Observă că am definit `another_function` *după* funcția `main` în codul sursă; am fi putut să o definim și înainte. Rust nu îi pasă unde definim funcțiile noastre, doar că sunt definite într-un loc vizibil pentru codul apelant.

Să începem un nou proiect binar denumit *functions* pentru a continua explorarea funcțiilore. Pune exemplul `another_function` în *src/main.rs* și rulează-l. Ar trebui să vezi următoarea ieșire:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

Liniile se execută în ordinea în care apar în funcția `main`. Mai întâi mesajul „Salut, lume!” este tipărit, apoi `another_function` este apelată și mesajul său este tipărit.

### Parametri

Funcțiile pot fi definite cu *parametri*, aceștia fiind variabile speciale care fac parte din semnătura unei funcții. Când o funcție are parametri ea poate fi apelată cu valori concrete pentru acești parametri. Tehnic, aceste valori concrete se numesc *argumente*, dar în conversațiile de zi cu zi, oamenii tind să folosească termenii *parametru* și *argument* în mod interschimbabil atât pentru fiecare variabilă în definiția unei funcții, cât și pentru valorile concrete transmise când o funcție este apelată.

În această versiune a lui `another_function`, noi adăugăm un parametru:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

Încearcă să rulezi acest program; ar trebui să primești următoarea ieșire:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

Declarația `another_function` are un parametru numit `x`. Tipul lui `x` este specificat ca fiind `i32`. Când transmitem `5` la `another_function`, macro `println!` plasează `5` acolo unde perechea de acolade cu `x` era în șirul de caractere format.

În semnăturile funcțiilor, *trebuie* să declari tipul fiecărui parametru. Acest lucru este o decizie deliberată în designul Rust: cererea de adnotații de tip în definițiile funcțiilor înseamnă că compilatorul aproape niciodată nu are nevoie de ele în alte părți din cod pentru a afla ce tip ai vrut să spui. De asemenea, compilatorul este capabil să ofere mesaje de eroare mult mai utile dacă știe ce tipuri așteaptă funcția.

Când definești mai mulți parametri, separă declarațiile de parametri cu virgule, în acest fel:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

Acest exemplu creează o funcție numită `print_labeled_measurement` cu doi parametri. Primul parametru se numește `value` și este `i32`. Al doilea se numește `unit_label` și este de tip `char`. Funcția apoi afișează text care conține atât `value` cât și `unit_label`.

Să încercăm să rulăm acest cod: înlocuiește programul curent din fișierul *src/main.rs* al proiectului tău *functions* cu exemplul de mai sus și rulează-l folosind `cargo run`:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

Deoarece am apelat funcția cu `5` ca valoare pentru `value` și `'h'` ca valoare pentru `unit_label`, ieșirea programului conține anume aceste valori.

### Instrucțiuni și expresii

Corpurile funcțiilor sunt compuse dintr-o serie de instrucțiuni, care se pot încheia opțional cu o expresie. Până acum, funcțiile pe care le-am discutat nu au inclus o expresie finală, însă ai observat expresii utilizate ca parte a instrucțiunilor. Spre deosebire de multe alte limbaje de programare, Rust este un limbaj bazat pe expresii - o distincție importantă de înțeles. În continuare, să analizăm ce reprezintă instrucțiunile și expresiile și cum această diferențiere influențează structura corpurilor funcțiilor.

* **Instrucțiunile** sunt directive care realizează o anumită acțiune și nu returnează o valoare.
* **Expresiile** evaluează o valoare rezultantă. Să vedem câteva exemple.

De fapt, am folosit deja instrucțiuni și expresii. Crearea unei variabile și atribuirea unei valori cu un cuvânt cheie `let` este o instrucțiune. În listarea 3-1, `let y = 6;` este o instrucțiune.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

<span class="caption">Listarea 3-1: O declarație a funcției `main` conținând o instrucțiune</span>

Definițiile de funcții sunt, de asemenea, instrucțiuni; întregul exemplu precedent este o instrucțiune în sine.

Instrucțiunile nu returnează valori. Prin urmare, nu poți atribui o instrucțiune `let` unei alte variabile, așa cum încearcă să facă următorul cod; vei obține o eroare:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

Când rulezi acest program, eroarea pe care o obții arată așa:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

Instrucțiunea `let y = 6` nu returnează o valoare, deci nu există nimic de atribuit lui `x`. Acest lucru este diferit față de ceea ce se întâmplă în alte limbaje, cum ar fi C și Ruby, unde atribuirea returnează valoarea atribuirii. În acele limbaje, poți scrie `x = y = 6` și atât `x`, cât și `y` vor avea valoarea `6`; însă nu în cazul lui Rust.

Expresiile evaluează o valoare și alcătuiesc majoritatea celorlalte coduri pe care le vei scrie în Rust. Consideră o operație matematică, cum ar fi `5 + 6`, care este o expresie care evaluează la valoarea `11`. Expresiile pot face parte din instrucțiuni: în Listarea 3-1, `6` în instrucțiunea `let y = 6;` este o expresie care evaluează valoarea `6`. Apelarea unei funcții este o expresie. Apelarea unui macro este o expresie. Și un nou bloc de cod creat cu acolade este o expresie, spre exemplu:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

Expresia:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

este un bloc care, în acest caz, evaluează la `4`. În continuare această valoare este atribuită lui `y` ca parte a instrucțiunii `let`. Reține că linia `x + 1` nu are un punct și virgulă la sfârșit, spre deosebire de cele mai multe linii pe care le-ai văzut până acum. Cauza e că expresiile nu includ semicoalone la sfârșit. Dacă adaugi un punct și virgulă la sfârșitul unei expresii, o transformi într-o instrucțiune, și ea nu va mai returna o valoare. Ține acest lucru în minte pe măsură ce vom explora în continuare valorile de retur a funcțiilor și expresiile.

### Funcții cu valori de retur

Funcțiile pot întoarce valori codului care le apelează. Noi nu dăm un nume valorilor de retur, dar trebuie să le declarăm tipul după o săgeată (`->`). În Rust, valoarea de retur a funcției este sinonimă cu valoarea ultimei expresii din blocul corpului acelei funcții. Poți returna mai devreme dintr-o funcție folosind cuvântul cheie `return` și specificând o valoare, dar majoritatea funcțiilor returnează ultima expresie în mod implicit. Iată un exemplu de funcție care returnează o valoare:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

Nu sunt apeluri de funcții, macrouri sau chiar declarații `let` în funcția `five` - doar numărul `5` în sine. Aceasta este o funcție perfect valabilă în Rust. Observă că tipul de retur al funcției este specificat și el, ca `-> i32`. Încearcă să rulezi acest cod; rezultatul ar trebui să arate așa:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

Numărul `5` în `five` este valoarea de retur a funcției, motiv pentru care tipul de retur este `i32`. Să examinăm acest lucru mai detaliat. Sunt două aspecte importante: În primul rând, linia `let x = five();` arată că folosim valoarea de retur a unei funcții pentru a inițializa o variabilă. Deoarece funcția `five` returnează un `5`, această linie ar fi echivalentă cu următoarea:

```rust
let x = 5;
```

În al doilea rând, funcția `five` nu are parametri și definește tipul valorii de retur, dar corpul funcției este un singur `5`, fără punct și virgulă, pentru că este anume expresia a cărei valoare vrem să o returnăm.

Să vedem un alt exemplu:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

Rularea acestui cod va afișa `Valoarea lui x este: 6`. Dar dacă punem un punct și virgulă la finalul liniei care conține `x + 1`, schimbând-o dintr-o expresie într-o declarație, vom obține o eroare:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

Compilarea acestui cod produce o eroare, astfel:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

Mesajul principal de eroare, `mismatched types`, relevă problema de bază cu acest cod. Definiția funcției `plus_one` spune că va returna un `i32`, dar declarațiile nu evaluează la o valoare, ceea ce este exprimat prin `()`, tipul unit. Prin urmare, nu se returnează nimic, ceea ce contrazice definiția funcției și duce la o eroare. În această ieșire Rust chiar oferă un mesaj care ar putea ajuta la corectarea problemei: sugerează eliminarea punctului și virgulei, care într-adevăr ar remedia eroarea.
