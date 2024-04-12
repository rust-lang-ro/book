## `RefCell<T>` și mutabilitatea interioară

*Mutabilitate interioară* este o metodologie de proiectare în Rust care îți permite să modifici datele chiar dacă există referințe imutabile la acele date; de obicei, această acțiune este restricționată de regulile de împrumut. Pentru a modifica datele, această metodă folosește cod `unsafe` în interiorul unei structuri de date pentru a flexibiliza regulile standard ale Rust privind mutația și împrumutul. Codul `unsafe` semnalează compilatorului că ne asumăm verificarea regulilor manual, în loc să ne bazăm pe compilator să facă acest lucru; vom detalia codul `unsafe` în capitolul 19.

Tipurile care implementează conceptul de mutabilitate interioară pot fi utilizate doar atunci când suntem capabili să garantăm respectarea regulilor de împrumut în timp real, chiar dacă compilatorul nu poate oferi această asigurare. În consecință, codul `unsafe` este învelit într-o interfață API sigură, iar tipul exterior al structurii rămâne imutabil.

Să ne aprofundăm înțelegerea acestui concept prin analiza tipului `RefCell<T>`, care utilizeară mutabilitatea interioară.

### Aplicarea regulilor de împrumut la rulare cu `RefCell<T>`

În contrast cu `Rc<T>`, tipul `RefCell<T>` indică o unică posesiune asupra datelor pe care le deține. Atunci, ce anume diferențiază `RefCell<T>` de un tip ca `Box<T>`? Să revedem regulile de împrumut învățate în Capitolul 4:

* În orice moment, se poate avea *fie* (dar nu și cumulat) o referință mutabilă sau oricâte referințe imutabile.
* Referințele trebuie mereu să fie valide.

Când folosim referințe și `Box<T>`, invarianțele regulilor de împrumut sunt garantate la timpul de compilare. Pentru `RefCell<T>`, aceste invarianțe sunt aplicate *la rulare*. Când încălcăm aceste reguli utilizând referințe, vom întâmpina o eroare de compilator. Cu `RefCell<T>`, dacă regulile sunt încălcate, programul va genera panică și se va opri.

Avantajul verificării regulilor de împrumut în timpul compilării este acela că erorile sunt descoperite mai devreme în ciclul de dezvoltare, și nu influențează performanța la rulare, deoarece toată analiza este finalizată în prealabil. Aceasta este de ce verificarea în momentul compilării este preferată în majoritatea situațiilor și reprezintă implicita alegere în Rust.

Beneficiul verificării regulilor de împrumut în timpul rulării este că acesta permite anumite situații sigure pentru memoria sistemului, ce ar fi fost altfel refuzate de către verificările de compilare. Analiza statică, cum ar fi cea realizată de compilatorul Rust, este de regulă precaută. Iar unele caracteristici ale codului sunt imposibil de identificat prin simpla analiză: cea mai cunoscută problemă fiind Problema Opririi (the Halting Problem), care este dincolo de scopul acestei cărți, însă constituie un subiect fascinant de cercetare.

Deoarece unele analize sunt de neefectuat, dacă compilatorul Rust nu poate fi absolut sigur că un cod respectă regulile de posesiune, există riscul ca acesta să refuze un program corect; o abordare foarte precaută. Dacă Rust ar accepta un cod greșit, încrederea utilizatorilor în promisiunile Rust ar fi subminată. Pe de altă parte, respingerea unui program corect este doar o neplăcere pentru programator, fără consecințe grave. Tipul `RefCell<T>` este valoros atunci când ești convins că regulile de împrumut sunt urmate în codul tău, dar compilatorul nu poate confirma și asigura asta.

Ca și `Rc<T>`, `RefCell<T>` este menit pentru scenarii în care să se opereze pe un singur fir de execuție și va cauza o eroare la compilare dacă încerci să-l folosești într-un context multithreading. Vom discuta cum să accesăm funcționalitățile `RefCell<T>` într-o aplicație multithreading în Capitolul 16.

Aceasta este o recapitulare a motivelor de a alege `Box<T>`, `Rc<T>`, sau `RefCell<T>`:

* `Rc<T>` permite mai mulți posesori pentru aceleași date; `Box<T>` și `RefCell<T>` sunt limitate la un singur posesor.
* `Box<T>` aduce posibilitatea de împrumuturi imutabile sau mutabile verificate în timpul compilării; `Rc<T>` permite doar împrumuturi imutabile verificate în aceeași manieră; `RefCell<T>` oferă împrumuturi imutabile sau mutabile verificate la runtime.
* Fiindcă `RefCell<T>` admite împrumuturi mutabile verificate pe parcursul execuției, poți modifica valoarea din interiorul `RefCell<T>` chiar dacă acesta este imutabil.

A modifica valoarea în interiorul unei variabile imutabile ilustrează modelul de *mutabilitate interioară*. Să investigăm o situație în care mutabilitatea interioară este avantajoasă și să inspectăm cum este posibil acest lucru.

### Mutabilitatea internă: Un împrumut mutabil către o valoare imutabilă

Ca urmare a regulilor de împrumut, atunci când ai o valoare imutabilă, nu poți obține un împrumut mutabil pentru aceasta. De exemplu, codul următor nu va fi compilat:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

Încercând să compilezi codul de mai sus, ai întâmpina următoarea eroare:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

Există, însă, situații în care ar fi de folos ca o valoare să se poată modifica însăși în metodele sale, dar să pară imutabilă din perspectiva altor părți ale codului. Codul din exteriorul metodelor acelei valori nu ar putea să modifice valoarea. Utilizarea `RefCell<T>` reprezintă o modalitate de a avea mutabilitate internă, dar `RefCell<T>` nu evită regulile de împrumut în mod complet: verificatorul de împrumut din compilator permite această mutabilitate internă, dar regulile de împrumut sunt verificate în timpul execuției, nu la compilare. Dacă încalci regulile, vei primi un `panic!` în loc de o eroare de compilare.

Să analizăm un exemplu practic în care utilizăm `RefCell<T>` pentru a muta o valoare imutabilă și să descoperim utilitatea acestui lucru.

#### Un caz practic pentru mutabilitatea internă: Obiectele mock

În timpul procesului de testare, un programator poate utiliza un tip ca substitut pentru altul pentru a analiza anumite comportamente și pentru a confirma că acestea sunt implementate corect. Acest substitut este numit *dublură de testare* (test double). Poți gândi la acesta ca echivalentul unei „dubluri de cascadorie” în filme, unde cineva intervine și joacă rolul unui actor pentru a realiza o scenă complexă. Obiectele *mock* (de imitare) sunt un tip specific de dublură de testare care înregistrează ce se petrece în timpul unui test, permițându-ți să verifici că acțiunile întreprinse sunt cele corecte.

În Rust nu există obiecte în modul tradițional ca în alte limbaje, nici nu dispune de funcționalitate pentru obiecte mock încorporată în librăria standard, ca în alte limbaje. Cu toate acestea, este posibil să creezi o structură care să îndeplinească funcțiile unui obiect mock.

Să ne uităm la scenariul pe care urmează să-l testăm: vom construi o librărie care supraveghează o valoare în comparație cu o valoare maximă și trimite mesaje pe baza proximității valorii sale față de acea valoare maximă. De exemplu, librăria ar putea fi utilizată pentru a monitoriza cota de apeluri API la care un utilizator are acces.

Librăria va oferi exclusiv funcții de monitorizare a distanței față de valoarea maximă și de stabilire a mesajelor care trebuie transmise și în ce momente. Se așteaptă ca aplicațiile care folosesc librăria să implementeze mecanismul de trimitere a acestor mesaje: fie că e vorba de integrarea unui mesaj în aplicație, expedierea unui email, trimiterea unui mesaj text sau orice altă metodă. Nu este necesar ca librăria să fie la curent cu aceste detalii. Tot ce necesită este o implementare a trăsăturii pe care o vom oferi și pe care o numim `Messenger`. Listarea 15-20 ilustrează codul acestei biblioteci:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

<span class="caption">Listarea 15-20: O bibliotecă pentru monitorizarea proximității unei valori față de o valoare maximă și emiterea de avertismente la atingerea anumitor niveluri</span>

Un aspect esențial al acestui cod este faptul că trăsătura `Messenger` are o metodă numită `send`, ce primește o referință imutabilă la `self` și textul mesajului. Această trăsătură constituie interfața pe care mock-ul creat de noi trebuie să o implementeze pentru ca să fie folosit la fel ca un obiect real. A doua Parte importantă este că dorim să testăm comportamentul metodei `set_value` din clasa `LimitTracker`. Putem modifica valoarea parametrului `value`, însă `set_value` nu returnează nimic pe care să ne bazăm aserțiunile. Ne dorim să putem afirma că dacă inițiem un `LimitTracker` cu un element ce implementează trăsătura `Messenger` și o anumită valoare pentru `max`, atunci când furnizăm valori diferite pentru `value`, mesagerul primește instrucțiuni să expediază mesaje corespunzătoare.

Avem nevoie de un mock object care, în loc să trimită un email sau un mesaj text când se apelează metoda `send`, să înregistreze doar mesajele pe care e solicitat să le trimită. Putem crea un exemplar nou al obiectului mock, iniția un `LimitTracker` care folosește acest mock, invoca metoda `set_value` pe `LimitTracker` și apoi să verificăm dacă obiectul mock conține mesajele pe care le anticipăm. Listarea 15-21 prezintă o tentativă de implementare a unui obiect mock în acest sens, însă verificatorul de împrumut nu permite acest lucru:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

<span class="caption">Listarea 15-21: Tentativa de implementare a unui `MockMessenger` refuzată de verificatorul de împrumut</span>

În codul de test prezentat, definim structura `MockMessenger` care include un câmp `sent_messages` cu un `Vec` de `String-uri` menit a ține evidența mesajelor ce urmează a fi expediate. Introducem, de asemenea, funcția asociată `new`, care permite crearea simplă a noilor instanțe `MockMessenger` cu o listă inițială goală de mesaje. Mai departe, punem în aplicare trăsătura `Messenger` pentru `MockMessenger`, astfel încât să putem integra un `MockMessenger` într-un `LimitTracker`. În definiția metodei `send`, încorporăm mesajul primit ca parametru în lista `sent_messages` a `MockMessenger`.

Testul nostru are ca obiectiv să determine comportamentul `LimitTracker`-ului atunci când i se cere să ajusteze `value` la o valoare ce depășește 75% din `max`. Inițial, instanțiem un `MockMessenger` nou, care pornește cu zero mesaje înregistrate. Urmează crearea unui `LimitTracker` la care atașăm o referință spre `MockMessenger` și stabilim `max` la 100. Executăm metoda `set_value` a `LimitTracker`ului cu valoarea 80, ce excede 75% din 100. Confirmăm apoi că lista de mesaje monitorizată de `MockMessenger` ar trebui să conțină acum un mesaj.

Totuși, acest test întâmpină o problemă, așa cum este evidențiat aici:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

Nu putem actualiza `MockMessenger` pentru a evidenția mesajele, pentru că metoda `send` utilizează o referință imutabilă la `self`. De asemenea, nu putem adopta sugestia din mesajul de eroare de a folosi `&mut self`, din motivul că semnătura metodei `send` nu ar mai fi compatibilă cu cea definită în trăsătura `Messenger` (ești încurajat să testezi și să vezi ce mesaj de eroare primești).

Aceasta este o situație în care mutabilitatea internă ne poate veni în ajutor! Vom stoca `sent_messages` într-un `RefCell<T>`, iar apoi metoda `send` va putea modifica `sent_messages` pentru a reține mesajele observate. Listarea 15-22 ne prezintă cum arată acest lucru:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

<span class="caption">Listarea 15-22: Utilizarea `RefCell<T>` pentru modificarea unei valori interne când valoarea externă este considerată imutabilă</span>

Câmpul `sent_messages` este acum de tipul `RefCell<Vec<String>>` în loc de `Vec<String>`. În funcția `new`, inițiem o instanță nouă de `RefCell<Vec<String>>` care încapsulează vectorul gol.

În implementarea metodei `send`, primul parametru este tot un împrumut imutabil al `self`, în conformitate cu definiția trăsăturii. Invocăm `borrow_mut` pe `RefCell<Vec<String>>` din `self.sent_messages` pentru a accesa o referință mutabilă la valoarea din `RefCell<Vec<String>>`, care este vectorul. După aceea, putem apela `push` pe referința mutabilă la vector pentru a ține evidența mesajelor trimise în timpul testului.

Ultima ajustare ce trebuie făcută este la nivelul aserțiunii: pentru a vedea câte elemente sunt în vectorul intern, invocăm `borrow` pe `RefCell<Vec<String>>` pentru a obține o referință imutabilă la vector.

Având o idee generală asupra modului de utilizare a `RefCell<T>`, să explorăm acum în profunzime cum funcționează acesta!

#### Monitorizarea împrumuturilor la execuție cu `RefCell<T>`

Când definim referințe imutabile și mutabile, aplicăm sintaxa `&` și respectiv `&mut`. În cazul folosirii `RefCell<T>`, ne bazăm pe metodele `borrow` și `borrow_mut`, ce reprezintă o parte din API-ul sigur (safe) al `RefCell<T>`. Metoda `borrow` generează pointerul inteligent de tip `Ref<T>`, iar `borrow_mut` produce pointerul inteligent de tip `RefMut<T>`. Având în vedere că ambele tipuri de pointeri implementează `Deref`, putem interacționa cu ei la fel ca și cu referințele convenționale.

`RefCell<T>` contabilizează câți pointeri inteligenți `Ref<T>` și `RefMut<T>` sunt activi la moment. La fiecare apel al metodei `borrow`, `RefCell<T>` crește contorul de împrumuturi imutabile active. Odată cu ieșirea unei valori `Ref<T>` din domeniul de vizibilitate, contorul respectiv scade cu unu. În conformitate cu regulile de împrumut stabilite la compilare, `RefCell<T>` permite existența simultană a mai multor împrumuturi imutabile sau a unui singur împrumut mutabil.

Dacă încălcăm aceste reguli, spre deosebire de obținerea unei erori de compilare, cum ar fi cazul cu referințele standard, implementarea `RefCell<T>` va declanșa o panică la execuție. Listarea 15-23 modifică implementarea metodei `send` prezentată în Listarea 15-22. Demonstrăm intenționat încercarea de a activa două împrumuturi mutabile pentru același domeniu de vizibilitate, pentru a arăta că `RefCell<T>` intervine pentru a preveni acest lucru la timpul execuției.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

<span class="caption">Listarea 15-23: Crearea a două referințe mutabile în același domeniu de vizibilitate pentru a demonstra că `RefCell<T>` va genera panică</span>

Inițializăm variabila `one_borrow` pentru pointerul inteligent `RefMut<T>` returnat de funcția `borrow_mut`. Apoi, inițializăm o nouă împrumutare mutabilă în mod similar în variabila `two_borrow`. Aceasta rezultă în două referințe mutabile în același domeniu de vizibilitate, lucru interzis. Când executăm testele pentru biblioteca noastră, codul din Listarea 15-23 va compila fără greșeli, dar testul nu va reuși:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

Notăm că s-a generat panică cu mesajul `already borrowed: BorrowMutError`. Aceasta este modalitatea prin care `RefCell<T>` gestionează încălcarea regulilor de împrumut în timpul execuției programului.

Alegerea de a prinde erorile de împrumut în timpul execuției, în loc de în timpul compilării, cum am făcut în acest caz, ar putea însemna identificarea greșelilor din cod mai târziu în procesul de dezvoltare: posibil chiar doar după lansarea codului în producție. De asemenea, codul va suporta un mic cost suplimentar de performanță la execuție din cauza urmăririi împrumuturilor în timp real, în loc de în timpul compilării. Însă utilizarea `RefCell<T>` permite scrierea unui obiect mock care se poate modifica pentru a înregistra mesajele pe care le primește, chiar și într-un context unde sunt permise doar valori nealterabile. `RefCell<T>` poate fi folosit, acceptând anumite compromisuri, pentru o funcționalitate sporită față de referințele standard.

### Mai mulți posesori de date mutabile prin combinarea `Rc<T>` cu `RefCell<T>`

O metodă frecventă de a utiliza `RefCell<T>` este în combinație cu `Rc<T>`. Reamintim că `Rc<T>` permite să avem mai mulți proprietari pentru aceleași date, însă ne oferă doar acces imutabil la ele. Dacă deținem un `Rc<T>` ce include un `RefCell<T>`, vom putea avea o valoare care să aibă posesori multipli *și* să fie mutabilă!

De exemplu, rememorăm exemplul cu lista de tip cons prezentat în Listarea 15-18, unde am utilizat `Rc<T>` pentru a permite mai multor liste să partajeze posesiunea unei alte liste. Fiindcă `Rc<T>` permite doar valori imutabile, nu putem altera nicio valoare în listă odată ce acestea au fost create. Adăugând `RefCell<T>`, obținem posibilitatea de a modifica valorile în cadrul listelor. Listarea 15-24 ilustrează că, integrând `RefCell<T>` în definiția `Cons`, putem schimba valoarea depozitată în toate listele:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

<span class="caption">Listarea 15-24: Utilizarea `Rc<RefCell<i32>>` pentru a crea o `Listă` ce poate fi modificată</span>

Construim o valoare care este instanța `Rc<RefCell<i32>>` și o păstrăm într-o variabilă numită `value` pentru a putea fi accesată direct ulterior. Apoi, generăm o `Listă` în `a` cu o variantă `Cons` care conține `value`. Este necesar să clonăm `value` pentru ca atât `a`, cât și `value` să dețină posesiunea asupra valorii interne `5`, în loc să transferăm posesiunea de la `value` la `a` sau ca `a` să împrumute de la `value`.

Încapsulăm lista `a` cu ajutorul `Rc<T>`, astfel încât, când creăm listele `b` și `c`, acestea să se poată referi la `a`, așa cum am făcut în Listarea 15-18.

După ce am format listele `a`, `b` și `c`, dorim să adăugăm 10 la valoarea din `value`. Acest lucru îl realizăm apelând `borrow_mut` pe `value`, care se folosește de funcția de dereferențiere automată prezentată în Capitolul 5 (consulteză secțiunea [„Unde este operatorul `->`?”][wheres-the---operator]<!-- ignore -->) pentru a dereferenția `Rc<T>` la valoarea `RefCell<T>` internă. Metoda `borrow_mut` ne generează un smart pointer `RefMut<T>`, iar noi utilizăm operatorul de dereferențiere pentru a schimba valoarea internă.

La afișarea listelor `a`, `b` și `c`, constatăm că toate prezintă noua valoare modificată de 15, nu cea inițială de 5:
```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

Această abordare este extrem de ingenioasă! Utilizând `RefCell<T>`, avem de-a face cu o valoare de listă `List` care pare imutabilă din exterior. Însă, avem la dispoziție metodele de pe `RefCell<T>` care ne permit accesul la mutabilitatea interioară, astfel încât putem interveni asupra datelor când este necesar. Verificările de runtime privind regulile de împrumut ne apără împotriva conflictelor de date, fiind în anumite cazuri rațional să oferim în schimb o ușoară diminuare a vitezei pentru această flexibilitate adăugată structurilor noastre. E important de reținut că `RefCell<T>` nu funcționează pentru codul executat pe mai multe thread-uri! Alternativa sigură pentru thread-uri la `RefCell<T>` este `Mutex<T>`, pe care o vom explora în Capitolul 16.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
