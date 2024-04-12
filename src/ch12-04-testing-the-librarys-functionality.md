## Dezvoltarea funcționalității bibliotecii cu Test-Driven Development

După ce am separat logica în *src/lib.rs* și ne-am limitat la colectarea argumentelor și la gestionarea erorilor în *src/main.rs*, testarea funcționalității nucleului codului nostru s-a simplificat considerabil. Acum este posibil să apelăm funcții direct cu diferiți parametri și să verificăm valorile de retur fără a necesita execuția binarului din linia de comandă.

Vom începe această secțiune adăugând logica de căutare la programul `minigrep`, prin aplicarea principiilor test-driven development (TDD) și urmând pașii de mai jos:

1. Redactează un test care pică și execută-l pentru a confirma că eșuează din cauza anticipată.
2. Elaborează sau ajustează strict necesarul de cod pentru a asigura trecerea noului test.
3. Refactorizează codul pe care l-ai introdus sau modificat și verifică dacă testele rămân valide.
4. Reia procesul de la primul pas!

Deși reprezintă doar una dintre metodele de dezvoltare software, TDD joacă un rol cheie în structurarea codului. Abordarea de a scrie testul înaintea codului care va determina succesul acestuia contribuie la menținerea unei rate constante de acoperire a testelor de-a lungul întregului proces.

Urmează să testăm implementarea funcționalității care realizează căutarea expresiei de interogare în conținutul unui fișier, rezultând într-o listă cu liniile ce se potrivesc cu interogarea specificată. Vom aduce această capabilitate într-o funcție denumită `search`.

### Scrierea unui test ce eșuează

Pentru că nu ne mai sunt necesare, să eliminăm instrucțiunile `println!` din
*src/lib.rs* și *src/main.rs* pe care le-am folosit pentru a verifica comportamentul programului. Apoi, în *src/lib.rs*, adăugăm un modul `tests` cu o funcție de testare, așa cum am făcut în [Capitolul 11][ch11-anatomy]<!-- ignore -->. Funcția de testare specifică comportamentul pe care îl dorim de la funcția `search`: aceasta va prelua o interogare și textul de căutat și va returna doar liniile din text care includ interogarea. Listarea 12-15 prezintă acest test, care deocamdată nu poate fi compilat.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

<span class="caption">Listarea 12-15: Crearea unui test ce eșuează pentru funcția `search` pe care ne dorim s-o avem</span>

Testul acesta caută string-ul `"duct"`. Textul supus căutării este din trei rânduri, dintre care doar unul îl conține pe `"duct"` (De reținut că backslash-ul după ghilimeaua de început spune compilatorului Rust să nu includă un caracter de schimbare de linie la începutul conținutului literalului de tip string). Afirmăm că valoarea returnată de funcția `search` va conține exclusiv rândul pe care îl așteptăm.

În prezent, nu putem să rulăm acest test și să vedem că eșuează pentru că testul nici măcar nu se compilează: funcția `search` nu există încă! Respectând principiile TDD, vom adăuga strict atât cod cât este necesar pentru a face testul să se compileze și să ruleze, prin definirea unei funcții `search` care întoarce constant un vector gol, așa cum se arată în Listarea 12-16. Apoi, testul ar trebui să se compileze și să eșueze pentru că un vector gol nu se potrivește cu un vector ce conține rândul `"safe, fast, productive."`

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

<span class="caption">Lista 12-16: Definirea strict necesarului pentru funcția `search` astfel încât testul nostru să fie compilabil</span>

Este necesar să definim explicit durata de viață `'a` în semnatura funcției `search` și să folosim această durată de viață atât pentru argumentul `contents` cât și pentru valoarea de retur. În [Capitolul 10][ch10-lifetimes]<!-- ignore --> am învățat că parametrii duratei de viață stabilesc legătura dintre durata de viață a unui argument și durata de viață a valorii returnate. În acest exemplu, specificăm că vectorul returnat trebuie să includă secțiuni de string-uri care fac referire la secțiuni din argumentul `contents` (și nu din `query`).

Cu alte cuvinte, îi comunicăm lui Rust că datele returnate de funcția `search` vor exista pentru tot atâta timp cât există datele furnizate funcției `search` prin argumentul `contents`. Acest aspect este crucial! Datele la care se referă o secțiune trebuie să fie valide pentru ca referința să fie, de asemenea, validă; dacă compilatorul crede că generăm secțiuni de string-uri din `query` în loc de `contents`, verificarea de siguranță va fi realizată greșit.

Dacă uităm să adăugăm adnotările de durată de viață și încercăm să compilăm această funcție, vom întâmpina următoarea eroare:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust nu poate determina de unul singur care dintre cele două argumente este cel necesar, deci trebuie să îi specificăm în mod explicit. Deoarece `contents` este argumentul ce conține întregul nostru text și dorim să returnăm fragmentele care se potrivesc, putem deduce că `contents` este argumentul care trebuie legat de valoarea de retur prin utilizarea sintaxei pentru durate de viață.

În alte limbaje de programare nu este necesară asocierea argumentelor cu valorile de retur în cadrul semnăturii funcției, însă cu practică, acest proces va deveni mai simplu. Poate fi util să compari acest exemplu cu secțiunea [“Validând referințe cu timpuri de viață”][validating-references-with-lifetimes]<!-- ignore --> din Capitolul 10.

Să executăm acum testul:

```console
{{#include ../listings/ch12-an-io-project/listing-12-16/output.txt}}
```

Minunat, testul eșuează exact cum anticipam. Să facem acum testul să treacă!

### Scrierea codului pentru a trece testul

Testul nostru actual eșuează deoarece returnăm mereu un vector gol. Pentru a corecta asta și pentru a implementa funcția `search`, programul trebuie să urmeze acești pași:

* Parcurge fiecare linie din conținut.
* Verifică dacă linia conține string-ul nostru de căutare.
* Dacă îl conține, adaugă-l la lista valorilor pe care urmează să le returnăm.
* Dacă nu, nu întreprinde nici o acțiune.
* Întoarce lista rezultatelor care corespund criteriului de căutare.

Să abordăm fiecare pas, începând cu parcurgerea liniilor.

#### Parcurgerea liniilor cu metoda `lines`

Rust oferă o metodă utilă pentru a efectua iterația string-ului linie cu linie, numită în mod convenabil `lines`, care funcționează așa cum este arătat în Listarea 12-17. De notat că deocamdată acest cod nu va compila.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

<span class="caption">Listarea 12-17: Parcurgerea fiecărei linii din `contents` </span>

Metoda `lines` returnează un iterator. Vom discuta detalii despre iteratori în [Capitolul 13][ch13-iterators]<!-- ignore -->, însă îți amintești că ai întâlnit modul de utilizare a iteratorilor în [Listarea 3-5][ch3-iter]<!-- ignore -->, unde am folosit bucla `for` împreună cu un iterator pentru a rula codul pe fiecare element dintr-o colecție.

#### Căutarea interogării în fiecare linie

Acum, să verificăm dacă fiecare linie curentă conține string-ul căutat în interogarea noastră. Din fericire, string-urile dispun de o metodă foarte utilă denumită `contains` care realizează această verificare pentru noi! Adăugăm o apelare la `contains` în funcția `search`, așa cum este prezentat în Listarea 12-18. Nu uita că la acest moment codul încă nu va compila.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

<span class="caption">Listarea 12-18: Introducem funcționalitatea pentru a verifica dacă linia conține string-ul specificat în `query`</span>

În această etapă, ne concentrăm pe dezvoltarea funcționalităților. Pentru ca programul să compileze, trebuie să returnăm o valoare din corpul funcției, conform cu ceea ce am promis în semnătura funcției.

#### Salvarea liniilor potrivite

Pentru a completa această funcție, avem nevoie de un mod de a salva liniile care se potrivesc și pe care vrem să le returnăm. Pentru asta, putem iniția un vector mutabil înainte de bucla `for` și folosim metoda `push` pentru a adăuga o `line` în vector. După bucla `for`, returnăm vectorul, așa cum se arată în Listarea 12-19.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

<span class="caption">Listarea 12-19: Salvarea liniilor care se potrivesc pentru a le putea returna</span>

Acum, funcția `search` ar trebui să returneze numai liniile care includ `query`, și testul nostru ar trebui să treacă. Să încercăm să rulăm testul:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Testul nostru a trecut cu succes, deci suntem siguri că funcționează!

În acest moment, putem lua în considerare posibilități de refactoring pentru implementarea funcției de căutare, în timp ce ne asigurăm că testele rămân trecute pentru a păstra aceeași funcționalitate. Codul din funcția de `search` nu este rău, dar nu beneficiază de unele funcționalități utile ale iteratorilor. Ne vom reîntoarce la acest exemplu în [Capitolul 13][ch13-iterators]<!-- ignore -->, când vom explora pe larg iteratorii și vom vedea cum poate fi îmbunătățit.

#### Utilizând funcția `search` în funcția `run`

Acum că funcția `search` este funcțională și testată, trebuie să o apelăm din funcția `run`. Vom transmite valoarea `config.query` și conținutul pe care `run` îl extrage din fișier către funcția `search`. După aceea, `run` va afișa fiecare linie pe care `search` o returnează:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/src/lib.rs:here}}
```

Continuăm să utilizăm o buclă `for` pentru a prelua fiecare linie returnată de `search` și a o afișa. Acum, programul complet ar trebui să funcționeze! Să-l testăm, mai întâi cu un cuvânt care ar trebui să returneze o singură linie din poezia Emilyi Dickinson, "frog":

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Excelent! Acum să încercăm un cuvânt care va potrivi mai multe linii, precum "body":

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

Și în final, să verificăm că nu obținem nicio linie atunci când căutăm un cuvânt ce nu există în poezie, precum "monomorphization":

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Excelent! Ne-am creat propria variantă a unui instrument clasic și am învățat foarte multe despre structura aplicațiilor. Am aprofundat și noțiuni despre manipularea fișierelor, durata de viață, testare și analiza comenzilor introduse în linia de comandă.

Pentru a completa acest proiect, vom explica pe scurt cum să interacționăm cu variabilele de mediu și cum să scriem pe eroare standard, ambele fiind importante atunci când dezvoltați aplicații pentru linia de comandă.

[validating-references-with-lifetimes]:
ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
