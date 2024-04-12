## Tipul Slice

*Secționarea* (slicing) îți permite să referențiezi o secvență contiguă de elemente dintr-o colecție, fără a face referire la întreaga colecție. O secțiune este un fel de referință și, prin urmare, ea nu deține posesiunea.

Iată o problemă de programare interesantă: scrie o funcție care acceptă un string format din cuvinte separate de spații și care returnează primul cuvânt descoperit în acest string. Dacă funcția nu găsește vreun spațiu în string, atunci întregul string este un singur cuvânt, caz în care întregul string ar trebui returnat.

Să analizăm cum am formula semnătura acestei funcții fără a utiliza secționări, pentru a înțelege mai bine problema pe care secționarea o va soluționa:

```rust,ignore
fn first_word(s: &String) -> ?
```

Funcția `first_word` utilizează un parametru de tip `&String`. Nu ne interesează posesiunea asupra valorii, deci acesta este modul corect de a proceda. Însă, ce valoare ar trebui să returneze funcția? În momentul de față, nu dispunem de un mijloc prin care să ne referim la o *porțiune* a unui string. Totuși, o soluție ar fi să returnăm indexul la care se încheie cuvântul, index care este marcat de un spațiu. Să încercăm această abordare, așa cum este prezentată în Listarea 4-7.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

<span class="caption">Listare 4-7: Funcția `first_word` care returnează un
index de byte în cadrul parametrului de tip `String`</span>

Avem nevoie să parcurgem `String`-ul element cu element și să determinăm dacă
o valoare este un spațiu. Pentru aceasta, vom converti `String` într-un array de byte utilizând metoda `as_bytes`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

În pasul următor, vom crea un iterator care să parcurgă array-ul de bytes, folosind metoda `iter`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

Vom detalia mai mult despre iteratori în Capitolul 13. Până atunci, este important de ştiut că `iter` este o metodă care returnează fiecare element al unei colecții, iar `enumerate` este un înveliş peste rezultatul lui `iter`. Ea returnează fiecare element ca parte a unei tuple. Primul element al tuplei returnate de `enumerate` este indexul, iar al doilea este o referință la element. Acest lucru este mai practic decât să calculăm noi înșine indexul.

Faptul că metoda `enumerate` returnează o tuplă ne dă posibilitatea de a folosi anumite modele pentru a destrăma acea tuplă. Vom detalia mai mult despre aceste modele în Capitolul 6. În cadrul buclei `for`, folosim un model care plasează `i` pe postura de index în tuplă, și `&item` pentru unicul byte din tuplă. Din moment ce `.iter().enumerate()` ne furnizează o referință la element, folosim `&` în model.

În interiorul buclei `for`, căutăm byte-ul care semnifică spațiul, folosind sintaxa literală a byte-ului. Dacă găsim un spațiu, vom returna poziția acestuia. Dacă nu, returnăm lungimea string-ului prin utilizarea `s.len()`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

Avem acum o metodă prin care putem identifica indexul corespunzător sfârșitului primului cuvânt din string. Totuși, întâmpinăm o problemă. Valoarea pe care o returnăm este un `usize` care, luat izolat, este doar un număr cu semnificație în contextul `&String`. Altfel spus, dat fiind faptul că acesta este o valoare separată de `String`, nu putem garanta faptul că aceasta va rămâne validă în viitor. Ia în considerare programul din Listarea 4-8 care utilizează funcția `first_word` din Listarea 4-7.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

<span class="caption">Listarea 4-8: Stocarea rezultatului obținut în urma apelării funcției `first_word`, urmată de modificarea conținutului variabilei tip `String` </span>

Acest program este compilat fără nicio eroare și ar continua să funcționeze corect chiar și dacă am folosi `word`  după apelarea metodei `s.clear()`. Întrucât `word` nu este legat de starea lui `s` în nici un fel, `word` păstrează în continuare valoarea `5`. Am putea utiliza valoarea `5` cu variabila `s` pentru a încerca să extragem primul cuvânt, însă am avea de-a face cu un bug, deoarece conținutul lui `s` s-a modificat de la momentul stocării valorii `5` în `word`.

Preocuparea constantă legată de faptul că indexul din `word` ar putea să nu mai fie în sincronizare cu datele din `s` este anevoioasă și ne expune erorilor! Administrarea acestor indici devine și mai fragilă dacă am scrie o funcție `second_word`. Semnătura acesteia ar trebui să arate în felul următor:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Acum există doi indici pe care trebuie să-i urmărim: cel de început *și* cel de sfârșit. În plus, avem din ce în ce mai multe valori care rezultă din date procesate într-o anumită stare, dar nu se conectează cu acea stare. Suntem înconjurați de trei variabile nelegate care trebuie să rămână sincronizate.

Din fericire, Rust oferă o soluție eficientă la această problemă: secționarea string-urilor.

### Secțiuni de string-uri

O *secțiune de string* reprezintă o referință la o porțiune dintr-un `String`, având următoarea formă:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

`hello` nu este o referință la întregul `String`, ci la o secțiune din acesta, secțiune specificată prin adăugarea elementului `[0..5]`. Secțiunile sunt create utilizând un interval în cadrul parantezelor pătrate, prin specificarea formatului `[indexul_de_start..indexul_de_sfârșit]`, unde `indexul_de_start` reprezintă prima poziție din secțiune, iar `indexul_de_sfârșit` este cu o unitate mai mare decât ultima poziție din secțiune. Intern, structura de date a secțiunii stochează poziția de start și lungimea secțiunii, lungime care corespunde cu diferența dintre `indexul_de_sfârșit` și `indexul_de_start`. Astfel, în cazul `let world = &s[6..11];`, `world` este o secțiune care conține un pointer către octetul din poziția 6 din `s`, având lungimea de `5`.

Figura 4-6 ilustrează clar acest aspect.

<img alt="Trei tabele: primul tabel reprezintă datele de pe stivă ale lui s, care indică octetul de la indexul 0 din tabelul ce conține datele string-ului &quot;hello world&quot; aflat pe heap. Al treilea tabel reprezintă datele de pe stivă ale secțiunii world, care au lungimea de 5 și indică octetul 6 din tabelul cu datele de pe heap."
src="img/trpl04-06.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-6: Secțiune de string ce face referire la o parte a unui `String`</span>

Folosind sintaxa specifică Rust pentru diapazoane, `..`, dacă vrei să începi de la indexul 0, poți omite valoarea înaintea celor două puncte. Altfel spus, cele două exemple de mai jos sunt echivalente:

```rust
let s = String::from("salut");

let slice = &s[0..2];
let slice = &s[..2];
```

La fel, dacă secțiunea ta include ultimul byte al string-ului, poți omite numărul de la final. Prin urmare, următoarele două exemple sunt echivalente:

```rust
let s = String::from("salut");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

De asemenea, poți renunța la ambele valori pentru a prelua o secțiune din întregul string. Acest fapt înseamnă că următoarele două exemple sunt identice:

```rust
let s = String::from("salut");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Notă: Indicii de interval pentru secțiunea de string-uri trebuie să
> corespundă la limitele caracterelor UTF-8 valide. Dacă încerci să creezi o
> secțiune de string în mijlocul unui caracter multi-byte, programul tău va
> ieși cu o eroare. Pentru a introduce conceptul de secțiuni de string-uri,
> presupunem că utilizăm doar ASCII în această secțiune. O discuție mai
> detaliată despre manipularea UTF-8 se găsește în secțiunea
> [„Stocarea textului codificat UTF-8 cu String-uri”][strings]<!-- ignore -->
> din Capitolul 8.

Cu toate aceste informații în minte, să rescriem funcția `first_word` pentru a returna o secțiune. Tipul ce denotă "secțiunea de string" se scrie ca `&str`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

Obținem indexul corespunzător sfârșitului cuvântului într-un mod similar cu cel prezentat în Listarea 4-7, prin identificarea primei apariții a unui spațiu. În momentul în care identificăm un spațiu, returnăm o secțiune din string folosind punctul de început al string-ului și indexul spațiului ca indici de început și sfârșit.

Astfel, la apelul funcției `first_word`, primim o singură valoare, care este strâns legată de datele inițiale. Această valoare este constituită dintr-o referință către punctul de start al secțiunii și numărul de elemente ce se regăsesc în secțiune.

Similar, returnarea unei secțiuni ar funcționa și în cazul unei funcții numite `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Acum dispunem de o interfață API simplificată, al cărei utilizare eronată este semnificativ redusă, deoarece compilatorul se va asigura că referințele din `String` rămân valide. Îți amintești de erorile din programul prezentat în Listarea 4-8? Acolo, am obținut indexul care indica sfârșitul primului cuvânt, dar ulterior am golit string-ul, ceea ce a făcut ca indexul nostru să devină inutilizabil. Deși acest cod era logic incorect, nu am întâmpinat nicio eroare imediată. Problemele ar fi devenit vizibile mai târziu, dacă am fi continuat să folosim indexul primului cuvânt cu un string gol. Utilizarea secțiunilor face această eroare imposibilă și ne avertizează mult mai devreme cu privire la problemele din codul nostru. Folosind versiunea de secțiune a `first_word`, vom întâmpina o eroare în timpul compilării:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

Avem următoarea eroare de compilare:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Luați în considerare regulile de împrumutare: dacă avem o referință imutabilă la un anumit element, nu putem obține, în același timp, și o referință mutabilă la acel element. Funcția `clear` are nevoie să trunchieze o variabilă de tip `String`, deci trebuie să obțină o referință mutabilă la aceasta. Funcția `println!`, care urmează după apelul la `clear`, utilizează referința la variabila `word`. Deci, referința imutabilă utilizată de `println!` trebuie să fie încă activă la momentul acela. Cu toate acestea, limbajul Rust nu permite ca o referință mutabilă (utilizată în `clear`) și o referință imutabilă (utilizată în `word`) să existe simultan, fapt care conduce la eșecul compilării. Acesta este un exemplu de cum Rust nu doar că face API-ul nostru mai ușor de utilizat, dar elimină eficient și o întreagă clasă de erori chiar în timpul compilării!

<!-- Old heading. Do not remove or links may break. -->
<a id="string-literals-are-slices"></a>

#### Literalii de string ca secțiuni

Vă reamintim că am discutat anterior despre modul în care literalii de tip string sunt stocați în interiorul codului binar. Acum, având cunoștințe despre secțiuni, putem înțelege într-un mod mai profund literalii de tip string:

```rust
let s = "Salut, lume!";
```

Aici, tipul variabilei `s` este `&str`: reprezintă o secțiune care indică un anumit punct specific în codul binar. Acesta este, de asemenea, motivul pentru care literalii de tip string sunt imutabili; `&str` este o referință imutabilă.

#### Utilizarea secțiunilor de string ca parametri

Înțelegând faptul că putem prelua secțiuni din literale și valori `String`, acest lucru ne permite să facem o îmbunătățire suplimentară a funcției `first_word`, însemnând în special modificarea modului în care aceasta este semnată:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Un programator Rust mai experimentat ar opta pentru semnătura de tip prezentată în Listarea 4-9, deoarece aceasta ne permite să utilizăm aceeași funcție atât pentru valorile `&String`, cât și pentru cele `&str`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

<span class="caption">Listare 4-9: Optimizarea funcției `first_word` prin folosirea unei secțiuni de string ca tip pentru parametrul `s`</span>

Dacă dispunem de o secțiune de string, o putem transmite direct. Dacă avem un `String`, putem transmite fie o secțiune a `String`-ului, fie o referință la `String`. Această versatilitate profită de funcționalitatea *deref coercions*, un aspect pe care îl vom explora în secțiunea [„Coerciții Deref implicite cu funcții și metode”][deref-coercions] din Capitolul 15.

Definind o funcție care să preia o secțiune de string în locul unei referințe la un `String`, API-ul nostru devine mai general și util, fără a compromite orice altă funcționalitate:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

### Alte tipuri de secțiuni

Secțiunile de string-uri, așa cum probabil ți-ai imaginat deja, sunt specifice pentru string-uri. Totuși, există și un tip de secțiune cu o aplicabilitate mai largă. Gândește-te la următorul array:

```rust
let a = [1, 2, 3, 4, 5];
```

La fel cum am dori să facem referire la o anumită porțiune dintr-un string, ne-ar putea interesa să ne referim la o anumită parte a unui array. Am face-o astfel:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```


Această secțiune are tipul `&[i32]`. Funcționează exact la fel ca secțiunile de string-uri, prin stocarea unei referințe la primul element și a lungimii acestuia. Vei utiliza acest tip de secțiune pentru o multitudine de alte categorii de colecții. Vom adresa aceste colecții în mod detaliat când vom discuta despre vectori în Capitolul 8.

## Sumar

Conceptele de posesiune, împrumutare și utilizarea secțiunilor garantează siguranța memoriei în programele Rust la momentul compilării. Limbajul Rust îți oferă autonomie în gestionarea memoriei, asemenea altor limbaje de programare de sistem. Avantajul remarcabil al Rust constă în faptul că deținătorul datelor efectuează automat o curățenie a acestora atunci când iese din domeniul de vizibilitate. Astfel, nu este nevoie de scrierea și depanarea de cod adițional.

Posesiunea influențează modul în care funcționează multe alte componente ale limbajului Rust; de aceea, ne vom aprofunda în discutarea acestor concepte pe tot parcursul cărții. Să mergem mai departe la Capitolul 5, unde vom examina gruparea datelor în cadrul unei `structuri`.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods
