## Să `panic!`-ăm sau nu?

Cum alegi când să apelezi la `panic!` și când e preferabil să returnezi un `Result`? Odată ce codul intră în panică, nu mai există cale de recuperare. Ai putea utiliza `panic!` pentru orice eroare, indiferent dacă există o șansă de reparare sau nu, dar astfel decizi tu că o situație e de nerezolvat, în locul celui care folosește codul tău. Prin furnizarea unei valori `Result`, permiți utilizatorului codului să aleagă soluția adecvată contextului său sau să determine că o valoare `Err` este una definitivă, apelând la `panic!` pentru a-și asuma ireversibilitatea. De aceea, este indicat să returnezi un `Result` când concepi o funcție care ar putea să nu funcționeze cum trebuie.

În situații precum documentația de exemplu, prototipuri și teste, e mai potrivit să optezi pentru codul ce induce `panic!` decât să returnezi un `Result`. Explorăm acum de ce este asta așa, și vom discuta cazurile în care compilatorul nu detectează imposibilitatea unui eșec, dar tu în calitate de programator înțelegi situația. Capitolul se va încheia cu un set de linii directoare fundamentale pentru a decide dacă să folosești `panic!` în codul bibliotecilor.

### Exemple, cod de prototipare și teste

Când elaborezi un exemplu pentru a ilustra un concept, includerea codului complex pentru gestionarea erorilor poate aduce un minus de claritate. Se înțelege că, în exemple, utilizarea unei metode cum ar fi `unwrap`, care ar putea declanșa o panică, este doar un substituent temporar pentru modul în care ai gestiona normal erorile în aplicația ta, acest mod variind în funcție de codul existent.

Similar, metodele `unwrap` și `expect` sunt de mare ajutor în stadiul de prototipare, când încă nu ai hotărât cum să abordezi gestionarea erorilor. Ele lasă semne evidente în cod pentru momentul în care ești pregătit să îți consolidezi programul.

Dacă un apel al metodei nu reușește în timpul testării, ideal este ca întreg testul să fie afectat de acest eșec, chiar dacă metoda în cauză nu este funcționalitatea principală care se testează. Deoarece instrucțiunea `panic!` semnalează eșecul unui test, utilizarea `unwrap` sau `expect` este tocmai procedura adecvată în acest context.

### Situațiile în care ai mai multe informații decât compilatorul

Este recomandabil să apelezi `unwrap` sau `expect` atunci când deții o anumită logică de asigurare că `Result` va fi `Ok`, chiar dacă logica respectivă nu este interpretată de compilator. În asemenea cazuri, încă trebuie să gestionăm valoarea `Result`: orice funcție invocată are potențialul de a eșua în principiu, chiar dacă eșecul este logic imposibil în situația particulară actuală. Dacă ești capabil prin verificarea manuală a codului să asiguri absența unei variante `Err`, apelarea lui `unwrap` este complet justificată și chiar încurajată. În plus, documentarea motivului pentru care este exclusă o variantă `Err` în mesajul metodei `expect` este o practică deosebit de beneficiară. Aici este un exemplu:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```

Aici instanțiăm o structură `IpAddr` prin parsarea unui string predefinit. Cum `127.0.0.1` este clar o adresă IP validă, este logic să apelăm `expect`. Totuși, prezența unui string valid și predefinit nu influențează tipul de retur pentru metoda `parse`: rămânem cu o valoare de tip `Result`, iar compilatorul ne va solicita să gestionăm `Result` ca și când varianta `Err` ar fi o eventualitate, deoarece nu este echipat să recunoască automat că acest string este mereu o adresă IP validă. Dacă sursa string-ului cu adresa IP ar fi exterioară, adică provenind de la un utilizator și nu fiind stabilită inițial în cod, atunci cu certitudine am prefera o abordare mai atentă a valorii `Result`. Evidențierea faptului că adresa IP este predefinită în cod ne stimulează să actualizăm `expect` cu un cod de gestionare a erorilor mai avansat dacă e necesar să adaptăm sursa de unde obținem adresa IP în viitor.

### Recomandări pentru tratarea erorilor

Este prudent să lași codul tău să declanșeze panică în momentele când s-ar putea să ajungă într-o stare critică. Prin *stare critică* înțelegem situația în care o anumită asumpție, garanție, acord sau invariant a fost compromis, exemplificat prin primirea de valori invalide, contradictorii sau inexistente - plus una sau mai multe dintre următoarele circumstanțe:

* Starea critică este una imprevizibilă, nu una care ar apărea frecvent, cum ar fi erorile de introducere a datelor de către un utilizator.
* Codul tău de după acest punct presupune că nu se află în acea stare critică și nu verifică problema la fiecare etapă.
* Nu este posibil să exprimi aceste informații suficient de clar utilizând tipurile curente. Vom explora această idee prin intermediul unui exemplu în secțiunea [„Exprimarea stărilor și comportamentelor prin tipuri”][encoding]<!-- ignore --> din capitolul 17.

Dacă cineva folosește codul tău și introduce valori nesigure, este ideal să returnezi o eroare, dacă este posibil, astfel încât cel ce folosește biblioteca ta să determine cea mai bună acțiune de urmat. Totuși, în situații unde continuarea ar putea fi riscantă sau dăunătoare, decizia cea mai judicioasă ar fi să folosești `panic!`. Asta va notifica persoana care utilizează biblioteca ta despre defectul din codul său, permițând corectarea acestuia în cadrul dezvoltării. De asemenea, este adesea adecvat să folosești `panic!` atunci când execuți cod extern peste care nu ai control și acesta returnează o stare defectuoasă pe care nu ai cum să o corectezi.

Totuși, când un eșec este prevăzut, este preferabil să returnăm un `Result` în loc să apelăm `panic!`. Exemplele pot include situația în care un parser primește date corupte sau o solicitare HTTP care returnează un statut ce arată că s-a atins limita de rată. În aceste cazuri, returnarea unui `Result` indică faptul că eșecul este recunoscut ca o posibilitate așteptată, pe care codul apelant trebuie să o manajeze.

Când codul efectuează o operație care poate fi riscantă pentru un utilizator dacă este chemată cu valori invalide, trebuie să verifice dacă valorile sunt valide înainte și să panicheze dacă acestea nu sunt. Motivul principal este securitatea: lucrul cu date invalide poate crea vulnerabilități în codul tău. Acesta este motivul pentru care biblioteca standard va produce `panic!` dacă se încearcă accesul la memorie dincolo de limitele permise: încercarea de a accesa memoria care nu face parte din structura de date actuală este o problemă obișnuită de securitate. Funcțiile au adesea *contracte*: comportamentul lor este garantat doar dacă intrările îndeplinesc cerințele specificate. A panica atunci când un contract este încălcat este justificat deoarece o încălcare a contractului indică mereu o problemă din partea celui care apelează și nu este un tip de eroare care ar trebui gestionat explicit de către codul apelant. În esență, nu există o modalitate rezonabilă prin care codul apelant să poată remedia; *programatorii* apelanți trebuie să repare codul. Contractele unei funcții, în special atunci când nerespectarea lor conduce la panică, ar trebui să fie explicite în documentația API a respectivei funcții.

A include numeroase verificări de erori în toate funcțiile poate deveni copleșitor și plictisitor. Din fericire, sistemul de tipizare oferit de Rust și verificarea tipurilor efectuată de compilatorul lui Rust îți permit să automatizezi multe din aceste controale. Atunci când funcția ta specifică un anumit tip pentru un parametru, îți poți duce execuția codului mai departe având certitudinea că ai primit o valoare validă, grație compilatorului. De exemplu, dacă preferi un tip concret în locul tipului `Option`, programul se așteaptă să opereze cu *ceva* și nu cu *nimic*. Acest lucru înseamnă că nu trebuie să gestionezi separat cazurile `Some` și `None`, ci doar situația în care ai deja o valoare garantată. Astfel, încercările de a folosi funcția cu valori nule sunt oprite în faza de compilare, eliminând necesitatea efectuării acestei verificări în timpul execuției. Un alt exemplu, optarea pentru tipuri de numere întregi fără semn, precum `u32`, îți asigură că parametrul nu va putea fi niciodată negativ.

### Crearea de tipuri particularizate pentru validare

Să explorăm cum putem utiliza sistemul de tipuri din Rust pentru a ne asigura că avem valori valide, prin crearea de tipuri personalizate pentru validarea acestora. Revenind la jocul cu ghicirea numărului din Capitolul 2, unde codul cerea utilizatorului să ghicească un număr între 1 și 100, observăm că nu am validat dacă ghicirea utilizatorului se încadra în acest interval înainte de a o compara cu numărul secret; ne-am limitat doar la verificarea pozitivității ghicirii. Cu toate că în acest caz consecințele nerespectării intervalului nu erau majore – răspunsurile „Prea mare” sau „Prea mic” fiind adecvate –, tot ar fi benefică îndrumarea utilizatorilor spre ghiciri corecte și o diferențiere clară a comportamentului programului atunci când utilizatorul introduce un număr în afara intervalului sau, de exemplu, litere.

Un mod de a implementa acest lucru ar fi prin parsarea ghicirii ca un `i32`, care permite numere negative, și adăugarea unei verificări care să confirme că numărul se află în intervalul dorit:

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

Instrucțiunea `if` verifică dacă valoarea introdusă este în afara intervalului, informează utilizatorul în privința erorii și execută `continue` pentru a iniția o nouă iterație a buclei și a solicita o altă ghicire. După această verificare, comparațiile dintre `guess` și numărul secret pot fi efectuate cu certitudinea că `guess` cade între 1 și 100.

Cu toate acestea, soluția nu este optimă într-un context în care este critic ca programul să lucreze exclusiv cu valori între 1 și 100 – mai ales dacă mai multe funcții impun această limită –, căci inserarea aceluiași tip de verificare în fiecare dintre ele ar fi monotonă și ar putea influența performanța.

Ca alternativă, am putea defini un tip nou și să centralizăm validările într-o funcție dedicată creării de instanțe ale acestui tip, evitând astfel repetarea validărilor. Astfel, este sigur de utilizat noul tip în semnăturile funcțiilor, care ar putea opera cu încredere folosind valorile primite. În Listarea 9-13, prezentăm o metodă de a defini un tip `Guess`, care va crea o instanță validă a acestuia numai dacă funcția `new` este invocată cu o valoare între 1 și 100.

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file requires the `rand` crate. We do want to include it for reader
experimentation purposes, but don't want to include it for rustdoc testing
purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-13/src/main.rs:here}}
```

<span class="caption">Listarea 9-13: Tipul `Guess` ce acceptă numai valorile între 1 și 100</span>

Inițial, definim o structură cu denumirea `Guess`, având un câmp numit `value` care conține un `i32`. Acesta este locul unde va fi stocat numărul.

Ulterior, implementăm o funcție asociată intitulată `new` pentru structura `Guess`, pentru a crea instanțe ale tipului `Guess`. Funcția `new` este configurată să primească un parametru numit `value`, de tip `i32`, și să returneze un `Guess`. În corpul funcției `new`, verificăm valoarea lui `value` pentru a ne asigura că se încadrează între 1 și 100. În cazul în care `value` nu îndeplinește acest criteriu, utilizăm macro-ul `panic!`, care va semnala programatorului care a apelat codul despre prezența unui defect ce necesită rezolvare, deoarece constituirea unui `Guess` cu o valoare `value` exterioară acelui interval ar încălca prerogativele funcției `Guess::new`. Situațiile în care `Guess::new` ar putea cauza o panică trebuie detaliate în documentația API destinată publicului; vom aborda standardele de documentare ce marchează posibilitatea unei panici în documentația API pe care o veți alcătui în Capitolul 14. Dacă `value` satisface cerința, compilăm o nouă structură `Guess`, stabilind valoarea câmpului `value` la parametrul primit și întorcând `Guess`.

Mai departe, implementăm o metodă denumită `value` ce împrumută `self` și returnează un `i32`, fără a solicita alți parametri. Această metodă este adesea identificată ca *getter*, deoarece scopul ei este de a extrage informații din câmpurile proprii și de a le furniza extern. Această metodă publică este esențială dat fiind că atributul `value` al structurii `Guess` este privat. Este crucial pentru atributul `value` să fie privat, astfel încât codul care utilizează structura `Guess` să nu fie în măsură să seteze `value` în mod direct: codul din afara modulului *trebuie* să recurgă la funcția `Guess::new` pentru a asambla o instanță de `Guess`, garantându-se în acest mod că orice `Guess` posedă un `value` validat de condițiile prezentate în funcția `Guess::new`.

O funcție care manipulează numai numere între 1 și 100 poate alege să indice în semnătura sa că primește sau returnează o structură `Guess` în loc de un `i32`, ceea ce ar permite să ocolească efectuarea unor verificări în plus în conținutul său.

## Sumar

Capacitățile de gestionare a erorilor din Rust sunt concepute pentru a sprijini scrierea unui cod cât mai solid. Macro-ul `panic!` indică o stare irecuperabilă a programului și oferă posibilitatea de a întrerupe execuția, evitând continuarea cu valori greșite sau invalide. Enum-ul `Result` exploatează sistemul de tipuri al Rust pentru a scoate în evidență potențialul eșec al operațiunilor, din care codul tău ar putea să revină. `Result` poate fi folosit pentru a informa codul client că trebuie gestionată atât reușita, cât și eșecul. Folosirea judicioasă a `panic!` și a `Result` va crește fiabilitatea codului tău în fața problemelor inevitabile.

Având în vedere utilizările benefice ale genericilor în enum-urile `Option` și `Result` de către biblioteca standard, vom discuta în continuare despre funcționarea genericilor și modul în care pot fi implementați în codul tău.

[encoding]: ch17-03-oo-design-patterns.html#encoding-states-and-behavior-as-types
