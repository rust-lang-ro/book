## Utilizarea obiectelor-trăsătură pentru valori de diferite tipuri

Am discutat în Capitolul 8 faptul că vectorii au limitarea de a putea stoca elemente de un singur tip. O soluție de compromis a fost prezentată în Listarea 8-9, unde am definit o enumerare `SpreadsheetCell` cu variante pentru a suporta întregi, numere în virgulă mobilă și text. Acest lucru ne permitea să depozităm date de diverse tipuri în fiecare celulă, păstrând în același timp un vector ce reprezenta un rând de celule. Această modalitate este ideală când elementele interschimbabile din setul nostru sunt de tipuri fixe ce sunt cunoscute la momentul compilării codului.

Totuși, în anumite cazuri dorim ca utilizatorii bibliotecii noastre să aibă posibilitatea de a extinde setul de tipuri permise într-un context dat. Pentru a exemplifica cum s-ar realiza asta, vom construi un exemplu de unealtă GUI (interfață grafică cu utilizatorul) care parcurge o listă de elemente, invocând metoda `draw` pe fiecare pentru a le reda pe ecran, o abordare obișnuită în cadrul uneltelor GUI. Vom constitui un crate de tip bibliotecă denumit `gui`, care va conține structura de bază a unei astfel de biblioteci. Acest crate ar include tipuri de bază pentru folosire, precum `Button` sau `TextField`. În plus, utilizatorii `gui` vor dori să introducă tipuri proprii ce pot fi redate: de pildă, un programator poate adăuga o `Image`, iar altul un `SelectBox`.

Deși nu vom dezvolta o bibliotecă GUI completă în acest exemplu, vom ilustra modul în care componentele ar interacționa. La momentul creării bibliotecii, nu putem anticipa toate tipurile pe care alți programatori le-ar putea defini. Ceea ce știm este că `gui` trebuie să gestioneze diverse valori de tipuri diferite și să invoce metoda `draw` pentru fiecare dintre acestea. Nu trebuie să cunoaștem detaliile specifice ale acțiunii metodei `draw`, ci doar sa fim siguri că această metodă va fi disponibilă pentru a fi apelată.

Pentru a realiza aceasta într-un limbaj cu moștenire, am defini o clasă denumită `Component` care ar include o metodă numită `draw`. Alte clase, cum ar fi `Button`, `Image` și `SelectBox`, ar moșteni din `Component` și, astfel, ar moșteni metoda `draw`. Ele ar putea să realizeze o supraîncărcare a metodei `draw` pentru a defini comportamentul lor specific, dar biblioteca noastră ar putea considera toate aceste instanțe ca fiind de tip `Component` și ar putea apela `draw`. Cu toate acestea, fiindcă Rust nu suportă moștenirea, avem nevoie de o altă metodă de structurare a bibliotecii `gui` astfel încât utilizatorii să poată să o extindă cu noi tipuri.

### Definirea unei trăsături pentru comportament comun

Pentru a implementa comportamentul dorit pentru `gui`, definim o trăsătură denumită `Draw` ce conține o metodă numită `draw`. Vom putea apoi defini un vector care primește un *obiect-trăsătură*. Un astfel de obiect-trăsătură se referă la o instanță a unui tip care implementează o trăsătură specificată și la un tabel folosit pentru a identifica metodele trăsăturii pe respectivul tip în timpul rulării. Creăm un obiect-trăsătură prin indicarea unui tip de pointer - de exemplu, o referență `&` sau un pointer inteligent `Box<T>` - urmat de cuvântul cheie `dyn`, și apoi trăsătura relevantă. (Cauza pentru care obiectele-trăsătură necesită un pointer este discutată în Capitolul 19, în secțiunea [„Tipuri cu dimensiune dinamică și trăsătura `Sized`.”][dynamically-sized]<!-- ignore -->) Obiectele-trăsătură pot fi folosite în loc de tipuri generice sau concrete. Oriunde sunt utilizate, sistemul de tipuri din Rust garantează la momentul compilării că orice valoare din acel context va implementa trăsătura specificată de obiectul-trăsătură. Astfel, nu este nevoie să cunoaștem toate tipurile posibile la momentul compilării.

Am menționat anterior că, în Rust, evităm să numim structurile și enumerările "obiecte" pentru a sublinia diferența față de conceptul de obiect din alte limbaje de programare. Diferența constă în faptul că, în Rust, datele din câmpurile structurii și comportamentul definit în blocurile `impl` sunt separate, în timp ce în alte limbaje combinația de date și comportament formează adesea ceea ce se numește obiect. Cu toate acestea, obiectele-trăsătură *sunt* asemănătoare cu obiectele din alte limbaje prin fuziunea dintre date și comportament. Totuși, ele se disting de obiectele tradiționale prin faptul că nu permit adăugarea de date suplimentare la un obiect-trăsătură. Obiectele-trăsătură nu prezintă aceeași utilitate generală ca obiectele din alte limbaje, fiind special concepute pentru abstractizarea unui comportament comun.

Listarea 17-3 ilustrează modul de definire a unei trăsături numite `Draw` cu o metodă `draw`:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-03/src/lib.rs}}
```

<span class="caption">Listarea 17-3: Definiția trăsăturii `Draw`</span>

Această sintaxă ar trebui să fie deja cunoscută datorită explicațiilor noastre din Capitolul 10 despre cum se definesc trăsăturile. Acum urmează o nouă sintaxă: Listarea 17-4 introduce o structură numită `Screen` care posedă un vector denumit `components`. Acest vector este de tip `Box<dyn Draw>`, fiind un obiect-trăsătură; serving ca substitut pentru orice tip dintr-un `Box` ce implementează trăsătura `Draw`.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-04/src/lib.rs:here}}
```

<span class="caption">Listarea 17-4: Definirea structurii `Screen` cu un câmp `components` ce deține un vector de obiecte-trăsătură implementând trăsătura `Draw`</span>

În structura `Screen`, vom defini o metodă numită `run` care va invoca metoda `draw` pentru fiecare dintre componente, așa cum este demonstrat în Listarea 17-5:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-05/src/lib.rs:here}}
```

<span class="caption">Listarea 17-5: Metoda `run` pentru `Screen` ce apelează metoda `draw` pe fiecare component în parte</span>

Aceasta metodă se comportă diferit de cazul în care am defini o structură care utilizează un parametru de tip generic cu delimitări de trăsătură. Un parametru de tip generic poate fi înlocuit numai cu un singur tip concret o singură dată, în timp ce obiectele-trăsătură permit folosirea mai multor tipuri concrete ca substituenți pentru obiectul-trăsătură în timpul execuției. Spre exemplu, am fi putut defini structura `Screen` utilizând un tip generic și o delimitare de trăsătură, după cum se vede în Listarea 17-6:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-06/src/lib.rs:here}}
```

<span class="caption">Listarea 17-6: O versiune alternativă de implementare a structurii `Screen` și a metodei sale `run` folosind generici și delimitări de trăsături</span>

Acest lucru ne limitează la o instanță `Screen` care ar deține o listă de componente de același tip, fie ele `Button` sau `TextField`. Dacă vei avea mereu colecții omogene, utilizarea tipurilor generice și a delimitărilor de trăsături este mai avantajoasă deoarece definițiile sunt monomorfizate în timpul compilării pentru a utiliza tipurile concrete.

Pe de altă parte, utilizând metoda cu obiectele-trăsătură, o instanță `Screen` poate conține un `Vec<T>` care include atât `Box<Button>` cât și `Box<TextField>`. Să analizăm cum funcționează acest mecanism și apoi vom discuta despre performanța lui în timpul execuției.

### Implementarea trăsăturii

Acum, vom introduce câteva tipuri care implementează trăsătura `Draw`. Să oferim tipul `Button`. Deoarece implementarea unei biblioteci GUI complete nu este acoperită de această carte, metoda `draw` va avea un corp fără o implementare concret utilă. Pentru a ne imagina cum ar arăta o posibilă implementare, un struct `Button` ar putea include câmpuri precum `width` (lățime), `height` (înălțime) și `label` (etichetă), așa cum este ilustrat în Listarea 17-7:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-07/src/lib.rs:here}}
```

<span class="caption">Listarea 17-7: Structura `Button` care implementează trăsătura `Draw`</span>

Câmpurile `width`, `height` și `label` ale lui `Button` vor diferi față de cele ale altor componente. De exemplu, tipul `TextField` ar putea include aceste câmpuri plus unul suplimentar `placeholder`. Fiecare dintre tipurile pe care intenționăm să le afișăm pe ecran va implementa trăsătura `Draw` folosind cod diferit în metoda `draw` pentru a defini modul specific de desenare, așa cum demonstrează `Button` aici (lipsind, desigur, codul GUI real, așa cum am menționat). De exemplu, pentru `Button` ar putea exista un bloc `impl` separat, conținând metode specifice acțiunilor declanșate de click-ul utilizatorului pe buton, metode ce nu ar fi relevante pentru `TextField`.

Dacă un utilizator al bibliotecii noastre optează să implementeze o structură `SelectBox` cu câmpurile `width`, `height` și `options`, va trebui să implementeze și acesta trăsătura `Draw` pentru tipul `SelectBox`, conform Listării 17-8:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-08/src/main.rs:here}}
```

<span class="caption">Listarea 17-8: Un alt crate care utilizează `gui` și implementează trăsătura `Draw` pe structura `SelectBox`</span>

Utilizatorii bibliotecii noastre pot acum să-și construiască funcția `main` pentru a genera o instanță `Screen`. Acesteia i se pot adăuga un `SelectBox` și un `Button`, situați fiecare într-un `Box<T>` pentru a-i transforma în obiecte-trăsătură. În continuare, pot invoca metoda `run` pe instanța de `Screen`, metoda care va apela funcția `draw` pentru fiecare dintre componente. Listarea 17-9 prezintă această implementare:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-09/src/main.rs:here}}
```

<span class="caption">Listarea 17-9: Folosirea obiectelor-trăsătură pentru a stoca valori de diferite tipuri ce implementează aceeași trăsătură</span>

Când am dezvoltat biblioteca, nu cunoșteam nimic despre adăugarea tipului `SelectBox`, dar implementarea noastră pentru `Screen` a reușit să opereze asupra noului tip și să-l redea, pentru că `SelectBox` implementează trăsătura `Draw`, și în consecință metoda `draw`.

Conceptul acesta — de a fi preocupat exclusiv de mesajele la care o valoare răspunde, nu de tipul său concret — este asemănător cu cel de *duck typing* în limbajele cu tip dinamic: dacă se mișcă ca o rață și măcăie ca o rață, atunci chiar este o rață! În implementarea metodei `run` de la `Screen`, prezentată în Listarea 17-5, `run` nu are nevoie să cunoască tipul concret al fiecărui component. Acesta nu verifică dacă componentul este un `Button` sau un `SelectBox`, ci pur și simplu invocă metoda `draw` asupra lui. Definind `Box<dyn Draw>` ca tip pentru valorile din vectorul `components`, am specificat că `Screen` necesită valori asupra cărora putem apela metoda `draw`.

Beneficiul utilizării obiectelor-trăsătură și al sistemului de tipizare din Rust, pentru a scrie cod într-o manieră similară cu duck typing, este că nu este nevoie să verificăm la execuție dacă o valoare implementează o metodă particulară, nici să ne îngrijorăm de posibile erori în cazul în care o valoare nu implementează metoda și totuși se face apel la ea. Rust nu va compila codul nostru dacă valorile nu respectă trăsăturile cerute de obiectele-trăsătură.

De exemplu, Listarea 17-10 ne ilustrează ce se întâmplă dacă încercăm să construim un `Screen` cu un `String` ca component:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-10/src/main.rs}}
```

<span class="caption">Listarea 17-10: Tentativa de a folosi un tip ce nu implementează trăsătura cerută de obiectul-trăsătură</span>

Următoarea eroare apare pentru că `String` nu implementează trăsătura `Draw`:

```console
{{#include ../listings/ch17-oop/listing-17-10/output.txt}}
```

Această eroare ne indică faptul că fie am introdus în `Screen` un element neintenționat, ceea ce înseamnă că ar trebui să folosim un alt tip, fie că trebuie să implementăm trăsătura `Draw` pe `String`, astfel încât `Screen` să poată apela metoda `draw` asupra lui.

### Obiectele-trăsătură folosesc apelare dinamică

Să ne amintim în secțiunea [„Performanța Codului Utilizând Generici”][performance-of-code-using-generics]<!-- ignore --> din Capitolul 10 despre discuția privind procesul de monomorfizare efectuat de compilator când utilizăm limite de trăsătură pe generici: compilatorul creează implementări non-generice ale funcțiilor și metodelor pentru fiecare tip concret folosit în locul unui parametru generic de tip. Codul rezultat din monomorfizare realizează *invocare statică*, adică atunci când compilatorul cunoaște ce metodă apelezi în timpul compilării. Aceasta se contrastează cu *invocare dinamică*, atunci când compilatorul nu este capabil să determine în timpul compilării care metodă va fi apelată. În cazurile de apelare dinamică, compilatorul generează cod care va decide în timpul execuției care metodă să invoce.

Utilizând obiectele-trăsătură, Rust este constrâns la utilizarea invocării dinamice. Compilatorul nu cunoaște toate tipurile care pot fi folosite cu codul care utilizează obiectele-trăsătură, astfel nu poate determina ce metodă implementată pentru care tip ar trebui apelată. În schimb, în timpul execuției, Rust se folosește de pointerii din interiorul obiectului-trăsătură pentru a identifica metoda de apelat. Această căutare implică un cost la execuție, care nu există în cazul invocării statice. Mai mult, invocarea dinamică împiedică compilatorul să facă inline la codul unei metode, ceea ce la rândul său inhibă anumite optimizări. Cu toate acestea, am câștigat o flexibilitate suplimentară în codul scris în Listarea 17-5 și am fost în stare să o susținem în Listarea 17-9, deci este un compromis pe care trebuie să-l luăm în considerare.

[performance-of-code-using-generics]:
ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch19-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait
