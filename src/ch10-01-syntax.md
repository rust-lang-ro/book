## Tipuri de date generice

Genericile sunt instrumente pe care le utilizăm pentru a construi definiții de elemente, cum ar fi semnăturile de funcții sau structurile, ce pot fi apoi folosite cu o multitudine de tipuri de date concrete. Înainte de toate, să ne uităm cum putem defini funcții, structuri, enumerări și metode folosind genericile. Ulterior, vom aborda impactul pe care îl au genericile asupra performanței codului.

### Definirea funcțiilor cu generici

Atunci când definim o funcție ce folosește generici, introducem genericii în semnătura funcției acolo unde, în mod obișnuit, am specifica tipurile de date pentru parametri și valoarea returnată. Acest demers face codul nostru mai maleabil și oferă mai multă funcționalitate celor ce apelează funcția, prevenind duplicarea de cod.

Continuând cu funcția noastră `largest`, Listarea 10-4 prezintă două funcții care identifică valoarea cea mai mare dintr-o secțiune. Apoi, vom unifica acestea într-o singură funcție care încorporează generici.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

<span class="caption">Listarea 10-4: Două funcții care se diferențiază prin numele și tipurile specificate în semnăturile lor</span>

Funcția `largest_i32`, pe care am extras-o în Listarea 10-3, localizează cel mai mare `i32` dintr-o secțiune. Funcția `largest_char` identifică cel mai mare `char` dintr-o secțiune. Cum ambele funcții au corpuri identice, vom elimina duplicarea introducând un parametru de tip generic într-o funcție singulară.

Pentru a parametriza tipurile în noua funcție singulară, trebuie să numim parametrul de tip, la fel cum nominalizăm parametrii valorici ai unei funcții. Orice identificator poate fi folosit ca nume de parametru de tip. Totuși, ne vom folosi de `T` conform convenției uzuale în Rust, unde numele parametrilor de tip sunt scurte, frecvent doar o literă, și urmează stilul de denumire UpperCamelCase. `T`, fiind prescurtarea pentru „type”, este alegerea preferată de majoritatea dezvoltatorilor de Rust.

Când utilizăm un parametru în corpul funcției, trebuie să îl declarăm în semnătură pentru ca compilatorul să înțeleagă la ce ne referim. În mod similar, atunci când utilizăm un nume pentru un parametru de tip în semnătura unei funcții, trebuie să declarăm acest nume de tip înainte de a-l folosi. Pentru a defini funcția generică `largest`, vom insera numele de tip în interiorul parantezelor unghiulare, `<>`, așezate între numele funcției și lista de parametri, în felul următor:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Descifrăm această definiție în felul următor: funcția `largest` funcționează generic pentru un anumit tip `T`. Funcția dispune de un parametru denumit `list`, care este o secțiune de elemente de tip `T`. Funcția `largest` va returna o referință către o valoare de același tip `T`.

Listarea 10-5 ne oferă definiția combinată a funcției `largest`, care incorporează tipul de date generic în semnătura sa. Listarea ilustrează și cum putem invoca funcția folosind fie o secțiune de valori `i32`, fie una de `char`. Observăm că acest cod nu va compila încă, dar vom adresa această problemă mai târziu în capitul curent.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

<span class="caption">Listarea 10-5: Funcția `largest` utilizând parametrii de tip generic; în stadiul actual codul nu se compilează</span>

Dacă încercăm să compilăm acest cod acum, ne vom confrunta cu următoarea eroare:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

Mesajul de ajutor ne îndreaptă atenția spre `std::cmp::PartialOrd`, care este o *trăsătură*, subiect ce va fi abordat în secțiunea următoare. Pentru moment, este important să înțelegem că această eroare ne transmite că implementarea funcției `largest` nu va opera corect pentru toate tipurile posibile ale lui `T`. Pentru că dorim să comparăm valorile de tip `T` în corpul funcției, ne limităm la acele tipuri ale căror valori pot fi comparate în ordine. Biblioteca standard facilitează acest lucru prin intermediul trăsăturii `std::cmp::PartialOrd`, pe care o puteți implementa pentru diverse tipuri (pentru mai multe detalii referitoare la această trăsătură, consultați Anexa C). Urmând sugestia din mesajul de ajutor, vom restricționa tipurile valide pentru `T` la cele care implementează `PartialOrd`, și astfel exemplul nostru va compila fără probleme, dat fiind că biblioteca standard furnizează implementări pentru `PartialOrd` atât pentru tipul `i32`, cât și pentru `char`.

### În definiția structurilor

Noi putem defini structuri care să încorporeze unul sau mai multe câmpuri care folosesc parametri de tip generic, prin intermediul sintaxei cu paranteze unghiulare `<>`. Listarea 10-6 înfățișează definiția structurii `Point<T>`, care reține valori ale coordonatelor `x` și `y` de orice tip.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

<span class="caption">Listarea 10-6: O structură `Point<T>` ce conține valori `x` și `y` de tip `T`</span>

Utilizarea genericilor în definirea structurilor urmează o sintaxă similară cu aceea din definițiile funcțiilor. Inițial, numele parametrului de tip generic e declarat între paranteze unghiulare după denumirea structurii. În continuare, folosim tipul generic în cadrul definiției structurii în locul unde, de obicei, sunt specificate tipuri de date fixe.

Este esențial să avem în vedere că, prin utilizarea unui unic tip generic în definirea `Point<T>`, noi comunicăm că structura `Point<T>` este generică peste un anumit tip `T`, și că câmpurile `x` și `y` sunt *în ambele situații* de același tip, oricare ar fi acesta. Astfel, dacă creăm o instanță `Point<T>` cu valori ale tipurilor diferite, așa cum arată Listarea 10-7, codul nu va fi compilabil.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

<span class="caption">Listarea 10-7: Câmpurile `x` și `y` trebuie să fie de același tip din cauză că ambele folosesc tipul de date generic `T`.</span>

În exemplul de față, o dată ce atribuim `x`-ului valoarea întreagă 5, îi semnalăm compilatorului că pentru această instanță de `Point<T>`, tipul generic `T` va fi un întreg. Apoi, dacă pentru `y` specificăm valoarea 4.0, care ar trebui să fie de același tip ca `x`, vom întâmpina o eroare de incompatibilitate a tipurilor, după cum urmează:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

Dacă dorim să definim o structură `Point` în care `x` și `y` să fie generice și să accepte tipuri diferite, ne putem folosi de parametri de tip generic multipli. Ca exemplu, în Listarea 10-8, am modificat definiția lui `Point` pentru a deveni generică peste tipurile de date `T` și `U`, unde `x` este de tip `T` și `y` de tip `U`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

<span class="caption">Listarea 10-8: Structura `Point<T, U>`, generică peste două tipuri, permițând ca `x` și `y` să fie de tipuri diferite</span>

Acum fiecare dintre exemplele prezentate pentru `Point` sunt posibile! Este permisă utilizarea unei varietăți de parametri de tip generic în definiția unei structuri, însă un exces în acest sens poate mări complexitatea codului și îl face greu de urmărit. Dacă-ți dai seama că ai nevoie de multe tipuri generice în codul tău, probabil ar fi benefică o restructurare pentru simplificarea codului.

### În definiția enumerărilor

Ca și în cazul structurilor, putem defini enumerări ce includ tipuri de date generice în variantele lor. Să revizuim enumerarea `Option<T>`, pusă la dispoziție de biblioteca standard, pe care am utilizat-o anterior în Capitolul 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Această definiție ar trebui acum să fie mai inteligibilă pentru tine. După cum poți vedea, enumerarea `Option<T>` este generică peste tipul `T` și are două variante: `Some`, care include o valoare de tipul `T`, și `None`, care nu include nicio valoare. Utilizând enumerarea `Option<T>`, putem exprima conceptul abstract al unei valori facultative și, fiindcă `Option<T>` este generic, această noțiune poate fi aplicată indiferent de tipul valorii facultative.

De asemenea, enumerările pot folosi mai multe tipuri generice. Definiția enumerării `Result`, pe care am folosit-o în Capitolul 9, este un astfel de exemplu:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Enumerarea `Result` este generică peste două tipuri, `T` și `E`, și încorporează două variante: `Ok`, ce include o valoare de tip `T`, și `Err`, ce include o valoare de tip `E`. Această definiție facilitează folosirea enumerării `Result` în orice context avem o operațiune ce ar putea avea succes (întorcând o valoare de un anumit tip `T`) sau ar putea eșua (întorcând o eroare de un anumit tip `E`). Aceasta este metoda pe care am aplicat-o atunci când am deschis un fișier în Listarea 9-3, unde `T` a fost înlocuit cu tipul `std::fs::File` pentru un caz de succes și `E` a fost înlocuit cu `std::io::Error` pentru cazurile de eroare în deschiderea fișierului.

Atunci când întâmpini în codul tău situații în care multiple structuri sau enumerări se diferențiază doar prin tipul valorilor pe care le conțin, poți evita repetiția prin aplicarea tipurilor generice.

### În definiția metodelor

Noi putem implementa metode pe structuri și enumerări, așa cum am făcut în Capitolul 5, utilizând și tipuri generice în definițiile lor. În Listarea 10-9 este prezentată structura `Point<T>`, definită anterior în Listarea 10-6, cu o metodă numită `x` implementată pe ea.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

<span class="caption">Listarea 10-9: Implementarea unei metode denumite `x` pe structura `Point<T>`, care returnează o referință către câmpul `x` de tip `T`</span>

Aici, am definit o metodă `x` pe `Point<T>` care oferă o referință către datele din câmpul `x`.

Este necesar să declarăm `T` imediat după `impl` pentru a putea folosi `T` în specificarea că implementăm metode pe structura `Point<T>`. Declarând `T` ca tip generic după `impl`, Rust înțelege că tipul din parantezele unghiulare din `Point` este generic și nu concret. Desigur, am fi putut alege un nume diferit pentru acest parametru generic, comparativ cu cel din definiția structurii, dar convenția sugerează utilizarea aceluiași nume. Metodele definite într-un bloc `impl` care declară tipul generic vor fi aplicabile pe orice instanță de `Point<T>`, indiferent de substituția tipului generic cu un tip concret.

Putem impune, de asemenea, anumite restricții asupra tipurilor generice când definim metode pe un anumit tip. De exemplu, putem implementa metode exclusiv pe instanțe de `Point<f32>` și nu pe `Point<T>` cu orice tip generic. În Listarea 10-10, utilizăm tipul concret `f32`, fără a declarăm tipuri după `impl`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

<span class="caption">Listarea 10-10: Un bloc `impl` specific pentru o structură cu un tip concret dat pentru parametrul generic `T`</span>

Prin acest cod, `Point<f32>` va avea o metodă `distance_from_origin`, în timp ce alte instanțe de `Point<T>`, unde `T` nu este `f32`, nu vor avea definită această metodă. Metoda calculează cât de departe este un punct de originea de coordonate (0.0, 0.0), folosind operațiuni matematice specifice pentru tipurile cu virgulă mobilă.

Parametrii generici din definiția unei structuri nu trebuie să corespundă întotdeauna cu cei din semnăturile metodelor respectivei structuri. Listarea 10-11 folosește tipurile generice `X1` și `Y1` pentru structura `Point` și `X2`, `Y2` pentru semnătura metodei `mixup`, pentru a ilustra mai clar concepția. Metoda creează o nouă instanță `Point` cu valoarea `x` din instanța `self` de tip `X1` și valoarea `y` din instanța de `Point` primită ca argument, de tip `Y2`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

<span class="caption">Listarea 10-11: O metodă care utilizează tipuri generice diferite de cele ale definiției structurii sale</span>

În funcția `main`, am definit un `Point` cu un `i32` pentru `x` (valoare `5`) și un `f64` pentru `y` (valoare `10.4`). Variabila `p2` este un `Point` care conține o secțiune de string pentru `x` (cu valoarea `"Hello"`) și un `char` pentru `y` (cu valoarea `c`). Apelând metoda `mixup` pe `p1` cu argumentul `p2` generăm variabila `p3`, ce va prelua valoarea `x` de tip `i32` de la `p1` și valoarea `y` de tip `char` de la `p2`. Apelul macro-ului `println!` va afișa: `p3.x = 5, p3.y = c`.

Exemplul servește la demonstrarea unei situații în care unii parametri generici sunt definiți în blocul `impl` și alții sunt incluși în definiția metodei propriu-zise. În acest context, parametrii generici `X1` și `Y1` sunt declarați alături de `impl` deoarece sunt asociați cu definiția structurii, în vreme ce `X2` și `Y2` sunt introduși odată cu definiția funcției `mixup`, având relevanță doar în contextul acelei metode.

### Performanța codului folosind generici

Ai putea să te întrebi dacă folosirea parametrilor de tip generic implică un cost la rulare. Vestea excelentă este că utilizarea genericilor nu va încetini executarea programului tău comparativ cu folosirea tipurilor concrete.

Rust atinge acest performanță prin monomorfizarea codului cu generici în timpul compilării. *Monomorfizarea* este procesul prin care codul generic este transformat în cod specific, prin completarea cu tipurile concrete utilizate în momentul compilării. În acest proces, compilatorul face contrariul demersurilor noastre din crearea funcției generice prezentată în Listarea 10-5: acesta analizează toate locurile unde este invocat codul generic și generează cod pentru tipurile concrete utilizate.

Explorăm acest mecanism prin intermediul enum-ului generic `Option<T>` din biblioteca standardă a limbajului Rust:

```rust
let integer = Some(5);
let float = Some(5.0);
```

La compilarea acestui cod, Rust efectuează monomorfizarea. Compilatorul identifică valorile folosite în instanțele `Option<T>` și recunoaște două variante ale lui `Option<T>`: una pentru `i32` și alta pentru `f64`. În consecință, extinde definiția generică a `Option<T>` în două versiuni specializate pentru `i32` și `f64`, substituind astfel definiția generică.

Versiunea monomorfizată a codului ar putea arăta astfel (compilatorul folosește denumiri diferite de cele alese de noi aici pentru exemplificare):

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

Genericul `Option<T>` este substituit cu definițiile specifice generate de compilator. Deoarece Rust transformă codul generic în cod care precizează tipul pentru fiecare instanță, nu întâmpinăm niciun cost suplimentar la rulare atunci când folosim genericii. Astfel, când codul este executat, performanța este echivalentă cu cea pe care am obține-o dacă am duplica manual fiecare definiție. Procesul de monomorfizare face ca genericii din Rust să fie extrem de performanți la executare.
