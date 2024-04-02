## Controlul fluxului

Abilitatea de a rula unele coduri în funcție dacă o condiție este `true` și de a rula un cod în mod repetat dacă o condiție este `true` sunt blocuri de bază în majoritatea limbajelor de programare. Cele mai comune constructe care îți permit să controlezi fluxul de execuție al codului Rust sunt expresiile `if` și buclele.

### Expresiile `if`

O expresie `if` îți permite să amifici codul în funcție de condiții. Tu furnizezi o condiție și apoi afirmi: „Dacă această condiție este îndeplinită, rulează acest bloc de cod. Dacă condiția nu este îndeplinită, nu rula acest bloc de cod”.

Creează un proiect nou numit *branches* în directoriul tău *projects* pentru a explora expresia `if`. În fișierul *src/main.rs*, introdu următoarea linie:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

Toate expresiile `if` încep cu cuvântul cheie `if`, urmate de o condiție. În acest caz, condiția verifică dacă variabila `number` are o valoare mai mică de 5. Plasăm blocul de cod care urmează să se execute dacă condiția este `true` imediat după condiție între parantezele acolade. Blocurile de cod asociate cu condițiile din expresiile `if` sunt uneori numite *arms* (ramuri), la fel ca ramurile din expresiile `match` despre care am discutat în [secțiunea „Compararea supoziției cu numărul secret”][comparison-the supposition-the secret-number] a capitolului 2.

Opțional, putem include și o expresie `else`, pe care am ales să o facem aici, pentru a oferi programului un bloc alternativ de cod care să se execute în cazul în care condiția evaluatează la `false`. Dacă nu oferi o expresie `else` și condiția este `false`, programul va omite blocul `if` și va trece la următoarea parte de cod.

Încearcă să rulezi acest cod; ar trebui să vezi următoarea ieșire:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

Să încercăm să schimbăm valoarea lui `number` într-o valoare care face ca condiția să fie `false` pentru a vedea ce se întâmplă:

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

Rulează din nou programul și uită-te la ieșire:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

Este de asemenea important de menționat că în acest cod condiția *trebuie* să fie un `bool`. Dacă condiția nu este un `bool`, vom obține o eroare. De exemplu, încearcă să rulezi următorul cod:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

De această dată, condiția `if` evaluează valoarea `3` și Rust aruncă o eroare:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

Eroarea indică faptul că Rust se aștepta la un `bool`, dar a primit un integer. Spre deosebire de limbajele precum Ruby și JavaScript, Rust nu va încerca să convertească automat tipurile non-Boolean într-un Boolean. Trebuie să fii explicit și să furnizezi întotdeauna `if` cu un Boolean ca și condiție. Dacă vrem ca blocul de cod `if` să ruleze doar atunci când un număr nu este egal cu `0`, de exemplu, putem schimba expresia `if` astfel:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

Rulând acest cod se va afișa `numărul a fost altceva decât zero`.

#### Tratarea mai multor condiții cu `else if`

Poți utiliza mai multe condiții prin combinarea `if` și `else` într-o expresie `else if`. De exemplu:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

Acest program are patru căi posibile pe care le poate urma. După ce îl rulezi, ar trebui să vezi următoarea ieșire:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

Când acest program este executat, el verifică fiecare expresie `if` pe rând și execută primul bloc pentru care condiția se evaluează la `true`. Reține că, deși 6 este divizibil cu 2, nu vedem ieșirea `numărul este divizibil cu 2`, nici nu vedem textul `numărul nu este divizibil cu 4, 3 sau 2` din blocul `else`. Asta pentru că Rust execută doar blocul pentru prima condiție `true`, iar odată ce o găsește, nu mai verifică restul.

Folosirea prea multor expresii `else if` poate încărca codul tău, așa că dacă ai mai mult de una, s-ar putea să vrei să refactorizezi codul. Capitolul 6 descrie o structură avansată de instrucțiuni condiționale în Rust numită `match` pentru aceste cazuri.

#### Folosirea `if` într-o instrucțiune `let`

Deoarece `if` este o expresie, o putem folosi în partea dreaptă a unei instrucțiuni `let` pentru a atribui rezultatul unei variabile, așa cum se vede în Listarea 3-2.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

<span class="caption">Listarea 3-2: Atribuirea rezultatului unei expresii `if` unei variabile</span>

Variabila `number` va fi legată de o valoare în funcție de rezultatul expresiei `if`. Rulează acest cod pentru a vedea ce se întâmplă:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

Amintește-ți că blocurile de cod se evaluează până la ultima expresie din ele, iar numerele în sine sunt, de asemenea, expresii. În acest caz, valoarea întregii expresii `if` depinde de ce bloc de cod se execută. Acest lucru înseamnă că valorile care au potențialul de a fi rezultate din fiecare ramură a `if` trebuie să fie același tip; în Listarea 3-2, rezultatele atât ale ramurii `if`, cât și ale celei `else` erau numere întregi `i32`. Dacă tipurile nu se potrivesc, așa cum este cazul în următorul exemplu, vom primi o eroare:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

La compilarea acestui cod primim o eroare deoarece ramurile `if` și `else` au tipuri de valori incompatibile, și Rust indică exact unde să găsim problema în textul programului:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

Expresia în blocul `if` se evaluează ca un număr întreg, iar expresia din blocul `else` se evaluează ca un string. Acest lucru nu funcționează deoarece variabilele trebuie să aibă un singur tip, iar Rust trebuie să știe în timpul compilării care este tipul variabilei `number`, decisiv. Cunoașterea tipului `number` îi permite compilatorului să verifice dacă tipul este valid oriunde folosim `number`. Rust nu ar putea face asta dacă tipul `number` ar fi fost determinat doar la runtime; compilatorul ar fi fost mai complex sau ar fi făcut mai puține garanții despre cod, dacă ar fi trebuit să țină cont de mai multe tipuri ipotetice pentru orice variabilă.

### Repetarea cu bucle

Este adesea util să execuți un bloc de cod de mai multe ori. Pentru acest lucru, Rust oferă mai multe *bucle*, care vor rula prin codul din interiorul corpului buclei până la sfârșit și apoi vor începe imediat de la început. Pentru a experimenta cu buclele, să facem un nou proiect numit *loops*.

Rust are trei tipuri de bucle: `loop`, `while` și `for`. Să încercăm fiecare.

#### Repetarea codului cu `loop`

Cuvântul cheie `loop` indică Rust să execute un bloc de cod iar și iar pentru totdeauna sau până când îi spunem explicit să se oprească.

Ca exemplu, modifică fișierul *src/main.rs* din directoriul tău *loops* pentru a arăta în felul următor:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

Când rulăm acest program, vom vedea `din nou!` afișat iar și iar continuu până când oprim manual programul. Majoritatea terminalelor suportă comanda rapidă <span class="keystroke">ctrl-c</span> pentru a întrerupe un program care
este blocat într-o buclă continuă. Încearcă:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

Simbolul `^C` reprezintă locul în care ai apăsat <span class="keystroke">ctrl-c</span>. Este posibil să vezi sau nu cuvântul `din nou!` afișat după `^C`, în funcție de unde era codul în buclă când a primit semnalul de întrerupere.

Rust oferă de asemenea un mod prin care poți ieși dintr-o buclă folosind cod. Poți plasa cuvântul cheie `break` în interiorul buclei pentru a indica programului când să se oprească execuția buclei. Amintește-ți că am făcut asta în jocul de ghicire din secțiunea [“Oprirea după o ghicire corectă”][quitting-after-a-correct-guess]<!-- ignore --> din Capitolul 2 pentru a opri programul când utilizatorul a câștigat jocul prin ghicirea numărului corect.

Am folosit de asemenea `continue` în jocul de ghicire, care într-o buclă indică programul să sară peste orice cod rămas în această iterație a buclei și să treacă la următoarea iterație.

#### Returnarea valorilor din bucle

Una dintre utilizările unei instrucțiuni `loop` este de a reîncerca o operație despre care știi că ar putea eșua, cum ar fi verificarea dacă un thread și-a terminat treaba. Probabil că vei avea nevoie și de a trece rezultatul acelei operațiuni în afara buclei către restul codului tău. Pentru a face asta, poți adăuga valoarea pe care vrei să o returnezi după expresia `break` pe care o folosești pentru a opri bucla; acea valoare va fi returnată din bucla astfel încât să o poți utiliza, așa cum se arată aici:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

Înainte de buclă, noi declarăm o variabilă numită `counter` și o inițializăm la
`0`. Apoi declaram o variabilă numită `result` pentru a deține valoarea returnată din
buclă. La fiecare iterație a buclei, adăugăm `1` la variabila `counter`, și apoi verificăm dacă `counter` este egal cu `10`. Când este, folosim cuvântul cheie `break` cu valoarea `counter * 2`. După buclă, folosim un punct și virgulă pentru a încheia declarația care alocă valoarea pentru `result`. În final, noi afișăm valoarea în `result`, care în acest caz este `20`.

#### Etichete de buclă pentru deosebirea între bucle multiple

Dacă ai bucle în interioarele altei bucle, `break` și `continue` se aplică buclei interne din acel punct. Poți specifica opțional o *etichetă de buclă* pe o buclă pe care o poți folosi apoi cu `break` sau `continue` pentru a specifica că acele cuvinte cheie se aplică buclei etichetate în locul celei mai interne bucle. Etichetele de buclă trebuie să înceapă cu un singur apostrof. Iată un exemplu cu două bucle îmbricate:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

Bucla externă are eticheta `'counting_up`, și va număra în sus de la 0 la 2. Bucla internă fără o etichetă numără în jos de la 10 la 9. Primul `break` care nu specifică o etichetă va ieși doar din bucla internă. Declarația `break 'counting_up;` va ieși din bucla externă. Acest cod afișează:

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

#### Buclele condiționale cu `while`

Un program va avea frecvent nevoie să verifice o condiție în timpul unei bucle. Cât timp condiția rămâne `true`, bucla se execută. Când condiția nu mai este `true`, programul apelează `break`, ceea ce oprește bucla. Comportamentul acesta poate fi realizat printr-o combinație de `loop`, `if`, `else` și `break`; poți să încerci chiar acum acest lucru într-un program, dacă vrei. Cu toate acestea, acest pattern este atât de frecvent încât Rust include o construcție specială pentru acesta, numită buclă `while`. În Listarea 3-3, utilizăm `while` pentru a rula bucla de trei ori, numărând în jos de fiecare dată, și după aceea, în afara buclei, afișăm un mesaj și ieșim din program.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

<span class="caption">Listarea 3-3: Utilizarea unei bucle `while` pentru a rula cod atât timp cât o condiție este adevărată</span>

Această construcție reduce semnificativ imbricările care ar fi fost necesare dacă s-ar folosi `loop`, `if`, `else` și `break`, oferind în același timp o mai mare claritate. Atâta timp cât o condiție este evaluată ca fiind `true`, codul rulează; în caz contrar, acesta iese din buclă.

#### Parcurgerea unei colecții cu `for`

Ai opțiunea de a folosi structura `while` pentru a parcurge elementele unei colecții, cum ar fi un array. De exemplu, bucla din listarea 3-4 afișează fiecare element din array-ul `a`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

<span class="caption">Listarea 3-4: Parcurgerea fiecărui element al unei colecții
folosind o buclă `while`</span>

Aici, codul numără elementele din array. Începe de la indexul `0`, și apoi rulează până atinge indexul final din array (adică, când expresia `index < 5` nu mai este `true`). Rularea acestui cod va afișa fiecare element din array:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

Toate cele cinci valori ale array-ului apar în terminal, după cum și ne așteptam. Deși `index` va atinge la un moment dat valoarea `5`, bucla se oprește înainte de a încerca să extragă o a șasea valoare din array.

Cu toate acestea, această abordare este predispusă la erori; am putea provoca panică programului dacă valoarea indexului sau condiția de test este incorectă. De exemplu, dacă ai modificat definiția array-ului `a` pentru a avea patru elemente, dar ai uitat să actualizezi condiția la `while index < 4`, codul s-ar opri cu panică. De asemenea rularea este lentă, deoarece compilatorul adaugă cod de execuție pentru a efectua verificarea condițională a indexului în granițele array-ului la fiecare parcurgere a buclei.

O alternativă mai concisă ar fi o buclă `for` cu executarea unui cod pentru fiecare element din colecție. O buclă `for` arată ca în codul din listarea 3-5.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

<span class="caption">Listarea 3-5: Parcurgerea fiecărui element al unei colecții
folosind o buclă `for`</span>

Dacă vom rula acest cod vom vedea aceeași ieșire ca în listarea 3-4. Mai important, noi am îmbunătățit acum securitatea codului și am eliminat riscul de bug-uri care ar putea rezulta din depășirea sfârșitului array-ului sau din ne-parcurgerea integrală și omisiunea unor elemente.

Folosind bucla `for`, nu mai e nevoie să îți amintești să modifici orice alt cod dacă schimbi numărul de valori din array, cum ar fi cazul cu metoda folosită în listarea 3-4.

Siguranța și conciziunea buclelor `for` le fac cel mai des folosit concept de bucle în Rust. Chiar și în situații în care se dorește de rulat un anumit cod de un număr de ori, cum ar fi exemplul cu numărătoarea inversă care a folosit o buclă `while` în listarea 3-3, majoritatea programatorilor Rust ar folosi o buclă `for` utilizând un `Range` furnizat de biblioteca standard, care generează toate numerele în ordine începând de la un număr și terminând înainte de un alt număr.

Iată cum ar arăta numărătoarea inversă folosind o buclă `for` și o altă metodă despre care încă nu am vorbit, `rev`, pentru a inversa șirul:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

Acest cod pare mai elegant, nu-i așa?

## Sumar

Ai reușit! Acesta a fost un capitol considerabil: ai învățat despre variabile, tipuri de date scalare și compuse, funcții, comentarii, expresii `if` și bucle! Pentru a te antrena cu conceptele discutate în acest capitol, încearcă să scrii programe care să facă următoarele lucruri:

* Convertarea temperaturilor între Fahrenheit și Celsius.
* Generearea unui al *n*-lea număr Fibonacci.
* Imprimarea versurile „Un elefant se legăna”, profitând de repetiția din cântec.

Când ești gata să continui, vom discuta despre un concept din Rust care *nu* există în mod obișnuit în alte limbaje de programare: posesiunea.

[comparing-the-guess-to-the-secret-number]:
ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]:
ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
