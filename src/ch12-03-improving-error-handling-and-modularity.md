## Refactorizarea pentru îmbunătățirea modularității și gestionarea erorilor

Pentru a îmbunătăți programul nostru, ne vom axa pe rezolvarea a patru probleme care sunt legate de structura programului și de modul cum gestionează acesta erorile potențiale. În primul rând, funcția noastră `main` îndeplinește în prezent două sarcini: parsează argumentele și citește fișiere. Odată cu creșterea programului nostru, numărul de sarcini gestionate de funcția `main` de asemenea va crește. Pe măsură ce o funcție acumulează mai multe responsabilități, devine tot mai dificil de analizat, mai complicat de testat și mai greoi de modificat fără a afecta una dintre funcționalitățile sale. Este preferabil să separăm funcționalitățile astfel încât fiecare funcție să fie responsabilă doar de o anumită sarcină.

Al doilea aspect ce necesită atenție este faptul că, deși `query` și `file_path` sunt variabile de configurare pentru program, avem și variabile cum ar fi `contents` ce sunt folosite pentru implementarea logicii programului. Cu cât blocul `main` se extinde, cu atât va fi necesar să aducem în scop mai multe variabile; iar odată cu creșterea numărului de variabile în domeniul de vizibilitate, devine tot mai complicat să urmărim funcția fiecăreia. Ar fi mai adecvat să consolidăm variabilele de configurare într-o singură structură pentru astfel a clarifica rolul lor.

A treia problemă constă în utilizarea instrucțiunii `expect` pentru generarea unui mesaj de eroare când citirea fișierului eșuează, care acum se rezumă doar la `Should have been able to read the file`. Eșecul citirii unui fișier poate surveni din diverse motive - cum ar fi absența fișierului sau lipsa permisiunii de acces. Momentan, am afișa același mesaj indiferent de situație, fără a furniza ceva informativ utilizatorului.

A patra problemă este legată de utilizarea repetată a lui `expect` în tratamentul diferitelor erori și faptul că, dacă un utilizator execută programul fără a oferi suficiente argumente, se va întâlni cu o eroare de tip `index out of bounds` din partea Rust, o eroare care nu descrie clar problema. Ideal ar fi ca întreg codul de gestionare a erorilor să fie centralizat, astfel încât viitorii dezvoltatori să aibă un singur punct de referință la care să se raporteze dacă logica de gestionare a erorilor necesită ajustări. Concentrând codul destinat erorilor într-un loc unic ne garantăm că mesajele generate sunt pertinente și utile pentru utilizatorii finali.

Să ne ocupăm de aceste patru probleme printr-o atentă refactorizare a proiectului.

### Separarea responsabilităților în proiectele binare

Problema organizațională de atribuire a responsabilităților pentru diverse sarcini funcției `main` este frecventă în multe proiecte binare. Drept consecință, comunitatea Rust a elaborat ghiduri pentru descompunerea preocupărilor separate ale unui program binar când funcția `main` începe să crească prea mult în dimensiune. Procesul cuprinde următorii pași:

* Împarte programul într-un *main.rs* și un *lib.rs* și transferă logica programului
  în *lib.rs*.
* Dacă logica de procesare a liniei de comandă este simplă, aceasta poate să
  rămână în *main.rs*.
* Când logica de procesare a liniei de comandă devine complexă, extrage-o
  din *main.rs* și mut-o în *lib.rs*.

Responsabilitățile ce ar trebui să rămână în funcția `main` în urma acestui proces se limitează la:

* Apelarea logicii de procesare a liniei de comandă cu valorile argumentelor
* Configurarea oricăror setări suplimentare
* Invocarea unei funcții `run` din *lib.rs*
* Tratarea erorii dacă `run` returnează o eroare

Această metodologie se axează pe separarea responsabilităților: *main.rs* se ocupă de execuția programului, iar *lib.rs* gestionează integral logica specifică sarcinii. Deoarece funcția `main` nu poate fi testată în mod direct, această structură îți oferă posibilitatea să testezi toată logica programului prin includerea ei în funcții din *lib.rs*. Codul rezidual din *main.rs* va fi atât de concis încât corectitudinea lui se poate constata prin simpla lectură. Să procedăm la reconfigurarea programului nostru aplicând acești pași.

#### Extragerea parserului de argumente

Vom separa funcționalitatea de parsare a argumentelor într-o funcție pe care `main` o va chema, pregătind astfel mutarea logicii de parsare a argumentelor din linia de comandă în *src/lib.rs*. Listarea 12-5 ilustrează noul început al funcției `main`, unde aceasta apelează o nouă funcție `parse_config`, definită momentan în *src/main.rs*.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

<span class="caption">Listarea 12-5: Extragerea unei funcții `parse_config` din `main`</span>

Continuăm să adunăm argumentele liniei de comandă într-un vector, dar în loc de a atribui direct valoarea argumentului de la indexul 1 variabilei `query` și valoarea argumentului de la indexul 2 variabilei `file_path` în funcția `main`, acum transmitem întreg vectorul către funcția `parse_config`. Funcția `parse_config` are acum rolul de a determina care argument corespunde căreia dintre variabile și trimite valorile înapoi în `main`. În `main` continuăm să creăm variabilele `query` și `file_path`, însă `main` nu mai are rolul de a face corelația dintre argumentele de comandă și variabile.

Această restructurare ar putea să pară inutilă pentru un program de dimensiuni atît de reduse ca al nostru, însă anume așa și refactorizăm: în pași mici și consecutivi. După ce efectezi această modificare, rulează iar programul pentru a te asigura că procesul de parsare a argumentelor funcționează în continuare. E recomandat să verifici progresul frecvent, pentru a putea identifica mai ușor sursa problemelor atunci când apar.

#### Gruparea valorilor de configurare

Putem face încă un mic pas pentru a îmbunătăți funcția `parse_config`.
În prezent, returnăm o tuplă, dar apoi o descompunem imediat în componentele ei individuale. Acest lucru poate semnala faptul că nu am ajuns încă la abstracția corectă.

Alt semnal care indică posibilitatea îmbunătățirii este partea `config` din `parse_config`, ce sugerează că cele două valori returnate sunt interconectate și constituie împreună o configurație unitară. În momentul de față, nu comunicăm acest sens în structura datelor, decât prin gruparea valorilor într-o tuplă; în loc de aceasta, vom încorpora cele două valori într-o structură și le vom atribui câmpurilor nume sugestive. Aceasta va simplifica munca viitorilor dezvoltatori care vor interacționa cu codul, facilitând înțelegerea interdependenței valorilor și scopurilor pe care le servesc.

Listarea 12-6 prezintă îmbunătățirile aduse funcției `parse_config`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

<span class="caption">Listarea 12-6: Refactorizarea funcției `parse_config` pentru returnarea unei instanțe a structurii `Config`</span>

Am introdus o structură denumită `Config`, concepută cu câmpurile `query` și `file_path`. Semnătura funcției `parse_config` reflectă acum faptul că se returnează o valoare de tip `Config`. În conținutul `parse_config`, acolo unde înainte returnam secțiuni care făceau referire la valori de tip `String` din `args`, configurăm acum `Config` astfel încât să conțină în directă posesiune valorile sale `String`. Variabila `args` din funcția `main` este proprietara valorilor argumente și permite funcției `parse_config` doar să le împrumute, iar dacă `Config` ar încerca să preia controlul asupra acestor valori atunci situația ar contraveni regulilor Rust privind împrumutul.

Există multiple metode prin care am putea controla datele de tip `String`; cea mai simplă, deși posibil ineficientă, este să apelăm metoda `clone` pe respectivele valori. Acest lucru ar genera o copie integrală a datelor, pe care instanța `Config` le-ar deține, consumând mai mult timp și memorie decât dacă am stoca o referință la datele string. Totuși, clonarea datelor are avantajul de a ne simplifica codul, nefiind necesar să gestionăm duratele de viață ale referințelor. În această situație, renunțarea la o cantitate mică de performanță în schimbul simplificării este un compromis justificat.

> ### Dezavantajele utilizării funcției `clone`
>
> Mulți dintre cei care programează în Rust au tendința de a evita utilizarea
> funcției `clone` pentru a soluționa problemele de posesiune, din cauza
> impactului pe care îl are asupra timpului de execuție. În
> [Capitolul 13][ch13]<!-- ignore -->, vei învăța metode mai eficiente pentru
> aceste tipuri de situații. Totuși, în acest moment, nu este o problemă să
> copiezi câteva string-uri pentru a avansa în progresul tău, deoarece aceste
> copieri se vor face doar o singură dată și atât string-ul de interogare,
> cât și calea directoriului tău sunt destul de reduse ca dimensiune. Este
> preferabil să ai un program funcțional care nu este optimizat la maximum
> decât să încerci să optimizezi excesiv codul din prima încercare. Pe măsură
> ce vei deveni mai versat în Rust, va fi mai simplu să pornești direct cu
> soluția cea mai eficientă, dar pentru moment, este complet acceptabil să
> apelezi la `clone`.

Am modificat funcția `main` astfel încât să atribuie instanța de `Config` întoarsă de `parse_config` unei variabile denumite `config`, și am actualizat codul care anterior utiliza variabilele `query` și `file_path` separat, astfel încât acum să acceseze câmpurile structurii `Config`.

În acest fel, codul nostru comunică mai eficient faptul că `query` și `file_path` sunt corelate și că funcția lor este de a seta configurarea pentru comportamentul programului. Orice porțiune de cod care le folosește va ști că trebuie să le caute în instanța `config`, în câmpurile cu numele corespunzător scopului lor.

#### Crearea unui constructor pentru `Config`

Până acum, am separat logica de parsare a argumentelor liniei de comandă din `main` și am inclus-o în funcția `parse_config`. Acest demers ne-a ajutat să observăm că valorile pentru `query` și `file_path` sunt interconectate și această conexiune trebuie evidențiată în codul nostru. În consecință, am introdus structura `Config` pentru a numi scopul conex al `query` și `file_path` și pentru a putea returna numele acestor valori sub forma unor nume de câmpuri ale structurii din cadrul funcției `parse_config`.

Așadar, fiindcă rolul funcției `parse_config` este de a inițializa o instanță `Config`, putem transforma `parse_config` dintr-o funcție obișnuită într-o funcție numită `new`, asociată acum structurii `Config`. Prin această modificare, codul nostru va deveni mai idiomatic. Putem crea instanțe ale tipurilor din biblioteca standard, cum ar fi `String`, invocând `String::new`. În mod similar, prin schimbarea funcției `parse_config` în `new`, asociată cu `Config`, vom putea crea instanțe de `Config` invocând `Config::new`. Listarea 12-7 ilustrează schimbările pe care trebuie să le efectuăm.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

<span class="caption">Listarea 12-7: Transformarea `parse_config` în `Config::new`</span>

Am actualizat locul din `main` unde `parse_config` era apelată, pentru a folosi în schimb `Config::new`. Am redenumit `parse_config` în `new` și am transferat-o într-un bloc `impl`, asociind astfel funcția `new` cu structura `Config`. Încearcă să compilezi din nou codul pentru a te asigura că funcționează cum trebuie.

### Remediind problemele de tratare a erorilor

Acum ne vom concentra pe îmbunătățirea gestionării erorilor. Reamintim că încercarea de a accesa valorile din vectorul `args` la indexul 1 sau 2 poate provoca panică în program dacă vectorul are mai puțin de trei elemente. Încercând rularea programului fără niciun argument; ieșirea va arăta în felul următor:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

#### Îmbunătățind mesajul de eroare

În Listarea 12-8, includem o verificare în funcția `new` ce se asigură că secțiunea este destul de lungă înainte de a accesa indecșii 1 și 2. Dacă secțiunea nu este suficientă, programul va intra în panică și va afișa un mesaj de eroare îmbunătățit.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

<span class="caption">Listarea 12-8: Adăugând o verificare pentru numărul de argumente</span>

Acest cod este similar cu funcția `Guess::new` pe care am creat-o în Listarea 9-13][ch9-custom-types]<!-- ignore -->, unde am folosit macro-ul `panic!` când valoarea `value` era în afara limitei valorilor valide. Aici, în loc de verificarea unui interval de valori, ne asigurăm că lungimea `args` este cel puțin 3, iar restul funcției poate opera sub presupunerea că această condiție este îndeplinită. Dacă `args` are mai puțin de trei elemente, această condiție este verificată și se apelează macro-ul `panic!` pentru a opri imediat programul.

Cu aceste câteva linii de cod suplimentare în `new`, să consultăm din nou rularea programului fără argumente pentru a vedea cum arată eroarea acum:

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Rezultatul este mai bun: în sfîrșit avem un mesaj de eroare adecvat. Totuși, rămânem cu informații suplimentare care nu sunt necesare utilizatorilor noștri. Se pare că metoda utilizată în Listarea 9-13 nu este cea mai potrivită aici: un apel la `panic!` e mai adecvat pentru o problemă de programare decât una de utilizare, așa cum am discutat în Capitolul 9. În schimb, vom aplica o altă metodă pe care ai învățat-o în Capitolul 9—[returnarea unui `Result`][ch9-result] care indică fie succesul, fie o eroare.

<!-- Old headings. Do not remove or links may break. -->
<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### Returnarea unui `Result` în loc de apelarea `panic!`

Putem opta pentru returnarea unei valori `Result` care va include o instanță `Config` în cazul unui succes și va detalia problema în situația unei erori. Intenționăm să schimbăm numele funcției din `new` în `build`, deoarece mulți programatori presupun că funcțiile `new` nu ar trebui să eșueze niciodată. În comunicarea dintre `Config::build` și `main`, folosim tipul `Result` pentru a semnala potențiala apariție a unei probleme. Astfel, putem ajusta funcția `main` să convertească o variantă `Err` într-o eroare mai prietenoasă pentru utilizatorii noștri, fără textul suplimentar legat de `thread 'main'` și `RUST_BACKTRACE` care însoțește apelul la `panic!`.

Listarea 12-9 prezintă modificările necesare valorii de retur a funcției denumite acum `Config::build` și ale corpului acesteia necesare pentru a returna un `Result`. Notăm că aceste schimbări nu vor compila decât după ce actualizăm și funcția `main`, lucru pe care îl vom face în lista următoare.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

<span class="caption">Listarea 12-9: Returnarea unui `Result` din `Config::build`</span>

Funcția `build` returnează un `Result` care conține o instanță `Config` în caz de succes și un `&'static str` în caz de eroare, valorile de eroare fiind întotdeauna literali de string cu durata de viață `'static`.

Am efectuat două schimbări în corpul funcției: în loc să folosim `panic!` când utilizatorul nu furnizează suficiente argumente, acum returnăm o valoare `Err` și am ambalat valoarea de retur `Config` într-un `Ok`. Aceste ajustări asigură conformitatea funcției cu noua sa semnătură de tip.

Returnând o valoare `Err` în cadrul `Config::build`, funcția `main` poate gestiona valoarea `Result` returnată din `build` și poate încheia procesul într-o manieră mai curată în situația unei erori.

<!-- Old headings. Do not remove or links may break. -->
<a id="calling-confignew-and-handling-errors"></a>

#### Apelarea `Config::build` și gestionarea erorilor

Pentru a gestiona cazul de eroare și a afișa un mesaj prietenos pentru utilizatori, trebuie să actualizăm funcția `main` pentru a prelucra `Result` returnat de `Config::build`, așa cum e demonstrat în Listarea 12-10. De asemenea, ne asumăm responsabilitatea de a încheia execuția instrumentului de linie de comandă cu un cod de eroare non-zero, rol care anterior îl avea `panic!`, și implementăm această funcție manual. Un cod de ieșire non-zero este o convenție care semnalează procesului ce a invocat programul nostru că acesta s-a terminat cu o stare de eroare.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

<span class="caption">Listarea 12-10: Terminarea execuției cu un cod de eroare dacă inițializarea unui `Config` nu reușește</span>

În această listare am folosit metoda `unwrap_or_else`, care nu a fost încă detaliată: `unwrap_or_else` este definită pentru `Result<T, E>` de către biblioteca standard. Prin utilizarea `unwrap_or_else`, definim un comportament personalizat pentru gestionarea erorilor, care nu recurge la `panic!`. Dacă `Result` este o valoare `Ok`, metoda funcționează similar cu `unwrap`, returnând valoarea din interiorul `Ok`. În schimb, dacă avem o valoare de tip `Err`, metoda invocă codul specificat într-o *închidere* (closure), adică o funcție anonimă definită de noi și pasată ca argument la `unwrap_or_else`. Vom aborda închiderile în detaliu în [Capitolul 13][ch13]<!-- ignore -->. Deocamdată, este suficient să înțelegeți că `unwrap_or_else` va trimite valoarea internă a erorii `Err`, în acest caz șirul static `"not enough arguments"`, către închiderea noastră prin intermediul argumentului `err`, care este plasat între barele verticale. Astfel, închiderea poate folosi variabila `err` atunci când se execută.

Am adăugat un nou rând `use` pentru a include `process` din biblioteca standard în domeniul de vizibilitate. Codul din închidere care va fi executat în caz de eroare conține doar două linii: afișăm valoarea `err` și invocăm `process::exit`. Funcția `process::exit` oprește programul imediat și returnează numărul care a fost specificat ca cod de ieșire. Acesta este un comportament similar cu cel din gestionarea bazată pe `panic!` prezentată în Listarea 12-8, însă de data aceasta fără output-ul adițional. Acum să testăm:

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Excelent! Acest mesaj este mult mai accesibil utilizatorilor noștri.

### Extragerea logicii din `main`

Odată finalizată refactorizarea analizei configurației, ne îndreptăm acum spre logica programului. Conform celor stabilite în [„Separarea problemelor pentru proiectele binare”](separation-of-concerns-for-binary-projects)<!-- ignore -->, extragem o funcție denumită `run` care va cuprinde toată logica existentă în funcția `main`, cu excepția părților care se ocupă de configurarea inițială sau de gestionarea erorilor. La finalizare, `main` va fi concisă și ușor de verificat printr-o simplă inspecție, iar noi vom fi capabili să scriem teste pentru întreaga logică restantă.

Listarea 12-11 ilustrează procesul de extracție a funcției `run`. Pentru moment, aceasta reprezintă un pas mic și gradual în îmbunătățirea codului. Funcția continuă să fie definită în *src/main.rs*.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

<span class="caption">Listarea 12-11: Extragerea funcției `run`, care conține restul logicii din program</span>

Funcția `run` încorporează acum toată logica rămasă din `main`, începând de la etapa de citire a fișierului. Aceasta primește o instanță de `Config` ca parametru.

#### Returnarea erorilor de către funcția `run`

Cu partea rămasă de logică a programului separată acum în funcția `run`, avem oportunitatea de a îmbunătăți gestionarea erorilor, așa cum am procedat pentru `Config::build` în Listarea 12-9. În loc să lăsăm programul să genereze panică prin apelarea `expect`, funcția `run` va returna un `Result<T, E>` când ceva nu funcționează corect. Aceasta ne va da posibilitatea de a centraliza mai eficient gestionarea erorilor în `main`, într-un mod accesibil utilizatorului. Listarea 12-12 prezintă modificările necesare la semnătura și corpul funcției `run`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

<span class="caption">Listarea 12-12: Modificarea funcției `run` pentru a returna `Result`</span>

Am realizat trei schimbări importante aici. Prima, schimbarea tipului de retur pentru funcția `run` în `Result<(), Box<dyn Error>>`. Inițial, această funcție returna tipul unit `()`, pe care acum îl păstrăm ca valoare returnată în cazul de succes `Ok`.

Ca tip de eroare, am utilizat *obiectul-trăsătură* `Box<dyn Error>` (și am importat `std::error::Error` în context cu o directivă `use` la începutul fișierului). Vom discuta despre obiecte-trăsătură în [Capitolul 17][ch17]<!-- ignore -->. Deocamdată, e de ajuns să știm că `Box<dyn Error>` înseamnă că funcția va returna un tip care implementează trăsătura `Error`, fără a specifica tipul exact al valorii de retur. Aceasta oferă flexibilitatea de a returna diferite valori ale erorilor în diferite scenarii de eroare. Cuvântul `dyn` este o abreviere pentru „dinamic” (dynamic).

În al doilea rând, am înlocuit apelul la `expect` cu operatorul `?`, despre care am discutat în [Capitolul 9][ch9-question-mark]<!-- ignore -->. În loc să provoace `panic!` atunci când întâlnește o eroare, `?` va returna valoarea erorii din funcția actuală pentru ca apelantul să o poată gestiona.

În al treilea rând, funcția `run` returnează acum o valoare `Ok` în caz de reușită. În semnătura ei, am declarat tipul de succes al funcției `run` ca fiind `()`, ceea ce ne obligă să închidem valoarea tipului unit într-o valoare `Ok`. Sintagma `Ok(())` poate părea inițial ciudată, dar utilizarea `()` în acest fel este modul convențional de a semnala că apelăm `run` doar pentru efectele sale secundare; nu are o valoare de retur relevantă pentru noi.

La rularea acestui cod, compilarea se va efectua, dar va afișa un avertisment:

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust ne avertizează că am ignorat valoarea `Result` și că această valoare `Result` ar putea semnala că s-a produs o eroare. Nu am verificat însă dacă există sau nu o astfel de eroare, iar compilatorul ne atenționează că, probabil, intenționăm să adăugăm ceva cod pentru gestionarea erorilor! Să corectăm acum acest aspect.

#### Tratarea erorilor returnate de `run` în funcția `main`

Detectăm erorile și le gestionăm printr-o metodă similară cu cea utilizată anterior pentru `Config::build`, în Listarea 12-10, dar cu o subtilă deosebire:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

Optăm pentru `if let` în defavoarea lui `unwrap_or_else` atunci când verificăm dacă `run` dă o valoare de tip `Err` și invocăm `process::exit(1)` în acest caz. Funcția `run` nu returnează o valoare pe care am vrea să-i facem `unwrap`, spre deosebire de ceea ce se întâmplă cu `Config::build`, care ne oferă instanța `Config`. Deoarece `run` returnează `()` când operează cu succes, suntem interesați exclusiv de depistarea unei erori și, prin urmare, nu avem nevoie de `unwrap_or_else` pentru a extrage valoarea neîmpachetată, care ar fi de altfel doar `()`.

Procedura funcțiilor `if let` și `unwrap_or_else` este la fel în ambele contexte: afișăm mesajul de eroare și terminăm execuția.

### Divizarea codului într-un crate de tip bibliotecă

Proiectul nostru `minigrep` se prezintă excelent până acum! Urmează să împărțim conținutul fișierului *src/main.rs* și să transferăm unele porțiuni de cod în fișierul *src/lib.rs*. Astfel, vom putea testa codul și vom avea un fișier *src/main.rs* cu responsabilități reduse.

Să transferăm toate părțile de cod care nu sunt funcția `main` din *src/main.rs* în *src/lib.rs*:

* Definiția funcției `run`
* Declarațiile `use` aplicabile
* Definiția structurii `Config`
* Definiția metodei `Config::build`

Fișierul *src/lib.rs* ar trebui să conțină semnăturile ilustrate în Listarea 12-13 (am omis corpurile funcțiilor pentru rezum). Atenție, acest cod nu va compila până nu facem modificările necesare în *src/main.rs*, așa cum este indicat în Listarea 12-14.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs:here}}
```

<span class="caption">Listarea 12-13: Transferul lui `Config` și `run` în *src/lib.rs*</span>

Am folosit generos cuvântul `pub`: la structura `Config`, la câmpurile și metoda sa `build`, precum și la funcția `run`. De acum, dispunem de un crate de tip bibliotecă cu un API public ce poate fi testat!

Trebuie acum să aducem în domeniul de vizibilitate al crate-ului binar din *src/main.rs* codul pe care l-am mutat în *src/lib.rs*, după cum este ilustrat în Listarea 12-14.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

<span class="caption">Listarea 12-14: Folosirea crate-ului de tip bibliotecă `minigrep` în *src/main.rs*</span>

Introducem linia `use minigrep::Config` pentru a aduce tipul `Config` din crate-ul de tip bibliotecă în sfera de accesibilitate a crate-ului binar și prefixăm funcția `run` cu numele crate-ului nostru. Tot ansamblul de funcționalități ar trebui să fie acum interconectat și să funcționeze corect. Execută programul cu `cargo run` pentru a te asigura că totul merge bine.

Ce muncă intensivă! Dar prin aceasta ne-am pregătit pentru succes pe termen lung. Acum este considerabil mai simplu să gestionăm erorile și am modularizat codul. De acum înainte, majoritatea activității noastre se va concentra în *src/lib.rs*.

Să profităm de această modularitate nou dobândită prin executarea unei sarcini care ar fi fost complicată cu codul vechi dar este simplă cu noul cod: să scriem niște teste!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch17]: ch17-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
