## Trăsături avansate

Am abordat inițial trăsăturile în secțiunea [„Trăsături: Definirea comportamentului partajat”][traits-defining-shared-behavior]<!-- ignore --> din Capitolul 10, fără să intrăm în complexitățile mai avansate ale acestora. Având acum o înțelegere mai profundă despre Rust, putem explora aceste detalii mai sofisticate.

### Specificarea tipurilor placeholder în definițiile de trăsături prin utilizarea tipurilor asociate

*Tipurile asociate* sunt folosite pentru a lega un placeholder de tip de o anume trăsătură, permițând astfel definițiilor de metode ale trăsăturii să utilizeze aceste tipuri placeholder în semnăturile lor. Cine implementează o trăsătură va specifica tipul concret care urmează să fie utilizat în locul tipului placeholder pentru implementarea specifică. În acest fel, putem defini o trăsătură care folosește anumite tipuri fără a fi nevoie să cunoaștem exact care sunt aceste tipuri până când trăsătura este implementată.

Deși majoritatea caracteristicilor avansate prezentate în acest capitol sunt necesare doar ocazional, tipurile asociate sunt cam de mojloc: se folosesc mai rar decât funcționalitățile explicate în restul cărții, dar totuși mai frecvent decât alte caracteristici discutate aici.

Un exemplu de trăsătură ce folosește un tip asociat este trăsătura `Iterator` oferită de biblioteca standard a limbajului Rust. Acest tip asociat este denumit `Item` și înlocuiește tipul valorilor pe care structura ce implementează trăsătura `Iterator` intenționează să le parcurgă. Definiția trăsăturii `Iterator` este ilustrată în Listarea 19-12.

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-12/src/lib.rs}}
```

<span class="caption">Listarea 19-12: Definiția trăsăturii `Iterator` cu un tip asociat `Item`</span>

Tipul `Item` funcționează ca un înlocuitor, iar metoda `next` este definită astfel încât să returneze valori de tip `Option<Self::Item>`. Implementatorii trăsăturii `Iterator` vor alege un tip specific pentru `Item`, iar `next` va returna un `Option` care contine o valoare de acest tip specific.

Deși tipurile asociate pot pare să fie similare cu genericii, care ne permit să definim o funcție fără a indica tipurile cu care poate lucra, există diferențe importante. Analizăm diferențele prin exemplificarea unei implementări a trăsăturii `Iterator` pe tipul `Counter`, care indică faptul că tipul `Item` este `u32`:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

Această formă de exprimare pare similară cu cea utilizată pentru generici. Astfel, de ce nu definim pur și simplu trăsătura `Iterator` utilizând generici, cum este ilustrat în Listarea 19-13?

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-13/src/lib.rs}}
```

<span class="caption">Listarea 19-13: O variantă ipotetică a trăsăturii `Iterator` folosind generici</span>

Folosirea genericiilor, ca în Listarea 19-13, ne obligă să adnotăm tipurile în fiecare implementare, pentru că putem, de exemplu, să implementăm `Iterator<String>` pentru `Counter` sau pentru alte tipuri. Acest lucru înseamnă că `Iterator` poate avea mai multe implementări pentru `Counter`, cu tipuri concrete diferite ale parametrilor generici la fiecare implementare. Atunci când folosim metoda `next` pentru `Counter`, este necesar să furnizăm adnotări de tip pentru a specifica ce implementare a `Iterator` dorim să utilizăm.

Când lucrăm cu tipuri asociate, nu este necesar să adnotăm tipuri, deoarece o trăsătură nu poate fi implementată de mai multe ori pentru un același tip. În listarea 19-12, unde se folosește definiția cu tipuri asociate, putem alege o singură dată care va fi tipul pentru `Item`, deoarece există doar una singură `impl Iterator for Counter`. Nu trebuie să indicăm că dorim un iterator ce produce valori `u32` de câte ori apelăm `next` pentru `Counter`.

Tipurile asociate reprezintă de asemenea o componentă centrală a contractului trăsăturii: implementatorii trăsăturii trebuie să furnizeze un tip care să servească drept înlocuitor pentru tipul asociat. Numele tipurilor asociate sunt de obicei alese pentru a reflecta modul în care vor fi folosite și este o practică bună să includem documentarea acestora în documentația API.

### Parametrii generici de tip implicit şi supraîncărcarea operatorilor

Atunci când folosim parametri generici de tip, putem specifica un tip concret implicit pentru tipul generic. Aceasta elimină necesitatea ca cei care implementează trăsătura să aleagă un tip concret dacă tipul implicit este corespunzător. Un tip implicit este specificat în momentul declarării unui tip generic cu sintaxa `<PlaceholderType=ConcreteType>`.

Un exemplu foarte bun unde această tehnică este benefică este cu *supraîncărcarea operatorilor*, unde poți personaliza comportamentul unui operator (precum `+`) în scenarii specifice.

Rust nu permite crearea unor operatori noi sau supraîncărcarea unor operatori arbitrari. Însă, poți supraîncărca operațiile și trăsăturile aferente listate în `std::ops` implementând trăsăturile asociate acelui operator. De exemplu, în Listarea 19-14 supraîncărcăm operatorul `+` pentru a aduna două instanțe ale clasei `Point`. Aceasta se realizează prin implementarea trăsăturii `Add` pentru struct-ul `Point`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-14/src/main.rs}}
```

<span class="caption">Listarea 19-14: Implementarea trăsăturii `Add` pentru a realiza supraîncărcarea operatorului `+` pentru instanțele clasei `Point`</span>

Metoda `add` adună valorile `x` și `y` din două instanțe de `Point` pentru a crea o nouă instanță `Point`. Trăsătura `Add` include un tip asociat denumit `Output` care determină tipul returnat de metoda `add`.

Tipul generic implicit din acest cod este definit în interiorul trăsăturii `Add`. Iată definiția acesteia:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Acest cod ar trebui să ne fie relativ familiar: o trăsătură cu o metodă unică și un tip asociat. Noutatea este `Rhs=Self`: această sintaxă este cunoscută sub numele de *parametri de tip implicit*. Parametrul de tip generic `Rhs` (abreviere pentru "right hand side", sau "partea dreaptă") definește tipul parametrului `rhs` în metoda `add`. Dacă nu specificăm un tip concret pentru `Rhs` când implementăm trăsătura `Add`, tipul lui `Rhs` va fi implicit `Self`, care este tipul la care aplicăm implementarea trăsăturii `Add`.

Când am realizat implementarea lui `Add` pentru `Point`, am optat pentru tipul implicit `Rhs` pentru că vroiam să adunăm două instanțe `Point`. Să examinăm un caz în care implementăm trăsătura `Add` personalizând tipul `Rhs` în loc să utilizăm tipul implicit.

Avem două structuri, `Millimeters` și `Meters`, care reprezintă valori în unități de măsură diferite. Această metodă de încapsulare a unui tip existent într-o altă structură este cunoscută drept *newtype pattern*, concept pe care îl explicăm mai amănunțit în secțiunea [“Utilizarea pattern-ului newtype pentru implementarea trăsăturilor externe pe tipuri externe”][newtype]<!-- ignore --> . Intenționăm să adunăm valori măsurate în milimetri cu cele în metri, iar implementarea `Add` trebuie să efectueze conversia corect. Avem posibilitatea de a implementa `Add` pentru `Millimeters` cu `Meters` ca tip `Rhs`, după cum este prezentat în Listarea 19-15.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-15/src/lib.rs}}
```

<span class="caption">Listarea 19-15: Implementarea trăsăturii `Add` pentru `Millimeters` astfel încât să putem adăuga `Millimeters` la `Meters`</span>

Pentru a aduna `Millimeters` cu `Meters`, specificăm `impl Add<Meters>` pentru a defini valoarea pentru parametrul de tip `Rhs`, în loc să folosim valoarea implicită `Self`.

Vei utiliza parametri de tip implicit în două situații:
* Pentru a extinde un tip fără a afecta codul existent
* Pentru a permite personalizări în anumite cazuri pe care majoritatea utilizatorilor nu le vor necesita

Trăsătura `Add` din biblioteca standard ilustrează acest al doilea scop: de cele mai multe ori, dorim adunarea a două tipuri identice, însă trăsătura `Add` oferă flexibilitatea de a merge dincolo de acest caz standard. Utilizarea unui parametru de tip implicit în definiția trăsăturii `Add` înseamnă că, de obicei, nu este necesar să specificăm acest parametru suplimentar. Prin urmare, nu este nevoie de scrierea unor porțiuni de cod standard pentru implementare, facilitând folosirea trăsăturii.

Primul scop este asemănător cu cel de-al doilea, dar aplicat în sens invers: dacă dorim să adăugăm un parametru de tip unei trăsături existente, acordându-i o valoare implicită va permite extinderea funcționalității acelei trăsături fără a compromite codul implementat anterior.

### Dezambiguizarea metodelor cu același nume prin sintaxa complet calificată

În Rust, nu există nicio restricție care să prevină o trăsătură să aibă o metodă cu același nume ca și o metoda din altă trăsătură, nici nu se interzice implementarea ambelor trăsături pe un singur tip. Este posibil, de asemenea, să implementăm direct pe tip o metodă cu același nume ca metodele din alte trăsături.

Când dorim să apelăm metode ce poartă același nume, trebuie să informăm Rust despre care dintre acestea intenționăm să o folosim. Considerăm codul prezentat în Listarea 19-16, unde am definit două trăsături, `Pilot` și `Wizard`, fiecare având o metodă denumită `fly`. Implementăm ambele trăsături pe tipul `Human`, care are deja implementată propria sa metodă `fly`. Fiecare metoda `fly` realizează o acțiune diferită.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-16/src/main.rs:here}}
```

<span class="caption">Listarea 19-16: Definirea a două trăsături cu o metodă `fly` și implementarea lor pe tipul `Human`, plus o metodă `fly` implementată direct pe `Human`</span>

Atunci când apelăm metoda `fly` pe o instanță de `Human`, compilatorul alege în mod standard metoda implementată direct pe tip, după cum arată Listarea 19-17.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-17/src/main.rs:here}}
```

<span class="caption">Listarea 19-17: Apelarea metodei `fly` pe o instanță de `Human`</span>

Execuția acestui cod va produce afișajul `*waving arms furiously*`, ceea ce indică faptul că Rust a apelat metoda `fly` implementată direct pe `Human`.

Dacă vrem să apelăm metodele `fly` din trăsăturile `Pilot` sau `Wizard`, trebuie să utilizăm o sintaxă diferită pentru a specifica exact metoda `fly` dorită. Listarea 19-18 ilustrează utilizarea acestei sintaxe.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-18/src/main.rs:here}}
```

<span class="caption">Listarea 19-18: Specificarea trăsăturii din care dorim să apelăm metoda `fly`</span>

Indicând numele trăsăturii înaintea numelui metodei, facem clar pentru Rust care implementare a `fly` dorim să o apelăm. Am putea de asemenea folosi `Human::fly(&person)`, care are aceeași semnificație cu `person.fly()` utilizat în Listarea 19-18, dar aceasta sintaxă este puțin mai lungă și nu este necesară dacă nu e nevoie de dezambiguizare.

Executarea acestui cod generează următorul afișaj:

```console
{{#include ../listings/ch19-advanced-features/listing-19-18/output.txt}}
```

Deoarece metoda `fly` folosește un parametru `self`, dacă am avea două *tipuri* care implementează o *trăsătură*, Rust ar putea determina ce implementare a unei trăsături să aleagă pe baza tipului parametrului `self`.

Însă, funcțiile asociate care nu sunt metode nu au un parametru `self`. Când avem mai multe tipuri sau trăsături care definesc funcții asociate non-metode cu același nume, Rust nu poate determina întotdeauna tipul pe care îl vizăm fără utilizarea *sintaxei complet calificate*. De exemplu, în Listarea 19-19 cream o trăsătură pentru un adăpost de animale care vrea să numească toți cățelușii *Spot*. Introducem trăsătura `Animal` cu o funcție asociată non-metodă `baby_name`. Structura `Dog`, care de asemenea implementează trăsătura `Animal`, definește propria funcție asociată non-metodă `baby_name`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-19/src/main.rs}}
```

<span class="caption">Listarea 19-19: O trăsătură cu o funcție asociată și un tip cu o funcție asociată cu același nume care mai și implementează trăsătura</span>

Implementăm codul pentru numirea tuturor cățelușilor în funcția `baby_name` asociată structurii `Dog`. Tipul `Dog` implementează, de asemenea, trăsătura `Animal`, care descrie caracteristici comune tuturor animalelor. Termenul pentru cățel este `puppy`, și acest lucru este exprimat în implementarea trăsăturii `Animal` pentru `Dog` în funcția `baby_name` ce aparține trăsăturii `Animal`.

În funcția `main`, invocăm funcția `Dog::baby_name`, care apelează funcția asociată definită direct în cadrul `Dog`. Acest cod produce următorul afișaj:

```console
{{#include ../listings/ch19-advanced-features/listing-19-19/output.txt}}
```

Rezultatul nu corespunde așteptărilor noastre. Dorim să apelăm funcția `baby_name` care face parte din trăsătura `Animal` pe care am implementat-o pentru `Dog`, pentru ca astfel codul să afișeze `A baby dog is called a puppy`. Metoda de specificare a numelui trăsăturii utilizată în Listarea 19-18 nu ajută în acest caz; dacă modificăm `main` conform codului din Listarea 19-20, vom întâmpina o eroare de compilare.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-20/src/main.rs:here}}
```

<span class="caption">Listarea 19-20: Încercarea de a invoca funcția `baby_name` din trăsătura `Animal`, însă Rust nu poate determina care implementare să o aleagă</span>

Pentru că `Animal::baby_name` nu primește un parametru `self` și ar fi posibil să existe alte tipuri care implementează trăsătura `Animal`, Rust nu poate decide care implementare a `Animal::baby_name` dorim să o utilizăm. Vom primi următoarea eroare de compilare:

```console
{{#include ../listings/ch19-advanced-features/listing-19-20/output.txt}}
```

Pentru a preciza și a informa Rust că vrem să folosim implementarea `Animal` specifică pentru `Dog` în detrimentul implementării pentru alt tip, trebuie să ne folosim de sintaxa complet calificată. Listarea 19-21 arată cum să utilizăm sintaxa complet calificată.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-21/src/main.rs:here}}
```

<span class="caption">Listarea 19-21: Utilizarea sintaxei complet calificate pentru a indica intenția de a apela funcția `baby_name` din trăsătura `Animal` așa cum este implementată pentru `Dog`</span>

Îi oferim lui Rust o adnotație de tip între parantezele unghiulare, semnalând că intenționăm să apelăm metoda `baby_name` din trăsătura `Animal`, așa cum este implementată pentru `Dog`. Specificăm că dorim să considerăm tipul `Dog` ca fiind un `Animal` când efectuăm acest apel de funcție. Urmată această metodă, codul nostru va afișa ceea ce intenționăm:

```console
{{#include ../listings/ch19-advanced-features/listing-19-21/output.txt}}
```

În general, sintaxa complet calificată se definește astfel:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Pentru funcțiile asociate care nu sunt metode, nu vom avea un `receiver`, ci doar lista celorlalte argumente. Poți folosi sintaxa complet calificată oriunde apelezi funcții sau metode. Totuși, poți să omiți orice parte a acestei sintaxe pe care Rust o poate infera din restul informațiilor din program. Această sintaxă mai detaliată este necesară numai în situațiile în care există mai multe implementări care utilizează același nume și Rust are nevoie de ajutor pentru a identifica care implementare vrei să o apelezi.

### Utilizarea super-trăsăturilor pentru a solicita funcționalitatea unei trăsături în cadrul alte trăsături

Există cazuri când vom scrie o definiție pentru o trăsătură ce depinde de o altă trăsătură: pentru ca un tip să implementeze prima trăsătură, dorim ca acel tip să implementeze și a doua trăsătură. Acest lucru este necesar pentru ca definiția trăsăturii noastre să poată utiliza elementele asociate ale celei de-a doua trăsături. Trăsătura de care depinde trăsătura noastră se numește o *super-trăsătură*.

Să considerăm, de exemplu, că dorim să creăm o trăsătură `OutlinePrint` care include metoda `outline_print`. Aceasta va tipări o valoare astfel încât să fie încadrată în asteriscuri. De pildă, pentru un struct `Point` ce implementează trăsătura `Display` din biblioteca standard și care returnează `(x, y)`, dacă invocăm `outline_print` pe o instanță `Point` cu `x` egal cu `1` și `y` egal cu `3`, ar trebui să obținem:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

Când implementăm metoda `outline_print`, dorim să facem uz de funcționalitatea oferită de trăsătura `Display`. De aceea, trebuie să specificăm că trăsătura `OutlinePrint` va funcționa numai cu tipurile ce implementează și `Display`, și care furnizează funcționalitatea necesară pentru `OutlinePrint`. Putem indica acest lucru în definiția trăsăturii, folosind notația `OutlinePrint: Display`. Această metodă este asemănătoare cu adăugarea unei delimitări de trăsătură. Listarea 19-22 ne demonstrează implementarea trăsăturii `OutlinePrint`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-22/src/main.rs:here}}
```

<span class="caption">Listarea 19-22: Implementarea trăsăturii `OutlinePrint` care necesită funcționalitatea trăsăturii `Display`</span>

Având în vedere că am stipulat ca `OutlinePrint` să necesite trăsătura `Display`, putem folosi funcția `to_string`, care se implementează automat pentru orice tip ce respectă trăsătura `Display`. Dacă am încerca să utilizăm `to_string` fără a adăuga două puncte `:` și fără a menționa trăsătura `Display` după numele trăsăturii, am întâmpina o eroare specificând că nu se găsește nicio metodă denumită `to_string` pentru tipul `&Self` în contextul actual.

Să examinăm ce se întâmplă atunci când încercăm să aplicăm trăsătura `OutlinePrint` unui tip care nu implementează `Display`, cum ar fi structura `Point`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

Vom primi o eroare care indică necesitatea implementării trăsăturii `Display`, neîndeplinită în cazul de față:

```console
{{#include ../listings/ch19-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Pentru a rezolva problema, vom implementa `Display` pe `Point` și astfel vom îndeplini cerința impusă de `OutlinePrint`, în felul următor:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

Astfel, implementarea trăsăturii `OutlinePrint` pe `Point` va compila cu succes, permițându-ne să apelăm `outline_print` pe o instanță de `Point` pentru a o afișa într-un contur de asteriscuri.

### Aplicarea pattern-ului newtype pentru implementarea trăsăturilor externe pe tipuri externe

În capitolul 10, secțiunea [„Implementarea unei trăsături pe un tip”][implementing-a-trait-on-a-type]<!-- ignore --> ne referim la regula orfanilor conform căreia putem implementa o trăsătură pe un tip doar dacă trăsătura sau tipul sunt locale crate-ului nostru. Poate fi ocolită această restricție utilizând *pattern-ul newtype*, ce presupune crearea unui nou tip în cadrul unui tuplă struct. (Această temă a fost abordată în secțiunea [„Utilizarea structurilor tuplă fără câmpuri denumite pentru a genera tipuri diferite”][tuple-structs]<!-- ignore --> din capitolul 5.) Tuple struct-ul în cauză va avea un singur câmp și va fi un înveliș subțire peste tipul căruia dorim să-i aplicăm o trăsătură. Astfel, tipul înveliș devine local crate-ului nostru, permițându-ne să implementăm trăsătura pe acesta.

Termenul *Newtype* provine din limbajul de programare Haskell. Nu există penalități de performanță la rulare când se utilizează acest pattern, tipul înveliș fiind eliminat în timpul compilării.

Spre exemplu, dacă dorim să implementăm `Display` pentru `Vec<T>`, regula orfanilor ne împiedică să o facem direct deoarece trăsătura `Display` și tipul `Vec<T>` sunt definite în afara crate-ului nostru. Putem defini un struct `Wrapper` ce encapsulează o instanță `Vec<T>`; apoi putem implementa `Display` pe `Wrapper` și să utilizăm valoarea `Vec<T>`, conform Listării 19-23.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-23/src/main.rs}}
```

<span class="caption">Listarea 19-23: Crearea unui tip `Wrapper` peste `Vec<String>` pentru a implementa `Display`</span>

Implementarea `Display` accesează `Vec<T>` intern utilizând `self.0`, dat fiind că `Wrapper` este un tuple struct și `Vec<T>` este elementul de la indexul 0 în tuplă. În continuare, putem folosi funcționalitatea `Display` pentru `Wrapper`.

Principalul dezavantaj al acestei tehnici este că `Wrapper` reprezintă un tip nou și nu deține metodele valorii pe care o conține. Ar trebui să implementăm toate metodele lui `Vec<T>` pe `Wrapper`, astfel încât acestea să fie delegate către `self.0`, permițându-ne să lucrăm cu `Wrapper` ca și cum ar fi un `Vec<T>`. Pentru ca noul tip să aibă toate metodele tipului intern, o soluție ar fi implementarea trăsăturii `Deref` (discutată în capitolul 15 în secțiunea [„Utilizarea pointerilor inteligenți la fel ca referințele obișnuite cu trăsătura `Deref`”][smart-pointer-deref]<!-- ignore -->) pe `Wrapper` pentru a returna tipul intern. Dacă nu dorim ca tipul `Wrapper` să dispună de toate metodele tipului intern - de exemplu, pentru a-i restricționa comportamentul - atunci trebuie să implementăm manual doar acele metode pe care le dorim.

Pattern-ul newtype este folositor chiar și în situații care nu implică trăsături. Să ne schimbăm acum perspectiva și să explorăm modalități avansate de a interacționa cu sistemul de tipuri Rust.

[newtype]: ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types [implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type 
[traits-defining-shared-behavior]: ch10-02-traits.html#traits-defining-shared-behavior 
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait 
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
