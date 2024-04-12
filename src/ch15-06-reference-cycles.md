## Ciclurile de referințe pot provoca scurgeri de memorie

Garanțiile oferite de Rust în privința securității memoriei fac dificilă, dar nu imposibilă, crearea accidentală a memoriei ce nu este curățată niciodată (ceea ce se numește *scurgere de memorie*). Rust nu garantează prevenirea totală a scurgerilor de memorie, deci scurgerile de memorie sunt considerate sigure din punct de vedere al securității memoriei în Rust. Vedem că Rust admite scurgerile de memorie când folosim `Rc<T>` și `RefCell<T>`: este posibil să construim referințe care formează un ciclu, unde elementele se referă reciproc. Acest proces determină scurgeri de memorie pentru că numărul de referințe pentru fiecare element din ciclu nu va scădea vreodată la zero, și în consecință, valorile nu vor fi niciodată eliberate.

### Formarea unui ciclu de referințe

Să vedem cum se poate forma un ciclu de referințe și cum să îl evităm, începând cu definiția enum-ului `List` și cu metoda `tail` din Listarea 15-25:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs}}
```

<span class="caption">Listarea 15-25: Definiție pentru o listă de tip cons care include un `RefCell<T>` ce ne permite să modificăm la ce anume se referă o variantă `Cons`</span>

Folosim o variantă modificată a definiției `List` prezentată în Listarea 15-5. Cel de-al doilea element în varianta `Cons` este acum `RefCell<Rc<List>>`, astfel că, în loc să modificăm valoarea `i32` cum am făcut în Listarea 15-24, acum dorim să putem modifica valoarea de tip `List` la care se referă varianta `Cons`. Am inclus și metoda `tail` pentru a facilita accesul la al doilea element atunci când avem de-a face cu o variantă `Cons`.

În Listarea 15-26, introducem funcția `main` care utilizează definițiile din Listarea 15-25. Codul creat generează o listă în variabila `a` și o altă listă în `b` care face referire la lista din `a`. Ulterior, modificăm lista din `a` astfel încât să indice către `b`, formând astfel un ciclu de referințe. Folosim instrucțiuni `println!` pentru a ilustra numărul de referințe la diferite momente ale acestui proces.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

<span class="caption">Listarea 15-26: Construirea unui ciclu de referințe între două valori `List` ce se referă reciproc</span>

Creăm o instanță `Rc<List>` ce conține o valoare de tip `List` în variabila `a` cu o listă inițială de `5, Nil`. Apoi, creăm o nouă instanță `Rc<List>` care păstrează o altă valoare `List` în variabila `b`, aceasta având valoarea 10 și făcând referire la lista din `a`.

Modificăm `a` astfel încât acum să indice spre `b` în loc de `Nil`, creând astfel un ciclu. Realizăm aceasta prin folosirea metodei `tail` pentru a accesa o referință la `RefCell<Rc<List>>` din `a`, pe care o salvăm în variabila `link`. Apoi aplicăm metoda `borrow_mut` pe `RefCell<Rc<List>>` pentru a schimba valoarea dintr-un `Rc<List>` ce conține un `Nil` în `Rc<List>` referențiat de `b`.

La rularea acestui cod, lăsând ultimul `println!` comentat deocamdată, vom primi următorul afișaj:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

Contorul de referințe pentru instanțele `Rc<List>` din `a` și `b` este de 2 după ce am modificat lista din `a` pentru a pointa acum către `b`. La terminarea funcției `main`, Rust renunță la variabila `b`, reducând contorul de referințe al instanței `b` `Rc<List>` de la 2 la 1. Memoria ocupată de `Rc<List>` în heap nu va fi eliberată în acest punct, deoarece contorul său de referințe este 1, nu 0. Apoi Rust renunță la `a`, diminuând contorul de referințe al instanței `a` `Rc<List>` și el de la 2 la 1. Nici memoria acestei instanțe nu poate fi eliberată, deoarece instanța `Rc<List>` rămâne cu referință la ea. Astfel, memoria alocată acestei liste va rămâne colectată pe durată nelimitată. Pentru a vizualiza acest ciclu de referințe, am creat o diagramă în Figura 15-4.

<img alt="Ciclu de referințe al listelor" src="img/trpl15-04.svg" class="center" />

<span class="caption">Figura 15-4: Un ciclu de referințe în care listele `a` și `b` se referă reciproc</span>

Dacă vei decomenta ultimul `println!` și executa programul din nou, Rust va încerca să afișeze acest ciclu cu `a` pointând către `b`, care la rândul său pointează către `a` și așa mai departe, până când va supraîncărca stiva.

Comparativ cu un program din lumea reală, consecințele creării unui ciclu de referințe în acest exemplu nu sunt foarte severe: imediat după ce creăm ciclul de referințe, programul se încheie. Totuși, în cazul unui program mai complex care alocă o cantitate mare de memorie într-un ciclu și o menține pentru mult timp, acesta ar consuma mai multă memorie decât necesar și ar putea suprasolicita sistemul, ceea ce ar duce la epuizarea memoriei disponibile.

Crearea accidentală a ciclurilor de referințe nu este un proces simplu, dar nu este nici imposibil. Dacă utilizezi valori de tip `RefCell<T>` ce conțin valori `Rc<T>` sau combinații tipuri similare cu mutabilitate interioară și numărare de referințe, trebuie să te asigura că nu formezi cicluri; nu te poți baza pe Rust pentru a identifica aceste probleme. Crearea unui ciclu de referințe constituie o eroare de logică în programul tău și ar trebui să utilizezi teste automate, recenzii de cod și alte metodologii de dezvoltare software pentru a le reduce la minim.

O altă soluție pentru evitarea ciclurilor de referințe este reorganizarea structurilor de date astfel încât unele referințe să reprezinte posesiunea, iar altele nu. În consecință, poți avea cicluri constituite din relații de posesiune și relații care nu implică posesiunea, pe când doar relațiile de posesiune influențează posibilitatea de a elibera o valoare. În Listarea 15-25, dorim ca variantele `Cons` să dețină întotdeauna listele lor, astfel încât reorganizarea structurii de date nu este posibilă. Să examinăm un exemplu folosind grafuri alcătuite din noduri părinte și noduri copil pentru a înțelege când relațiile non-posesive sunt o modalitate adecvată pentru prevenirea ciclurilor de referințe.

### Prevenirea ciclurilor de referințe: Convertirea unui `Rc<T>` în `Weak<T>`

Până acum, am arătat că invocarea `Rc::clone` mărește `strong_count` al unei instanțe `Rc<T>`, iar o instanță `Rc<T>` este eliberată din memorie doar când `strong_count` este 0. Poți crea de asemenea o *referință slabă* la valoarea conținută de o instanță `Rc<T>` apelând `Rc::downgrade` și furnizând o referință la `Rc<T>`. Referințele puternice reprezintă o metodă de a partaja posesiunea unei instanțe `Rc<T>`, în timp ce referințele slabe nu reflectă o relație de posesiune și prezența lor nu influențează momentul în care instanța `Rc<T>` este eliberată. Referințele slabe nu conduc la cicluri de referință deoarece orice ciclu care include referințe slabe se desface atunci când numărul referințelor puternice ale valorilor implicate ajunge la 0.

La apelarea `Rc::downgrade`, se obține un pointer inteligent de tip `Weak<T>`. Pe lângă creșterea `strong_count` în instanța `Rc<T>` cu 1 cum se întâmplă prin `Rc::clone`, invocarea `Rc::downgrade` crește `weak_count` cu 1. `Rc<T>` folosește `weak_count` pentru a contoriza câte referințe `Weak<T>` active există, asemeni lui `strong_count` pentru referințele puternice. O diferență majoră este că `weak_count` nu este nevoit să fie 0 pentru ca instanța `Rc<T>` să fie eliberată.

Din moment ce valoarea către care `Weak<T>` referă s-ar putea să fie deja eliberată, pentru a interacționa cu valoarea indicată de un `Weak<T>` e necesar să verifici dacă valoarea încă există. Acesta se face apelând metoda `upgrade` la o instanță de `Weak<T>`, care va returna `Option<Rc<T>>`. Dacă valoarea `Rc<T>` nu a fost eliberată, rezultatul va fi `Some`, în timp ce dacă valoarea `Rc<T>` a fost eliberată vei obține `None`. Cu `upgrade` returnând un `Option<Rc<T>>`, Rust se asigură că cazurile `Some` și `None` sunt adecvat gestionate, evitând existența unui pointer invalid.

Un exemplu ar fi, în loc să utilizăm o listă în care fiecare element știe doar despre următorul element, să construim un arbore unde elementele cunosc atât elementele succesoare care sunt copiii lor cât și elementul anterior care este părintele lor.

#### Crearea unei structuri de date de tip arbore: un `Node` cu noduri copil

Pentru început, vom construi un arbore în care nodurile cunosc nodurile lor copil. Vom crea un struct numit `Node` care conține propria valoare `i32` și referințe către valorile `Node` ale copiilor săi:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

Dorim ca un `Node` să își dețină propriii copii și mai dorim să partajăm această posesiune cu variabile pentru a putea accesa direct fiecare `Node` din arbore. Pentru a realiza acest lucru, definim elementele `Vec<T>` să fie de tip `Rc<Node>`. În plus, dorim să putem modifica care noduri sunt copiii altui nod, astfel încât avem un `RefCell<T>` peste `Vec<Rc<Node>>` în `children`.

În continuare, vom folosi definiția noastră de structură pentru a crea o instanță de tip `Node` denumită `leaf` cu valoarea 3 și fără copii, și altă instanță de tip `Node` numită `branch` cu valoarea 5 și cu `leaf` ca unul dintre copiii săi, așa cum se arată în Listarea 15-27:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

<span class="caption">Listarea 15-27: Crearea unui nod `leaf` fără copii și a unui nod `branch` cu `leaf` ca unul dintre copiii săi</span>

Facem o clonă a `Rc<Node>` din `leaf` și o stocăm în `branch`, astfel încât nodul din `leaf` acum are doi proprietari: `leaf` și `branch`. Putem naviga de la `branch` la `leaf` prin `branch.children`, dar nu există o cale de a merge de la `leaf` la `branch`. Motivul este că `leaf` nu are o referință către `branch` și nu cunoaște relația dintre ele. Vrem ca `leaf` să cunoască că `branch` este părintele său. Asta vom realiza în pasul următor.

#### Adăugarea unei referințe de la un nod copil la părintele său

Pentru a-l face pe nodul copil conștient de părintele său, trebuie să adăugăm un câmp `parent` în definiția noastră a structurii `Node`. Provocarea este să decidem ce tip ar trebui să fie `parent`. Știm că nu poate fi `Rc<T>`, deoarece acesta ar crea un ciclu de referințe cu `leaf.parent` care arată către `branch` și `branch.children` care arată înapoi către `leaf`, fapt ce ar cauza ca valorile lor de `strong_count` să nu ajungă niciodată la 0.

Privind relațiile dintr-o altă perspectivă, un nod părinte ar trebui să își dețină copiii: dacă un nod părinte este șters din memorie, nodurile sale copil ar trebui să fie de asemenea șterse. Cu toate acestea, un copil nu ar trebui să-și dețină părintele: dacă ștergem un nod copil, părintele ar trebui să rămână în existență. Acesta este exact un caz pentru referințele slabe!

Prin urmare, în loc de `Rc<T>`, vom folosi `Weak<T>` pentru tipul câmpului `parent`, mai exact `RefCell<Weak<Node>>`. Cu această schimbare, definiția structurii noastre `Node` acum arată în felul următor:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

Astfel, un nod va putea să facă referire la nodul părinte fără să îl dețină. În Listarea 15-28, actualizăm funcția `main` pentru a folosi această nouă definiție, astfel încât nodul `leaf` să aibă posibilitatea de a se referi la părintele său `branch`:

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

<span class="caption">Listarea 15-28: Un nod `leaf` cu o referință slabă către părintele său `branch`</span>

Procesul de creare a nodului `leaf` este similar cu cel din Listarea 15-27, cu excepția câmpului `parent`: `leaf` începe fără un părinte, deci creăm o instanță nouă și goală a referinței `Weak<Node>`.

Aici, când încercăm să obținem o referință la părintele lui `leaf` folosind metoda `upgrade`, primim o valoare `None`. Acest lucru ni se arată în afișajul de la prima instrucțiune `println!`:

```text
leaf parent = None
```

Atunci când construim nodul `branch`, acesta va avea de asemenea o nouă referință `Weak<Node>` în câmpul `parent`, fiindcă `branch` nu dispune de un nod părinte. Totuși, păstrăm `leaf` ca fiind unul dintre copiii lui `branch`. După ce obținem instanța `Node` în `branch`, putem modifica `leaf` pentru a-i atribui o referință `Weak<Node>` către părintele său. Apelăm metoda `borrow_mut` asupra lui `RefCell<Weak<Node>>` din câmpul `parent` al `leaf` și apoi folosim funcția `Rc::downgrade` pentru a crea o referință `Weak<Node>` către `branch` plecând de la `Rc<Node>` din `branch.`

După ce afișăm din nou părintele lui `leaf`, de data aceasta vom primi o varianta `Some` care include `branch`: acum `leaf` poate să-și acceseze părintele! În momentul în care afișăm `leaf`, evităm și ciclul care a condus anterior la o supraîncărcare de stivă cum a avut loc în Listarea 15-26; referințele `Weak<Node>` sunt afișate ca `(Weak)`:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

Absența unei ieșiri infinite ne indică faptul că acest cod nu a generat un ciclu de referințe. Acest lucru poate fi de asemenea confirmat prin valorile obținute la apelarea funcțiilor `Rc::strong_count` și `Rc::weak_count`.

#### Vizualizarea schimbărilor în `strong_count` și `weak_count`

Să vedem cum valorile `strong_count` și `weak_count` ale instanțelor `Rc<Node>` se modifică prin crearea unui nou domeniu de vizibilitate intern și mutând inițializarea lui `branch` în acest domeniu. Prin aceasta, putem observa ce se întâmplă când `branch` este creat și apoi când este eliberat la ieșirea din domeniu. Modificările sunt ilustrate în Listarea 15-29:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

<span class="caption">Listarea 15-29: Crearea `branch` într-un domeniu de vizibilitate intern și analiza numărului de referințe puternice și slabe</span>

După inițializarea lui `leaf`, `Rc<Node>` aferent are un `strong_count` de 1 și un `weak_count` de 0. În domeniul intern, creăm `branch` și îl asociem cu `leaf`, iar la acel punct, când printăm contoarele, `Rc<Node>` din `branch` va avea un `strong_count` de 1 și un `weak_count` de 1 (datorită lui `leaf.parent` care indică spre `branch` folosind `Weak<Node>`). La printarea contoarelor pentru `leaf`, vom observa că acesta va avea un `strong_count` de 2, deoarece `branch` deține acum un clone al `Rc<Node>` de la `leaf` în `branch.children`, dar `weak_count` va rămâne la 0.

La finalul domeniului intern, `branch` părăsește domeniul de vizibilitate, iar `strong_count` pentru `Rc<Node>` scade la 0, ceea ce duce la eliberarea lui `Node`. Valoarea lui `weak_count` de 1 provenind de la `leaf.parent` nu afectează eliberarea lui `Node`, așadar nu apar scurgeri de memorie!

Dacă încercăm să accesăm părintele lui `leaf` după ce domeniul intern se încheie, vom primi din nou `None`. La finalul programului, `Rc<Node>` în `leaf` va avea un `strong_count` de 1 și un `weak_count` de 0, fiindcă variabila `leaf` este din nou unica referință către `Rc<Node>`.

Toată logica de gestionare a contoarelor și de eliberare a valorilor este inclusă în `Rc<T>` și `Weak<T>`, precum și în implementările acestora ale trăsăturii `Drop`. Prin definirea relației dintre un nod copil și părintele acestuia ca fiind o referință `Weak<T>` în cadrul definiției lui `Node`, este posibil să ai noduri părinte care se referă către noduri copil și invers fără a genera cicluri de referințe și scurgeri de memorie.

## Sumar

Am acoperit în acest capitol cum să exploatați pointerii inteligenți pentru a face unele garanții și compromisuri diferite comparativ cu cele implicite în Rust prin referințele standard. Tipul `Box<T>` are o dimensiune definită și se referă la date aflate pe heap. Tipul `Rc<T>` monitorizează numărul de referințe la datele pe heap, permițând astfel datelor să fie deținute de mai mulți proprietari. `RefCell<T>`, prin mutabilitatea sa internă, ne oferă un tip care este util atunci când avem nevoie de un obiect imutabil, dar dorim să schimbăm o valoare internă a acestuia; totodată, se asigură că regulile de împrumut sunt respectate în timpul execuției, nu în timpul compilării.

Au fost, de asemenea, discutate trăsăturile `Deref` și `Drop`, care fac posibilă o mare parte din funcționalitățile pointerilor inteligenți. Am explorat ciclurile de referință care pot provoca scurgeri de memorie și cum să le prevenim prin utilizarea `Weak<T>`.

Dacă te-a captivat acest capitol și ai vrea să creezi proprii tăi pointeri inteligenți, te încurajăm să consulți [„The Rustonomicon”][nomicon] pentru mai multe informații de valoare.

În capitolul următor, vom explora programarea concurentă în Rust. Și chiar vei avea ocazia să înveți despre careva noi tipuri de pointeri inteligenți.

[nomicon]: ../nomicon/index.html
