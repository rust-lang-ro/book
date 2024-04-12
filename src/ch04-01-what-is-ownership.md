## Ce este posesiunea?

*Posesiunea* reprezintă un set de reguli care guvernează modul în care un program Rust gestionează memoria. Toate programele trebuie să gestioneze modul în care utilizează memoria unui calculator în timp ce rulează. Unele limbaje de programare dispun de colectare a gunoiului (garbage collector), care caută în mod regulat memoria care nu mai este utilizată pe măsură ce programul rulează; în alte limbaje de programare, programatorul trebuie să aloce și să elibereze explicit memoria. Rust folosește o a treia abordare: memoria este gestionată prin intermediul unui sistem de posesiune cu un set de reguli pe care compilatorul le verifică. Dacă oricare dintre reguli sunt încălcate, programul nu va fi compilat. Totuși nici o caracteristică a posesiunii nu va încetini viteza de rulare a programului.

Deoarece posesiunea este un concept nou pentru mulți programatori, este nevoie de ceva timp pentru a te obișnui cu acesta. Vestea bună este că cu cât devii mai experimentat cu Rust și cu regulile sistemului de posesiune, cu atât îți va fi mai ușor să dezvolți în mod natural cod care este sigur și eficient. Continuă să o faci!

Odată ce vei înțelege posesiunea, vei avea o bază solidă pentru înțelegerea caracteristicilor ce fac din Rust un limbaj unic. În acest capitol vei învăța despre posesiune prin câteva exemple care se concentrează asupra unei structuri de date foarte comune: string-urile.

> ### Stiva și heap-ul
>
> Multe limbaje de programare nu cer să te gândești prea mult la stivă și heap.
> Dar într-un limbaj de programare pentru sisteme ca Rust, în dependență dacă o
> valoare se află pe stivă sau heap afectează modul în care limbajul se
> comportă și necesită să iei anumite decizii. Părți din noțiunea de posesiune
> în raport cu stiva și heap-ul vor fi descrise mai târziu în acest capitol,
> așa că iată o scurtă explicație anticipată.
>
> Atât stiva cât și heap-ul sunt părți ale memoriei disponibile codului tău
> pentru a fi folosite la runtime, dar care sunt structurate în moduri
> diferite. Stiva stochează valorile în ordinea în care le primește și elimină
> valorile în ordinea inversă. Acest lucru este menționat ca
> *ultimul intrat, primul ieșit*. Gândește-te la o stivă de farfurii: când
> adaugi mai multe farfurii, le pui în vârful grămezii, și când ai nevoie de o
> farfurie, o iei de pe vârful grămezii. Adăugarea sau eliminarea de farfurii
> de la mijloc sau de jos nu ar funcționa la fel de bine! Adăugarea de date se
> numește *împingerea pe stivă* și eliminarea de date se numește
> *scoaterea de pe stivă*. Toate datele stocate pe stivă trebuie să aibă o
> dimensiune cunoscută și fixă. Datele cu o dimensiune necunoscută în momentul
> compilării sau o dimensiune care s-ar putea schimba trebuie să fie stocate pe
> heap.
>
> Heap-ul este mai puțin organizat: când pui date pe heap, ceri o anumită
> cantitate de spațiu. Alocatorul de memorie găsește un loc liber în heap care
> este suficient de mare, îl marchează ca fiind în uz și returnează un
> *pointer*, care este adresa acelui loc. Acest proces se numește
> *alocare pe heap* și este uneori abreviat drept doar *alocare* (împingerea
> valorilor pe stivă nu este considerată alocare). Deoarece pointer-ul către
> heap are o dimensiune cunoscută, fixă, poți stoca pointer-ul pe stivă, dar
> când vrei datele efective, trebuie să urmezi pointer-ul. Gândește-te că te-ai
> așezat la un restaurant. Când intri, precizezi numărul de persoane din grupul
> tău, iar gazda găsește o masă liberă care se potrivește tuturor și te conduce
> acolo. Dacă cineva din grupul tău vine târziu, poate întreba unde ai fost
> așezat să te găsească.
>
> Împingerea pe stivă este mai rapidă decât alocarea pe heap deoarece
> alocatorul nu trebuie să caute niciodată un loc unde să stocheze date noi;
> acel loc este întotdeauna în partea de sus a stivei. Comparativ, alocarea
> spațiului pe heap necesită mai multă muncă deoarece alocatorul trebuie mai
> întâi să găsească un spațiu suficient de mare pentru a susține datele și apoi
> să efectueze o evidență a datelor acum alocate pentru a se pregăti pentru
> următoarea alocare.
>
> Accesarea datelor în heap este mai lentă decât accesarea datelor pe stivă
> deoarece trebuie să urmezi un pointer pentru a ajunge acolo. Procesoarele
> contemporane sunt mai rapide dacă sar mai puțin prin memorie. Continuând
> analogia, ia în considerare un chelner la un restaurant care ia comenzi de la
> mai multe mese. Este mai eficient să obții toate comenzile de la o masă
> înainte de a trece la următoarea. Luând o comandă de la masa A, apoi o
> comandă de la masa B, apoi una de la A din nou, și apoi din nou una din B ar
> fi un proces mult mai lent. Din același motiv, un procesor își poate face
> treaba mai bine dacă lucrează pe date care sunt apropiate una de alta (așa
> cum este pe stivă) și mai lent când sunt răzlețite (așa cum poate fi pe
> heap).
>
> Când codul tău apelează o funcție, valorile transmise în funcție (inclusiv,
> potențial, pointeri către date pe heap) și variabilele locale ale funcției
> sunt introduse pe stivă. Când funcția se încheie, aceste valori sunt
> eliminate de pe stivă.
>
> Urmărirea care părți ale codului folosesc ce date pe heap, minimizarea
> cantității de date duplicate pe heap și eliberarea datelor neutilizate de pe
> heap astfel încât să nu rămâi fără spațiu sunt toate probleme pe care
> posesiunea le abordează. Odată ce înțelegi posesiunea, nu va trebui să te
> gândești prea des la stivă și la heap, dar știind că scopul principal al
> posesiunii este de a gestiona datele de pe heap poate ajuta la explicarea
> modului în care aceasta funcționează.

### Regulile posesiunii

În primul rând, să aruncăm o privire asupra regulilor posesiunii. Ține minte aceste reguli în timp ce lucrăm la exemplele care le ilustrează:

* Fiecare valoare în Rust are un *posesor*.
* Poate exista doar un singur posesor la un moment dat.
* Când posesorul iese din domeniul de vizibilitate (eng. out of scope), valoarea va fi eliminată.

### Domeniu de vizibilitate a variabilelor

Acum că am trecut de baza sintaxei Rust, nu vom include în toate exemplele codul `fn main() {`, astfel că în continuare asigură-te că introduci manual exemplele ce vor urma într-o funcție `main`. Ca rezultat exemplele noastre vor fi ceva mai concise, permițându-ne să ne concentrăm pe detaliile esențiale, fără prea mult cod de umplutură.

Ca prim exemplu de posesiune vom analiza *domeniul de vizibilitate* al unor variabile. Un domeniu de vizibilitate este intervalul în cadrul unui program în care un element este valid. Să luăm în considerare următoarea variabilă:

```rust
let s = "hello";
```

Variabila `s` se referă la un string literal, unde valoarea string-ului este inclusă direct în textul programului nostru. Variabila este validă de la punctul în care este declarată până la sfârșitul domeniului de vizibilitate curent. Listarea 4-1 arată un program cu comentarii care anunță unde ar fi validă variabila `s`.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

<span class="caption">Listarea 4-1: O variabilă și domeniul de vizibilitate în care este
validă</span>

Cu alte cuvinte, există două momente importante aici:

* Când `s` intră *în* domeniul de vizibilitate, este validă.
* Rămâne validă până când iese *din* domeniul de vizibilitate.

La acest moment, relația dintre domeniile de vizibilitate și momentele în care variabilele sunt valide este similară cu cea din alte limbaje de programare. Acum să continuăm studierea relațiilor de posesiune în Rust cu introducerea tipului `String`.

### Tipul `String`

Pentru a ilustra regulile de posesiune, avem nevoie de un tip de date mai complex decât cele pe care le-am descris în secțiunea [„Tipuri de date”][data-types] a Capitolului 3. Tipurile menționate anterior sunt de dimensiune cunoscută, deci pot fi stocate în stivă și eliminate din stivă când domeniul lor de vizibilitate se termină, și pot fi copiate rapid și trivial pentru a crea o nouă instanță independentă în cazul în care o altă parte a codului are nevoie să folosească aceeași valoare, dar într-un alt domeniu de vizibilitate. Însă noi vrem să accesăm datele care sunt stocate în heap și să explorăm cum anume Rust știe când să elibereze acele date, iar tipul `String` este un exemplu excelent.

Acum ne vom concentra doar asupra aspectelor tipului `String` care sunt relevante posesiunii. Aceste aspecte se aplică și altor tipuri de date complexe, fie că sunt furnizate de biblioteca standard sau create de tine. Mai pe larg vom discuta despre `String` în [Capitolul 8][ch8].

Am văzut deja literali ai tipului string, în care o valoare string este codificată direct în programul nostru. Literalii de string sunt convenabili, dar nu sunt potriviți pentru toate situațiile în care am dori să folosim text. Un motiv este că sunt imutabili. Altul este că nu fiecare valoare de string poate fi cunoscută din timp când scriem codul: de exemplu, ce se întâmplă dacă dorim să preluăm inputul de la utilizator și să îl stocăm? Pentru aceste situații Rust are un al doilea tip de string, `String`. Acest tip administrează date alocate în heap și astfel este capabil să stocheze un text care nu ne este cunoscut la momentul compilării. Poți crea un `String` dintr-un literal de string folosind funcția `from`, astfel:

```rust
let s = String::from("hello");
```

Operatorul cu dublu două puncte `::` ne permite să definim această funcție `from` în spațiul de nume al tipului `String` în loc să o numim cumva cum ar fi `string_from`. Vom discuta mai mult această sintaxă în secțiunea [„Sintaxa metodei”][method-syntax] din Capitolul 5, și când vorbim despre spațiul de nume cu module în [„Căi pentru referirea la un element în arborele modulelor”][paths-module-tree] din Capitolul 7.

Acest tip de string *poate* fi mutat:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

Deci, care este diferența aici? De ce poate fi `String` mutat, dar literalii nu? Diferența o constituie felul în care aceste două tipuri sunt reprezentate în memorie.

### Memorie și alocare

În cazul unui literal de string, cunoaștem conținutul în timpul compilării, astfel încât textul este codificat direct în executabilul final. Din acest motiv, literalele de string sunt rapide și eficiente. Dar aceste proprietăți provin doar din imutabilitatea literalului de string. Din păcate, nu putem pune un bloc de memorie în binar pentru fiecare fragment de text a cărui dimensiune este necunoscută în timpul compilării și a cărui mărime s-ar putea schimba în timpul rulării programului.

Cu tipul `String`, pentru a menține o bucată de text mutabilă, în creștere, trebuie să alocăm o cantitate de memorie pe heap, necunoscută în timpul compilării, pentru a deține conținutul. Aceasta înseamnă:

* Memoria trebuie solicitată de la alocatorul de memorie la runtime.
* Avem nevoie de o modalitate de a returna această memorie la alocator atunci când am terminat cu `String-ul` nostru.

Prima parte este realizată de noi: când apelăm `String::from`, implementarea sa solicită memoria de care are nevoie. Aceasta parte este comună pentru mai toate limbajele de programare.

Cu toate acestea, a doua parte este diferită. În limbajele cu un *colector de gunoi* (garbage collector (GC)), GC ține evidența și eliberează memoria care nu mai este folosită, programatorul nu mai având necesitatea de a se gândi la asta. Pe când în majoritatea limbajelor fără un GC e responsabilitatea noastră să identificăm când memoria nu mai este folosită și să apelăm codul pentru a o elibera explicit, la fel cum am solicitat-o. A face acest lucru corect este, de obicei, o problemă dificilă de programare. Dacă uităm, vom irosi memorie. Dacă o facem prea devreme, vom avea o variabilă nevalidă. Dacă o facem de două ori tot este o problemă. Trebuie să menținem perechi de exact o `alocare` cu exact o `dealocare`.

Rust alege o cale diferită: memoria este returnată automat odată ce variabila care o deține iese din domeniu de vizibilitate. Iată o versiune a exemplului nostru despre domeniu de vizibilitate de la Listarea 4-1 folosind un `String` în loc de un literal de string:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

Există un punct natural la care putem returna memoria ocupată de `String-ul` nostru la alocator: atunci când `s` iese din domeniu de vizibilitate. Când o variabilă iese din domeniu de vizibilitate, Rust apelează o funcție specială. Această funcție se numește [`drop`][drop]<!-- ignore -->, și este chiar locul unde autorul `String`-ului poate pune codul de returnare a memoriei. Rust apelează `drop` automat la închiderea acoladei.

> Notă: În C++, acest model de dealocare a resurselor la sfârșitul duratei de
> viață a unui element se numește uneori 
> *Resource Acquisition Is Initialization (RAII)*. Funcția `drop` din Rust îți
> va fi familiară dacă ai folosit modele RAII.

Felul acesta de operare a memoriei are un impact profund asupra modului în care este scris codul Rust. Poate părea simplu acum, dar comportamentul codului poate fi destul de neașteptat în situații mai complicate, atunci când avem mai multe variabile care utilizează datele pe care le-am alocat pe heap. Să explorăm acum unele așa situații.

<!-- Vechiul titlu. Nu îl eliminați sau link-urile s-ar putea strica. -->
<a id="ways-variables-and-data-interact-move"></a>

#### Variabile și interacționarea cu date folosind permutarea

Variabilele pot interacționa cu aceleași date în diferite moduri în Rust. Să privim la un exemplu utilizând un întreg în Lista 4-2.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

<span class="caption">Lista 4-2: Atribuirea valorii întregului de la variabila `x`
la `y`</span>

Probabil putem ghici ce face acest cod: „atribuie valoarea `5` lui `x`; apoi face
o copie a valorii din `x` și o atribuie lui `y`.” Acum avem două variabile, `x`
și `y`, și ambele sunt egale cu `5`. Așa și este, deoarece întregii în Rust sunt valori simple cu o mărime cunoscută, fixă, și aceste două valori `5` sunt împinse pe stivă.

Acum să ne uităm la versiunea cu un `String`:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

Aceasta arată foarte asemănător, printr urmare s-ar putea presupune că modul în care funcționează ar fi la fel: adică, a doua linie ar face o copie a valorii din `s1` și o atribuie lui `s2`. Doar că de data aceasta nu e chiar așa.

Priviți la Figura 4-1 pentru a vedea ce se întâmplă cu `String` în fundal. Un `String` este format din trei părți, arătate în stânga: un pointer spre memoria care deține conținutul string-ului, o lungime și o capacitate. Acest grup de date este stocat pe stivă. Pe dreapta e reprezentată memoria de pe heap care deține conținutul.

<img alt="Două tabele: prima tabelă conține reprezentarea s1 pe stivă,
constând din lungimea sa (5), capacitatea (5) și un pointer către prima
valoare din cea de-a doua tabelă. Cea de-a doua tabelă conține reprezentarea
datelor string-ului pe heap, byte cu byte." src="img/trpl04-01.svg" class="center"
style="width: 50%;" />

<span class="caption">Figura 4-1: Reprezentare în memorie a unui `String`
care deține valoarea `"hello"` și este atribuită lui `s1`</span>

Lungimea reprezintă câtă memorie, în octeți, folosește în prezent conținutul `String`-ului. Capacitatea este cantitatea totală de memorie, în octeți, pe care `String`-ul a primit-o de la alocator. Diferența între lungime și capacitate contează, dar nu în acest context, deci, pentru moment ignorăm capacitatea.

Când atribuim `s1` la `s2`, datele din `String` sunt copiate, însemnând că noi copiem pointer-ul, lungimea și capacitatea care sunt pe stivă. Noi nu copiem
datele de pe heap pe care pointer-ul le referă. Cu alte cuvinte, reprezentarea datelor în memorie arată ca Figura 4-2.

<img alt="Trei tabele: tabelele s1 și s2 reprezentând acele string-uri pe stivă,
respectiv, și amândouă adresând aceleași date din string de pe heap."
src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-2: Reprezentare în memorie a variabilei `s2`
care are o copie a pointer-ului, lungimii și capacității lui `s1`</span>

Reprezentarea *nu* e ca în figura 4-3, ea prezintă cum ar fi arătat memoria dacă Rust copia *și* datele din heap. Dacă Rust ar face acest lucru, operațiunea `s2 = s1` ar deveni foarte costisitoare în termeni de performanță la execuție, mai ales când datele de pe heap sunt  mari.

<img alt="Patru tabele: doua tabele reprezentând datele de pe stivă pentru s1 și s2,
și fiecare punctează la propria copie a datelor de string de pe heap."
src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-3: O altă posibilitate pentru ce ar putea face `s2 = s1`
dacă Rust ar copia de asemenea datele de pe heap</span>

Mai devreme, am spus că atunci când o variabilă iese din domeniul de vizibilitate, Rust apelează automat funcția `drop` și curăță memoria heap pentru acea variabilă. Dar Figura 4-2 arată ambele pointer-e de date adresând aceeași locație. Ar fi o problemă: când `s2` și `s1` ies din domeniul de vizibilitate, vor încerca amândouă să elibereze aceeași memorie. Acest lucru este cunoscut ca eroarea de *eliberare dublă* (double free) și este una dintre erorile de securitate a memoriei pe care le-am menționat anterior. Eliberarea memoriei de două ori poate duce la coruperea memoriei, ceea ce potențial poate provoca vulnerabilități de securitate.

Pentru a asigura securitatea memoriei, după linia `let s2 = s1;`, Rust consideră că `s1` nu mai este valid. Prin urmare, Rust nu trebuie să elibereze nimic atunci când `s1` iese din domeniul de vizibilitate. Să vedem ce se întâmplă când încercăm să folosim `s1` după ce `s2` este creat; nu va funcționa:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

Vom primi o eroare similară, deoarece Rust ne împiedică să utilizăm referința invalidată:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/output.txt}}
```

Dacă ai auzit termenii *copiere superficială* (shallow copy) și *copiere profundă* (deep copy) în timp ce lucrezi cu alte limbaje, conceptul de copiere a pointerului, lungimii și capacității fără a copia datele probabil sună ca făcând o copiere superficială. Dar deoarece Rust de asemenea invalidează prima variabilă, în loc să fie numită o copiere superficială, este cunoscută ca o *permutare*. În acest exemplu, am spune că `s1` a fost *permutată* în `s2`. Deci, ceea ce se întâmplă de fapt este ilustrat în Figura 4-4.

<img alt="Trei tabele: tabelele s1 și s2 reprezentând acele string-uri pe stivă,
respectiv, și ambele adresând aceleași date de string de pe heap.
Tabelul s1 este umbrit deoarece s1 nu mai este valid; doar s2 poate fi folosit 
pentru a accesa datele de pe heap." src="img/trpl04-04.svg" class="center" style="width:
50%;" />

<span class="caption">Figura 4-4: Reprezentarea în memorie după ce `s1` a fost
invalidat</span>

Aceasta rezolvă problema noastră! Cu doar `s2` validă, atunci când iese din domeniul de vizibilitate variabila va elibera singură memoria.

În plus, rezultă un fapt important pentru design-ul limbajului: Rust nu va crea niciodată automat o „copiere profundă” a datelor noastre. Prin urmare, orice *copiere automată* poate fi presupusă a fi ieftină în ceea ce privește performanța la runtime.

<!-- Old heading. Do not remove or links may break. -->
<a id="ways-variables-and-data-interact-clone"></a>

#### Variabile și interacționarea cu date folosind clonarea

Dacă vrem să copiem *în profunzime* datele unui `String` din heap, nu doar datele din stivă, putem folosi o metodă des întâlnită, numită `clone`. Vom discuta sintaxa metodelor în Capitolul 5, însă deoarece metodele sunt o caracteristică comună în multe limbaje de programare, probabil le-ai întâlnit deja.

Iată un exemplu al utilizării metodei `clone`:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

Acest lucru funcționează foarte bine și produce în mod explicit comportamentul prezentat în Figura 4-3, unde datele din heap *sunt* copiate.

Când vezi un apel către `clone`, știi că se execută un anumit cod arbitrar și că acest cod poate fi costisitor. Este un indicator vizual că se întâmplă ceva diferit.

#### Date doar pe stivă: trăsătura Copy

Există o altă nuanță despre care nu am discutat încă. Acest cod care folosește numere întregi, o parte din el a fost arătată în Listarea 4-2, funcționează și este valid:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

Dar acest cod pare să contrazică ceea ce tocmai am învățat: nu avem un apel la `clone`, dar `x` este încă valid și nu a fost permutat în `y`.

Motivul este că unele tipuri, cum ar fi numerele întregi care au o dimensiune cunoscută la compilare, sunt stocate în întregime pe stivă, astfel încât copierea valorilor lor este foarte rapidă. Astfel nu avem niciun motiv să vrem să prevenim `x` de a fi valid după ce am creat variabila `y`. Cu alte cuvinte, nu există nicio diferență între copierea profundă și cea superficială aici, deci apelarea la `clone` nu ar face nimic diferit de copierea superficială obișnuită, așa că o putem lăsa pentru alte cazuri.

Rust are o adnotare specială numită trăsătura `Copy` pe care o putem aplica la tipuri care sunt stocate pe stivă, la fel ca numerele întregi (vom vorbi mai multe despre trăsături în [Capitolul 10][traits]<!-- ignore -->). Dacă un tip implementează trăsătura `Copy`, variabilele care îl utilizează nu se permută, ci sunt copiate superficial, lucru care le păstrează valide după atribuirea lor altei variabile.

Rust nu ne va lăsa să adnotăm un tip cu `Copy` dacă tipul, sau oricare dintre părțile sale, a implementat trăsătura `Drop`. Dacă tipul are nevoie de ceva special să se întâmple când valoarea iese din domeniul de vizibilitate și adăugăm adnotarea `Copy` la acel tip, vom primi o eroare la compilare. Pentru a afla cum să adăugați adnotarea `Copy` la tipul tău pentru a implementa trăsătura, vedeți [“Trăsături derivate”][derivable-traits]<!-- ignore --> în Anexa C.

Deci, ce tipuri implementează trăsătura `Copy`? Puteți verifica documentația pentru
tipul dat pentru a fi sigur, dar ca o regulă generală, orice grup de valori scalare simple
poate implementa `Copy`, și nimic care necesită alocare sau este o formă de resursă nu poate implementa `Copy`. Iată câteva tipuri care 
implementează `Copy`:

* Toate tipurile de numere întregi, cum ar fi `u32`.
* Tipul Boolean, `bool`, cu valorile `true` și `false`.
* Toate tipurile de numere în virgulă mobilă, cum ar fi `f64`.
* Tipul de caractere, `char`.
* Tuple, dacă acestea conțin numai tipuri care de asemenea implementează `Copy`. De exemplu,  `(i32, i32)` implementează `Copy`, dar `(i32, String)` nu.

### Funcțiile și posesiunea

Transmiterea unei valori către o funcție e similară cu atribuirea acelei valori unei variabile. A transmite o variabilă către o funcție implică fie permutarea, fie copierea acelei valori, exact așa cum se întâmplă și la atribuire. În Lista 4-3, am pregătit un exemplu cu adnotări pentru a ilustra când intră și ies variabilele din domeniul de vizibilitate.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-03/src/main.rs}}
```

<span class="caption">Listarea 4-3: Funcții cu posesiunea și domeniul de vizibilitate adnotat</span>

Dacă am încercat să utilizăm `s` după apelul la `takes_ownership`, Rust ar genera o eroare la compilare. Aceste verificări statice ne protejează de greșeli. Încearcă să adaugi cod la `main` care folosește `s` și `x` pentru a vedea unde le poți folosi și unde regulile de posesiune te împiedică să faci asta.

### Valorile returnate și domeniul de vizibilitate

Returul valorilor poate transmite, de asemenea, posesiunea. Listarea 4-4 ilustrează un exemplu de funcție care returnează o valoare, folosind adnotări similare cu cele din Listarea 4-3.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

<span class="caption">Listarea 4-4: Transferul posesiunii prin valorile returnate</span>

Posesiunea unei variabile urmează același model de fiecare dată: atribuirea unei valori unei alte variabile determină permutarea acesteia. Când variabila, care include date pe heap, iese din domeniul de vizibilitate, valoarea este eliberată prin funcția `drop`, cu excepția situației în care posesiunea datelor a fost permutată către o altă variabilă.

Această abordare este funcțională, dar preluarea posesiunii și ulterior returnarea acesteia cu fiecare funcție se poate dovedi a fi un proces anevoios. Ce facem dacă vrem ca o funcție să utilizeze o valoare, fără a-i lua posesiunea? Este laborios faptul că tot ce trimitem în interiorul unei funcții trebuie să fie returnat înapoi dacă dorim să-l utilizăm din nou, în plus față de orice rezultate obținute în corpul funcției pe care am dori să le returnăm.

Rust ne oferă posibilitatea de a returna mai multe valori prin utilizarea unei tuple, așa cum este ilustrat în Listarea 4-5.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-05/src/main.rs}}
```

<span class="caption">Listarea 4-5: Returnarea posesiunii parametrilor</span>

Însă acest proces este prea laborios și complex pentru un concept ce ar trebui să fie simplu. Din fericire pentru noi, Rust dispune de o funcționalitate ce ne permite să folosim o valoare fără a-i transfera posesiunea, funcționalitate cunoscută sub numele de *referințe*.

[data-types]: ch03-02-data-types.html#data-types
[ch8]: ch08-02-strings.html
[traits]: ch10-02-traits.html
[derivable-traits]: appendix-03-derivable-traits.html
[method-syntax]: ch05-03-method-syntax.html#method-syntax
[paths-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[drop]: ../std/ops/trait.Drop.html#tymethod.drop
