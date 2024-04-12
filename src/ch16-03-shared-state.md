## Concurență cu stare partajată

Transmiterea mesajelor constituie o metodă eficientă de abordare a concurenței, dar nu este unica. O altă cale este ca mai multe fire de execuție să aibă acces simultan la aceleași date partajate. Să ne reamintim de sloganul din documentația limbajului Go: "nu comunicați prin partajarea memoriei".

Dar cum ar arăta comunicarea prin partajarea memoriei? Și de ce ar îndruma avocații transmiterii mesajelor împotriva utilizării partajării memoriei?

Fundamental, canalele în cadrul oricărui limbaj de programare sunt comparabile cu ideea de posesiune unică, în sensul că după ce trimiți o valoare printr-un canal, nu ar trebui să o mai folosești. În contrast, concurența cu memorie partajată se aseamănă cu posesiunea multiplă, permițând mai multor fire accesul concomitent la aceeași locație de memorie. Cum am observat în Capitolul 15, cu ajutorul pointerilor inteligenți, posesiunea multiplă implică o complexitate sporită, deoarece trebuie gestionată coexistența acestor proprietari multipli. Sistemul de tipuri din Rust și regulile sale de posesiune oferă un sprijin considerabil în obținerea unei gestionări corecte. Pentru a ilustra, să examinăm mutexurile, una dintre cele mai răspândite primitive pentru concurența bazată pe memorie partajată.

### Utilizarea unui mutex pentru acces exclusiv la date

*Mutex* reprezintă o abreviere pentru *excludere reciprocă*, adică un mutex permite accesul la date doar unui singur fir de execuție la un anumit moment. Pentru a accesa datele protejate de un mutex, un fir trebuie mai întâi să indice că își dorește accesul obținând *lock*-ul acestui mutex. Lock-ul este o structură de date care face parte din mutex și care monitorizează cine deține accesul exclusiv la date. Așadar, se spune că mutexul *protejează* datele pe care le conține folosind mecanismul de blocare.

Mutexurile sunt recunoscute pentru complexitatea lor în utilizare, de vreme ce trebuie să reținem două reguli importante:

* Trebuie să încerci să dobândești lock-ul înainte de a utiliza datele.
* Când ai terminat cu datele protejate de mutex, trebuie să eliberezi datele astfel încât alte fire să poată dobândi lock-ul.

Pentru o metaforă din viața reală a unui mutex, gândește-te la o dezbatere din cadrul unei conferințe care dispune de un singur microfon. Înainte ca un participant să poată interveni, trebuie să ceară sau să semnaleze că dorește să utilizeze microfonul. După ce a primit microfonul, poate să vorbească pentru perioada pe care o dorește și apoi să treacă microfonul următorului care a cerut să intervină. Dacă un participant uită să paseze microfonul când a terminat, nimeni altcineva nu va putea să vorbească. Dacă gestionarea microfonului partajat este defectuoasă, dezbaterea nu va decurge conform planificării!

Manevrarea mutexurilor poate fi un proces extrem de anevoios de stăpânit, ceea ce explică entuziasmul multora pentru canale. Totuși, datorită sistemului de tipuri și regulilor de posesiune din Rust, nu te poți încurca în operațiunile de blocare și deblocare.

#### API-ul lui `Mutex<T>`

Pentru a exemplifica utilizarea unui mutex, să demarăm prin incorporarea sa într-un context cu un singur fir de execuție, după cum vedem în Listarea 16-12:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

<span class="caption">Listarea 16-12: Explorarea API-ului `Mutex<T>` într-un context cu un singur fir, pentru simplificare</span>

Precum în cazul multor alte tipuri, instanțiem un `Mutex<T>` printr-o apelare la funcția asociată `new`. Ca să ajungem la datele din mutex, folosim metoda `lock` pentru a obține blocarea. Această operațiune va suspenda firul de execuție actual, fără a mai putea efectua alte sarcini până când îi vine rândul să preia controlul blocării.

Această apelare la `lock` ar da greș dacă un alt fir care are blocarea ar genera panică. În acest caz, blocarea n-ar mai putea fi preluată de nimeni, motiv pentru care am optat pentru `unwrap`, pentru a provoca panică în firul de execuție dacă ne confruntăm cu acest scenariu.

Odată blocarea obținută, putem considera valoarea returnată, pe care o numim `num` aici, ca pe o referință mutabilă către conținutul interior. Sistemul de tipuri Rust ne asigură că trebuie să avem blocarea înainte de a folosi valoarea din `m`. Fiindcă `m` este de tip `Mutex<i32>` și nu `i32`, este *imperativ* să invocăm `lock` pentru a putea opera cu valoarea `i32`. Datorită sistemului de tipuri nu ni se va permite să sărim peste acest pas.

Așa cum ai putea bănui, `Mutex<T>` funcționează ca un pointer inteligent. Mai precis, apelul la `lock` *returnează* un pointer inteligent denumit `MutexGuard`, ambalat într-un `LockResult` cu care ne-am ocupat prin `unwrap`. `MutexGuard` implementează `Deref` pentru a putea accesa datele interne, iar de asemenea dispune de implementarea `Drop`, care eliberează blocarea automat atunci când `MutexGuard` părăsește domeniul de vizibilitate, la finalul domeniului interior. Acest lucru înlătură riscul de a uita să eliberăm blocarea și de a împiedica folosirea mutexului de alte fire, pentru că eliberarea are loc automat.

După ce am eliberat blocarea, avem posibilitatea să afișăm valoarea mutex-ului și să constatăm că i-am modificat valoarea interioară `i32` la 6.

#### Partajarea unui `Mutex<T>` între fire multiple

Să încercăm să partajăm o valoare între mai multe fire de execuție utilizând `Mutex<T>`. Vom lansa 10 fire și le vom permite fiecăruia să crească un contor cu 1, astfel încât contorul să ajungă de la 0 la 10. Următorul exemplu din Listarea 16-13 va produce o eroare de compilare, eroare din care vom învăța mai multe despre cum să utilizăm `Mutex<T>` în mod corect.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

<span class="caption">Listarea 16-13: Zece fire fiecare incrementând un contor protejat prin intermediul unui `Mutex<T>`</span>

Începem prin a crea o variabilă `counter` ce conține un `i32` în cadrul unui `Mutex<T>`, așa cum am procedat în Listarea 16-12. Apoi, generăm 10 fire iterând printr-un interval de numere, utilizând `thread::spawn` și oferind tuturor firelor aceeași închidere: una care transferă contorul în interiorul firului de execuție, obține blocarea pe `Mutex<T>` invocând metoda `lock`, după care adaugă 1 la valoarea aflată în mutex. Când un fir termină de executat funcția lambda, `num` va ieși din domeniul de vizibilitate și va elibera blocarea, permițând altui fir să o preia.

În firul principal, colectăm toți descriptorii de join. Ca în Listarea 16-2, apelăm `join` pe fiecare descriptor, pentru a ne asigura că toate firele sunt finalizate. În acel moment firul principal va obține blocarea și va afișa rezultatul acestui program.

Anterior am menționat că acest exemplu nu va compila. Să descoperim acum motivul!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

Mesajul de eroare arată că valoarea `counter` a fost transferată în iterarea precedentă a buclei. Rust ne informează că nu avem posibilitatea de a transfera proprietatea `counter` în multiple fire de execuție. În continuare să corectăm eroarea de compilator cu o metodă de posesiune multiplă pe care am discutat-o în Capitolul 15.

#### Posesiunea multiplă cu fire multiple

În Capitolul 15, am oferit unei valori mai mulți proprietari utilizând pointerul inteligent `Rc<T>`, pentru a crea o valoare cu numărare referențială. Să încercăm același lucru aici și să vedem ce rezultă. Vom folosi `Mutex<T>` incapsulat în `Rc<T>` în Listarea 16-14 și vom clona `Rc<T>` înainte de a transfera posesiunea către fir.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

<span class="caption">Listarea 16-14: Tentativa de a folosi `Rc<T>` pentru a permite mai multor fire să dețină `Mutex<T>`</span>

Încercând din nou să compilăm, ne confruntăm cu... niște erori noi! Compilatorul se dovedește a fi un excelent mijloc de învățare.

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Ce mesaj de eroare copios! Să ne concentrăm pe informația crucială: `` `Rc<Mutex<i32>>` cannot be sent between threads safely `` (`Rc<Mutex<i32>>` nu se poate transmite în siguranță între fire). Compilatorul ne explică și de ce: ``the trait `Send` is not implemented for
`Rc<Mutex<i32>>` `` (trăsătura `Send` nu este implementată pentru `Rc<Mutex<i32>>`). Vom discuta despre `Send` în secțiunea viitoare; este una din trăsăturile esențiale care se asigură că tipurile pe care le utilizăm împreună cu firele de execuție sunt pregătite pentru scenarii de concurență.

Din nefericire, `Rc<T>` nu este proiectat să fie distribuit în siguranță între fire. Când `Rc<T>` este responsabil de contorizarea referințelor, el adaugă la contor pentru fiecare apel la `clone` și scade din contor când o clonă este descărcată. Însă nu apelează la nicio structură de concurență care să garanteze că modificările contorului nu sunt susceptibile de a fi întrerupte de un alt fir. Acest aspect poate conduce la erori de contorizare care, pe neobservate, pot cauza scurgeri de memorie sau pot determina distrugerea unei valori înainte de finalizarea folosinței sale. Avem nevoie de un tip similar cu `Rc<T>`, dar care să actualizeze contorul de referințe într-o manieră compatibilă cu utilizarea în fire de execuție.

#### Numărarea atomară a referințelor cu `Arc<T>`

Din fericire, `Arc<T>` *este* un tip la fel ca `Rc<T>` care este sigur pentru utilizare în contexte concurente. Litera *a* reprezintă *atomară*, adică este un tip cu *numărare atomară a referințelor*. *Atomicele* sunt o altă categorie de primitive pentru concurență pe care nu o vom discuta în detaliu aici: vezi documentația pentru [`std::sync::atomic`][atomic]<!-- ignore --> din biblioteca standard pentru mai multe informații. Deocamdată e suficient să știi că *atomicele* funcționează similare cu tipurile primitive și sunt sigure pentru a fi partajate între fire de execuție.

Poate te întrebi de ce toate tipurile primitive nu sunt atomice și de ce tipurile din biblioteca standard nu sunt construite să folosească implicit `Arc<T>`. Motivul este că siguranța pe mai multe fire de execuție aduce cu sine un cost de performanță pe care dorești să-l accepți numai când ai cu adevărat nevoie. Dacă operațiunile tale se desfășoară în cadrul unui singur fir de execuție atunci codul tău poate fi mai rapid dacă nu e nevoit să aplice garanțiile pe care atomicele le oferă.

Revenind la exemplul nostru: `Arc<T>` și `Rc<T>` partajează aceeași interfață API, deci putem ajusta programul nostru modificând linia `use`, apelul la `new` și apelul la `clone`. Codul din Listarea 16-15 va fi în final capabil să compileze și să ruleze:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

<span class="caption">Listarea 16-15: Utilizarea lui `Arc<T>` pentru a incapsula `Mutex<T>` astfel încât să permită partajarea posesiunii pe mai multe fire de execuție</span>

Iată ce va afișa codul:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Result: 10
```

Am reușit! Am contorizat de la 0 la 10, lucru care s-ar putea să nu pară impresionant, dar ne-a învățat o mulțime despre `Mutex<T>` și despre siguranța firelor de execuție. De asemenea, ai putea utiliza structura acestui program pentru a realiza operațiuni mai complexe decât simpla incrementare a unui contor. Cu această metodologie, poți descompune un calcul în segmente independente, repartiza aceste segmente între fire de execuție diferite, și folosi `Mutex<T>` pentru ca fiecare fir să își contribuie partea la rezultatul final.

Ia în considerare că, pentru operațiuni numerice simple, există tipuri mai accesibile decât `Mutex<T>` oferite de modulul [`std::sync::atomic`] din biblioteca standard[atomic]<!-- ignore -->. Aceste tipuri permit un acces sigur, concurent și atomic la tipurile de bază. Am ales utilizarea `Mutex<T>` cu un tip simplu în acest exemplu pentru a nu ne distrage atenția de la funcționarea lui `Mutex<T>`.

### Asemănările dintre `RefCell<T>`/`Rc<T>` și `Mutex<T>`/`Arc<T>`

Probabil ai remarcat că variabila `counter` este imutabilă, dar am reușit să obținem o referință mutabilă la valoarea dinăuntrul ei; acest lucru înseamnă că `Mutex<T>` asigură mutabilitate internă, așa cum o face familia `Cell`. Astfel cum am utilizat `RefCell<T>` în Capitolul 15 pentru a ne permite să modificăm conținutul unui `Rc<T>`, utilizăm `Mutex<T>` pentru a modifica conținutul unui `Arc<T>`.

Un alt detaliu important de menționat este că Rust nu poate să te protejeze de toate tipurile de erori de logică când folosești `Mutex<T>`. Reamintește-ți din Capitolul 15 că utilizarea lui `Rc<T>` prezenta riscul de a crea cicluri de referințe, în care două valori `Rc<T>` făceau referire reciprocă, provocând scurgeri de memorie. În mod asemănător, `Mutex<T>` prezintă riscul creării *interblocajelor* (deadlocks). Acestea se întâmplă când o operație necesită blocarea a două resurse și două fire de execuție au blocat fiecare câte una dintre acestea, determinându-le să aștepte reciproc la nesfârșit. Dacă te interesează interblocajele, încearcă să scrii un program în Rust care să genereze un interblocaj; apoi cercetează strategiile de evitare a interblocajelor pentru mutexuri in orice limbaj și încearcă să le aplici în Rust. Documentația API a bibliotecii standard pentru `Mutex<T>` și `MutexGuard` furnizează informații de valoare.

În încheierea acestui capitol, vom discuta despre trăsăturile `Send` și `Sync` și cum pot fi ele utilizate cu tipuri definite de utilizator.

[atomic]: ../std/sync/atomic/index.html
