## Cum să scriem teste

Testele sunt funcții Rust care verifică dacă codul non-test funcționează așa cum este așteptat. Corpurile funcțiilor de test îndeplinesc, de regulă, următoarele trei acțiuni:

1. Inițializarea datelor sau a stării necesare.
2. Executarea codului pe care dorești să-l testezi.
3. Confirmarea că rezultatele sunt cele așteptate.

Să examinăm caracteristicile specifice pe care Rust le pune la dispoziție pentru scrierea testelor care realizează aceste acțiuni. Printre acestea se numără atributul `test`, câteva macro-uri, precum și atributul `should_panic`.

### Anatomia unei funcții de test

În cea mai simplă formă, un test în Rust este o funcție adnotată cu atributul `test`. Atributele sunt meta-date despre porțiuni de cod Rust; un exemplu este atributul `derive` pe care l-am folosit atunci când lucram cu structurile în Capitolul 5. Ca să schimbi o funcție obișnuită într-o funcție de test, trebuie să adaugi `#[test]` pe linia deasupra lui `fn`. Atunci când executăm testele utilizând comanda `cargo test`, Rust compilează un binar runner de test care rulează funcțiile adnotate și raportează dacă fiecare dintre funcțiile de test a trecut sau nu.

Când începem un proiect nou de bibliotecă cu Cargo, un modul de test cu o funcție de test înăuntru ni se generează automat. Acest modul ne oferă un șablon pentru scrierea testelor, astfel încât să nu fie nevoie să căutăm structura exactă și sintaxa de fiecare dată când începem un proiect nou. Putem adăuga oricâte funcții de testare și module de testare dorim!

Vom investiga unele aspecte ale modului în care funcționează testele experimentând cu șablonul de test înainte de a testa efectiv orice cod. Apoi vom scrie teste concrete care vor apela codul pe care l-am scris și vom asigura că comportamentul său este cel corect.

Să creăm un nou proiect de bibliotecă denumit `adder` care va aduna două numere:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

Conținutul fișierului *src/lib.rs* din biblioteca dumneavoastră `adder` ar trebui să arate ca în Listarea 11-1.

<span class="filename">Numele fișierului: src/lib.rs</span>

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
cargo test
git co output.txt
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

<span class="caption">Listarea 11-1: Modulul de test și funcția generate automat de `cargo new`</span>

Pentru moment, să ignorăm primele două linii și să ne concentrăm pe funcția în sine. Notați adnotarea `#[test]`: acest atribut indică faptul că este o funcție de test, astfel încât runner-ul de test să știe să trateze această funcție ca pe un test. Ar putea exista și funcții care nu sunt de test în modulul `tests` pentru a aranja scenarii comune sau pentru a efectua operații periodice, prin urmare trebuie să indicăm întotdeauna care funcții sunt teste.

Corpul funcției exemplu folosește macro-ul `assert_eq!` pentru a afirma că `result`, care conține rezultatul adunării lui 2 cu 2, este egal cu 4. Aceasta afirmație servește ca un exemplu al formatului pentru un test tipic. Să o rulăm, pentru a vedea că acest test este valid.

Comanda `cargo test` efectuează rularea tuturor testelor din cadrul proiectului nostru, așa cum vedem în Listarea 11-2.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

<span class="caption">Listarea 11-2: Afişarea rezultatului testului generat automat</span>

Cargo a compilat și a executat testul cu succes. Ne este afișată linia `running 1 test`, urmând ca apoi să fie prezentat numele funcției de testare generate, `it_works`, și că rezultatul acestui test este `ok`. Rezumatul care indică `test result: ok.` confirmă faptul că toate testele au trecut, iar detaliile `1 passed; 0 failed` sumarizează numărul testelor care au avut succes și al celor care au eșuat.

Există posibilitatea ca un test să fie marcat ca ignorat, pentru a evita rularea lui în anumite condiții; acest lucru va fi discutat în secțiunea [“Ignorarea unor teste la cerere”][ignoring]<!-- ignore --> mai încolo în acest capitol. Deoarece nu am procedat așa în cazul de față, sumarul indică `0 ignored`. De asemenea, putem utiliza un argument în comanda `cargo test` pentru a rula doar testele care corespund unui anumit șir de caractere, un proces cunoscut sub numele de *filtrare*, care va fi explicat în secțiunea [“Rularea selectivă a testelor după nume”][subset]<!-- ignore -->. În exemplul nostru nu s-a realizat filtrarea, așadar sumarul indică `0 filtered out`.

Indicația `0 measured` se referă la teste de tip benchmark care măsoară performanța și, momentan, sunt disponibile doar în Nightly Rust. Pentru mai multe detalii, consultați [Documentația testelor de benchmark][bench].

Partea următoare a rezultatelor testelor, începând cu `Doc-tests adder`, reprezintă rezultatele testelor realizate pe documentație. Deși nu am creat încă teste de documentație, Rust permite compilarea oricărui cod exemplu prezent în documentația API-ului. Aceasta garantează menținerea sincronizării între documentație și cod! Modalitatea de redactare a testelor de documentație va fi discutată în secțiunea [“Testele din comentariile de documentație”][doc-comments]<!-- ignore --> din Capitolul 14. Momentan, vom lăsa deoparte ieșirea lui `Doc-tests`.

Să adaptăm testul astfel încât să corespundă specificațiilor dorite de noi. Începem prin a redenumi funcția `it_works` în ceva mai sugerativ, cum ar fi `exploration`, în felul următor:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Rulează comanda `cargo test` încă o dată. Rezultatul va afișa acum `exploration` în loc de `it_works`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Adăugăm un test suplimentar, dar intenționat să eșueze. Un test este marcat ca fiind nereușit atunci când o parte a funcției de test dezlănțuie panică. Testele sunt executate fiecare într-un fir de execuție propriu, iar dacă firul principal identifică prăbușirea unui astfel de fir, testul respectiv este declarat eșuat. Am învățat în Capitolul 9 că modalitatea cea mai directă de a declanșa panică este apelarea macro-ului `panic!`. Scrie noul test ca o funcție cu numele `another`, pentru ca fișierul tău *src/lib.rs* să fie conform cu Listarea 11-3.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs:here}}
```

<span class="caption">Listarea 11-3: Adăugarea unui test secund care va eșua prin invocarea macro-ului `panic!`</span>

Relansează testele folosind `cargo test`. Output-ul ar trebui să se alinieze cu Listarea 11-4, arătând reușita testului `exploration` și eșecul testului `another`.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

<span class="caption">Listarea 11-4: Rezultatele testelor când unul trece și celălalt eșuează</span>

Linia pentru `test tests::another` nu mai indică `ok`, ci `FAILED`. Apar două secțiuni noi între rezultatele individuale și sumar: prima detaliază motivele eșecului fiecărui test. În exemplul nostru, se prezintă că testul `another` a eșuat deoarece a `panicat la 'Make this test fail'` la linia 10 în fișierul *src/lib.rs*. Următoarea secțiune enumeră simplu numele testelor care au eșuat, util când avem multiple teste cu multe informații de eșec detaliate. Numele unui test eșuat poate fi folosit pentru a rula doar acel test, ușurând astfel depanarea; vom explora mai detaliat metodele de rulare a testelor în secțiunea [„Controlând cum sunt rulate testele”][controlling-how-tests-are-run]<!-- ignore -->.

În sumar, afișat la sfârșit, vedem că rezultatul global al testării este `FAILED`. Avem un test care a trecut și unul care a eșuat.

Având acum cunoștințe despre cum se prezintă rezultatele testelor în diverse condiții, să ne îndreptăm atenția către alte macro-uri, în afara de `panic!`, care sunt utilitare în cadrul testelor.

### Folosirea macro-ului `assert!` pentru a verifica rezultatele

Macro-ul `assert!`, oferit de biblioteca standard, este extrem de util pentru verificarea conformității unei condiții dintr-un test cu valoarea `true`. Oferim macro-ului `assert!` un argument care produce un rezultat Boolean. Dacă rezultatul este `true`, testul continuă fără întreruperi și este considerat cu succes. În cazul unui rezultat `false`, `assert!` inițiază `panic!`, rezultând într-un test nereușit. Implementarea macro-ului `assert!` ne ajută să ne asigurăm că funcționalitățile codului nostru operează conform așteptărilor.

Reamintim structura `Rectangle` și funcția ei `can_hold` prezentate în Capitolul 5, Listarea 5-15, acum reîntâlnite în Listarea 11-5. Vom include acest cod în fișierul *src/lib.rs* și vom dezvolta teste ce folosesc macro-ul `assert!`.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs:here}}
```

<span class="caption">Listarea 11-5: Reutilizarea structurii `Rectangle` și a metodei `can_hold`</span>

Având în vedere că metoda `can_hold` generează un Boolean, aceasta este perfectă pentru testarea cu macro-ul `assert!`. În Listarea 11-6, vom scrie un test pentru `can_hold` prin crearea unei instanțe `Rectangle` cu dimensiunile 8 în lățime și 7 în înălțime, verificând astfel că poate găzdui o altă instanță `Rectangle` de dimensiuni 5 cu 1.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

<span class="caption">Listarea 11-6: Un test pentru `can_hold` ce verifică dacă un
dreptunghi de dimensiuni mai mari poate de fapt să includă un dreptunghi mai mic</span>

În modulul `tests` am adăugat linia `use super::*;`. Acest modul respectă regulile de vizibilitate prezentate în Capitolul 7, secțiunea privind [„Utilizarea căilor pentru a face referire la un element în structura de module”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->
. Fiind un modul intern, este necesar să importăm codul ce trebuie testat din modulul extern în sfera de vizibilitate a modulului intern. Folosim un glob pentru ca tot ce este definit în modulul extern să fie disponibil aici, în modulul `tests`.

Numele testului nostru este `larger_can_hold_smaller`. Am construit cele două obiecte `Rectangle` necesare testului. Apoi, am utilizat macro-ul `assert!` pentru a evalua `larger.can_hold(&smaller)`, o expresie care ar trebui să returneze `true`, semn că testul nostru este corect. Verificăm acum rezultatul!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Rezultatul este conform așteptărilor! Acum, să proiectăm un alt test, care presupune că un dreptunghi mai mic nu poate conține unul mai mare:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Deoarece rezultatul corect pentru funcția `can_hold` în acest caz este `false`, 
este necesar să negăm rezultatul înainte de a-l transmite macro-ului `assert!`. Astfel,
testul nostru va reuși dacă `can_hold` returnează `false`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Avem două teste care au trecut! Să vedem acum ce se întâmplă cu rezultatele testelor când
introducem un defect în cod. Vom modifica implementarea metodei `can_hold`
înlocuind semnul mai mare cu un semn mai mic în comparația lățimilor:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

Rularea testelor acum generează următoarele:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Testele noastre au depistat defectul! Având în vedere că `larger.width` este 8 și `smaller.width` este 5, comparația lățimilor în `can_hold` acum oferă rezultatul `false`: 8 nu este
mai mic decât 5.

### Verificarea egalității cu macro-urile `assert_eq!` și `assert_ne!`

Pentru a testa corectitudinea unei funcționalități, este des întâlnită verificarea egalității între rezultatul codului în execuție și valoarea pe care o anticipăm. Aceasta se poate efectua cu ajutorul macro-ului `assert!`, unde se furnizează o expresie cu operatorul `==`. Dat fiind că acest tip de test este frecvent, biblioteca standard include două macro-uri speciale — `assert_eq!` și `assert_ne!` — care facilitează testarea. Acestea compară două valori pentru a determina dacă sunt egale sau diferite. În cazul în care aserțiunea eșuează, macro-urile afișează valorile comparate, ajutându-ne să înțelegem cauza eșecului. Spre deosebire, `assert!` indică pur și simplu că a rezultat o valoare `false` pentru expresia `==`, fără a dezvălui valorile care au generat această concluzie.

În Listarea 11-7, am definit o funcție cu denumirea `add_two`, care adaugă `2` la parametrul primit. Testăm funcția prin intermediul macro-ului `assert_eq!`.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

<span class="caption">Listarea 11-7: Testarea funcției `add_two` cu ajutorul macro-ului `assert_eq!`</span>

Verificăm dacă testul se validează.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

Ca argument pentru `assert_eq!`, trimitem valoarea `4`, care corespunde rezultatului funcției `add_two(2)`. Linia reprezentativă pentru acest test este `test tests::it_adds_two ... ok`, iar termenul `ok` ne confirmă că testul a avut succes.

Pentru a vedea cum reacționează `assert_eq!` în caz de eșec, să inserăm o eroare în cod. Schimbăm funcția `add_two` astfel încât acum să adauge `3`, nu `2`:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Repornim testele:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Testul nostru a depistat defectul! Testul `it_adds_two` a eșuat, iar mesajul indică faptul că aserțiunea care a dat greș a fost `` assertion failed: `(left == right)` `` și ne prezintă valorile pentru `left` și `right`. Această informație ne ajută să începem depanarea: argumentul `left` a fost `4`, dar argumentul `right`, unde aveam `add_two(2)`, era `5`. Imaginează-ți cât de util este acest lucru când derulăm multe teste concomitent.

E bine să știm că în anumite limbaje de programare și framework-uri de testare, parametrii funcțiunilor de aserțiune a egalității sunt denumiți `expected` și `actual`, iar ordinea în care sunt oferite argumentele e importantă. În Rust, totuși, aceștia se numesc `left` și `right`, și nu contează ordinea în care indicăm valoarea anticipată și valoarea produsă de cod. Aserțiunea din acest test ar putea fi exprimată de asemenea ca `assert_eq!(add_two(2), 4)`, ceea ce ar genera același mesaj de eșec, afișând `` assertion failed: `(left == right)` ``.

Macro-ul `assert_ne!` reușește dacă cele două valori pe care le oferim nu sunt identice și eșuează dacă sunt identice. Acest macro este deosebit de folositor în situațiile în care nu putem prezice ce valoare *va* rezulta, dar știm sigur ce valoare *nu ar trebui* să fie. Dacă, de exemplu, testăm o funcție care schimbă garantat intrarea într-un anume mod, însă modul exact de schimbare depinde de ziua în care rulăm testul, cel mai adecvat ar fi să afirmăm că rezultatul funcției nu e egal cu inputul.

La nivel intern, macro-urile `assert_eq!` și `assert_ne!` utilizează operatorii `==` și `!=`. Atunci când aserțiunile nu sunt valide, aceste macro-uri afișează argumentele folosind formatul de debug, presupunând că valorile comparate implementează trăsăturile `PartialEq` și `Debug`. Toate tipurile de date primitive și majoritatea tipurilor standard le implementează. Pentru structurile și enum-urile proprii, va fi necesar să implementați `PartialEq` pentru a testa egalitatea acestora și `Debug`, pentru a afișa valorile atunci când aserțiunea pică. Fiind trăsături care pot fi obținute prin derivare, după cum este explicat în Listarea 5-12 din Capitolul 5, de obicei aceasta se face simplu, prin adăugarea adnotării `#[derive(PartialEq, Debug)]` la definiția structurii sau a enum-ului. Pentru mai multe detalii despre aceste trăsături si altele derivabile, consultați Anexa C, [„Trăsături derivabile,”][derivable-traits]<!-- ignore -->.

### Adăugarea de mesaje de eroare particularizate

Este posibil să adăugați un mesaj personalizat care să fie afișat alături de mesajul de eșec, folosind argumente opționale în macro-urile `assert!`, `assert_eq!` și `assert_ne!`. Argumentele adiționale după cele obligatorii sunt înaintate macro-ului `format!` (abordat în Capitolul 8, în secțiunea [“Concatenarea utilizând operatorul `+` sau macro-ul `format!`"][concatenation-with-the--operator-or-the-format-macro]<!-- ignore -->), ceea ce înseamnă că puteți furniza un șir de formatare ce include locuri rezervate `{}` și valorile pentru completarea acestora. Mesajele personalizate sunt excelente pentru explicarea scopului unei afirmații și, atunci când un test nu este trecut, veți obține o înțelegere mai profundă a problemei cu care se confruntă codul.

De exemplu, presupunem că dispunem de o funcție ce emite o salutare personalizată și dorim să confirmăm că numele furnizat apare în ceea ce returnează funcția:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Cerințele programului nu au fost încă finalizate, și anticipăm că partea cu `Hello` din mesajul de salut va fi modificată. Am decis să evităm actualizarea testului de fiecare dată când cerințele se modifică, așa că în loc să căutăm o potrivire exactă cu valoarea întoarsă de funcția `greeting`, vom verifica doar dacă rezultatul include textul parametrului de intrare.

Introducem un defect în cod acum, schimbând funcția `greeting` pentru a nu mai include `name`, și vedem cum se prezintă eșecul testului fără un mesaj personalizat:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

Rularea acestui test oferă următoarele informații:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Acest rezultat doar ne informează că aserțiunea nu a reușit și pe ce linie se găsește. Un mesaj de eșec mai clarificator ar include valoarea returnată de funcția `greeting`. Adăugăm un mesaj personalizat ce include un șir de formatare cu un loc rezervat ce primește valoarea reală de la funcția `greeting`:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Acum, dacă executăm testul, vom obține un mesaj de eroare care ne oferă mai multe detalii:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

În acest mod, putem vedea clar valoarea reală primită în test, ce ne va ușura munca de depanare, informându-ne despre ce s-a petrecut în realitate, spre deosebire de ce ne așteptam să se întâmple.

### Verificarea de panică cu `should_panic`

Dincolo de a verifica valorile returnate, este esențial să ne asigurăm că tratăm corect condițiile de eroare în codul nostru. De pildă, să ne gândim la structura `Guess` definită în Capitolul 9, Listarea 9-13. Codul care implementează `Guess` se bazează pe premisa că instanțele de tip `Guess` vor include exclusiv valori între 1 și 100. De aceea, putem crea un test care să confirme că încercarea de a inițializa un obiect `Guess` cu o valoare din afara acestui interval va cauza o panică.

Procedăm adăugând atributul `should_panic` la funcția noastră de testare. Testul este considerat valid dacă se generează panică în codul funcției; eșuează în caz contrar.

Listarea 11-8 ilustrează un test care asigură că condițiile de eroare de la `Guess::new` se produc conform așteptărilor.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

<span class="caption">Listarea 11-8: Testarea unei condiții determinante pentru
generarea unei `panic!`</span>

Atributul `#[should_panic]` este așezat imediat după `#[test]` și înaintea funcției de test căreia i se aplică. Privim rezultatul când acest test își atinge scopul:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

Rezultatul este convingător! Acum, să provocăm intenționat o eroare înlăturând condiția ca funcția `new` să declanșeze o panică atunci când valoarea depășește 100:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

În momentul în care executăm testul din Listarea 11-8, observăm că va eșua:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}

Mesajul primit în această situație nu este foarte informativ, dar când inspectăm funcția de test, remarcăm că este marcată cu `#[should_panic]`. Înseamnă că testul a eșuat pentru că codul nu a provocat o panică.

Testele care folosesc `should_panic` pot fi neclare. Astfel de teste ar putea reuși chiar dacă panică apar din alte motive decât cele pe care le anticipam. Pentru a îmbunătăți precizia testelor cu `should_panic`, putem adăuga un parametru facultativ `expected` atributului `should_panic`. Aparatul de testare va confirma că mesajul de eroare conține textul furnizat. De pildă, să luăm în calcul codul modificat pentru `Guess` prezentat în Listarea 11-9, care ilustrează că funcția `new` declanșează panica cu mesaje diferite, bazat pe valoarea fiind prea mică sau prea mare.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

<span class="caption">Listarea 11-9: Testul unui `panic!` cu un mesaj ce conține un substring specificat</span>

Acest test va reuși fiindcă valoarea adăugată la parametrul `expected` din atributul `should_panic` se regăsește în mesajul emis la panicare de funcția `Guess::new`. Opțional, am fi putut alege să detaliem întreg mesajul la care ne așteptam, în acest caz fiind `Guess value must be less than or equal to 100, got 200.` Hotărâm ce mesaj dorim să includem în baza a cât de unic sau variabil este mesajul de panică și cât de exact dorim să fie testul. În situația de față, un substring al mesajului de panică este suficient pentru a verifica că în funcția de test s-a executat secțiunea `else if value > 100`.

Să vedem ce se întâmplă atunci când un test `should_panic` cu un mesaj `expected` nu trece. Vom crea din nou o eroare în codul nostru prin schimbarea locului conținuturilor blocurilor `if value < 1` cu `else if value > 100`:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Când executăm testul `should_panic`, acesta nu se desfășoară așa cum am dori:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

Mesajul de eroare arată că testul a cauzat o panică, cum era anticipat, însă mesajul de panică nu a inclus substring-ul dorit `'Guess value must be less than or equal to 100'`. În schimb, mesajul de panică primit a fost `Guess value must be greater than or equal to 1, got 200.` Acum, putem începe investigarea pentru identificarea sursei problemei din cod.

### Utilizarea `Result<T, E>` în teste

Până acum, testele create au generat panică atunci când s-au confruntat cu un eșec. O altă abordare este scrierea de teste care folosesc `Result<T, E>`. Ca exemplu, redau testul din Listarea 11-1, reconfigurat astfel încât să utilizeze `Result<T, E>` și să returneze `Err` în loc să declanșeze panică:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs}}
```

Acum, funcția `it_works` are tipul de retur `Result<(), String>`. În corpul acesteia, în loc să folosim macro-ul `assert_eq!`, vom returna `Ok(())` pentru un test reușit sau `Err` cu un `String` atașat în caz de eșec.

Folosirea `Result<T, E>` la scrierea testelor ne permite să aplicăm operatorul `?` direct în corpul acestora, oferind o metodă practică pentru redactarea testelor care trebuie să eșueze când orice operațiune internă returnează varianta `Err`.

Adnotarea `#[should_panic]` nu poate fi folosită în testele care returnează `Result<T, E>`. Dacă dorim să asigurăm că o operațiune specifică se finalizează cu o variantă `Err`, nu ar trebui să utilizăm operatorul `?` pentru valorile `Result<T, E>`, ci mai degrabă să validăm eșecul cu `assert!(value.is_err())`.

Având astfel mai multe metode de creare a testelor, este momentul oportun să înțelegem mai bine ce se întâmplă atunci când executăm testele și să explorăm opțiunile disponibile prin comanda `cargo test`.

[concatenation-with-the--operator-or-the-format-macro]:
ch08-02-strings.html#concatenation-with-the--operator-or-the-format-macro
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]:
ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
