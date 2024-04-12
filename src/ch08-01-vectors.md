## Păstrarea listelor de valori folosind vectori

Primul tip de colecție pe care îl vom discuta se numește `Vec<T>`, cunoscut mai bine ca *vector*. Vectorii ne permit să păstrăm mai multe valori într-o singură structură de date care plasează toate valorile una lângă alta în memorie. Un aspect important este că vectorii pot conține doar valori de același tip. Aceștia se dovedesc a fi foarte utili când ai o listă de elemente, precum liniile de text dintr-un fișier sau prețurile produselor dintr-un coș de cumpărături.

### Crearea unui vector nou

Pentru a crea un vector nou și gol, utilizăm funcția `Vec::new`, după cum urmează în Listarea 8-1.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

<span class="caption">Listarea 8-1: Crearea unui vector nou și gol pentru valori de tip `i32`</span>

Aici am adăugat o adnotare de tip deoarece nu introducem valori și Rust nu poate deduce tipul elementelor pe care dorim să le stocăm. Acest aspect este crucial. Vectorii sunt construiți folosind generice, iar utilizarea genericelor cu tipurile proprii le vom explora în Capitolul 10. Până atunci, este important să știi că tipul `Vec<T>` din biblioteca standard poate conține orice alt tip. Când inițializăm un vector pentru un anumit tip, specificăm acest tip între paranteze unghiulare. În Listarea 8-1, i-am indicat lui Rust că `Vec<T>` de la variabila `v` va conține elemente de tip `i32`.

De obicei, cel mai frecvent vei inițializa vectorii `Vec<T>` cu valori specifice și Rust va infera automat tipul de date pe care dorești să-l stochezi. Prin urmare, este rar necesar să oferi adnotări de tip. Rust oferă macro-ul `vec!`, care te ajută să creezi direct un vector nou cu valorile specificate. Listarea 8-2 arată cum să creezi un `Vec<i32>` care stochează valorile `1`, `2` și `3`. Tipul de date pentru întregi este `i32`, conform predefinirii pentru tipul întreg, așa cum am discutat în secțiunea [„Tipuri de date”][data-types]<!-- ignore --> din Capitolul 3.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

<span class="caption">Listarea 8-2: Crearea unui vector nou care include anumite valori</span>

Deoarece am furnizat valori inițiale de tip `i32`, Rust poate determina că `v` este de tipul `Vec<i32>`, așadar adnotarea de tip nu este necesară. Următorul pas este să învățăm cum putem modifica un vector.

### Actualizarea unui vector

Pentru a crea un vector și a-i adăuga elemente, putem utiliza metoda `push`. Urmărește exemplul din Listarea 8-3.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

<span class="caption">Listarea 8-3: Adăugarea valorilor într-un vector folosind metoda `push`</span>

Dacă vrem să modificăm o variabilă, trebuie să o declarăm ca fiind mutabilă, utilizând cuvântul cheie `mut`, după cum am explicat în Capitolul 3. Toate numerele inserate sunt de tip `i32`. Rust înțelege acest lucru automat din datele furnizate, astfel încât nu este necesară specificarea explicită a tipului `Vec<i32>`.

### Accesarea elementelor din vectori

Există două modalități prin care poți referenția o valoare stocată într-un vector: prin indexare sau utilizând metoda `get`. Pentru claritate, în exemplele următoare am specificat tipurile valorilor returnate de aceste două funcții.

În Listarea 8-4, sunt ilustrate ambele metode de accesare a unei valori dintr-un vector - prin indexare directă și folosind metoda `get`.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

<span class="caption">Listarea 8-4: Accesarea unui element dintr-un vector folosind sintaxa de indexare sau metoda `get`</span>

Aici trebuie să remarcăm anumite detalii. Folosim indicele `2` pentru a ajunge la al treilea element, având în vedere că vectorii sunt indexați începând cu zero. Operatorul `&` împreună cu `[]` ne oferă o referință către elementul situat la indicele specificat. Când apelăm metoda `get` cu un indice ca argument, primim o variantă `Option<&T>`, pe care putem să o utilizăm într-o instrucțiune `match`.

Rust ne oferă aceste două metode de referențiere astfel încât să putem alege cum dorești să răspundă programul atunci când accesăm un indice din afara limitelor vectorului. Să luăm un exemplu: ce se întâmplă dacă avem un vector cu cinci elemente și încercăm să accesăm un element la indicele 100 folosind fiecare metodă, așa cum vedem în Listarea 8-5.

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

<span class="caption">Listarea 8-5: Tentativa de a accesa elementul la indicele 100 dintr-un vector cu cinci elemente</span>

Rulând acest cod, prima metodă, cea cu `[]`, va provoca o eroare fără recuperare (panică), din cauza referinței la un element inexistent. Această methodă e utilă când vrem ca programul să se oprească în cazul unei tentative de a accesa un element dincolo de capătul vectorului.

Pe de altă parte, când metoda `get` primește un indice ce depășește limitele vectorului, aceasta returnează `None` fără a genera o panică. Ai alege această cale dacă posibilitatea unui acces la un indice în afara vectorului aparține situațiilor normale în aplicația ta. Astfel, codul va include logica de tratare a cazurilor atunci când rezultatul e `Some(&element)` sau `None`, după cum am analizat în Capitolul 6. De exemplu, un utilizator poate introduce accidental un număr prea mare, iar programul ar returna `None`. În acest caz, ai putea să-l informezi despre numărul de elemente din vector și să-i oferi o nouă șansă pentru a introduce un număr valabil, o soluție mult mai favorabilă decât închiderea programului.

Atunci când avem o referință valabilă, verificatorul de împrumut verifică regulile de proprietate și împrumut (abordate în Capitolul 4), pentru a se asigura că această referință, precum și orice alte referințe la conținutul vectorului, sunt valide. Amintește-ți de regula importantă care impune că nu poți deține referințe mutabile și imutabile simultan. Această regulă este ilustrată în Listarea 8-6, unde avem o referință imutabilă la primul element din vector, iar în același timp încercăm să adăugăm un element la final. Programul nu va funcționa dacă apoi încercăm să accesăm din nou acel element:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

<span class="caption">Listarea 8-6: Încercarea de adăugare a unui element la vector în timp ce există o referință către un element al său</span>

Compilarea acestui cod va genera următoarea eroare:


```console
{{#include ../listings/ch08-common-collections/listing-08-06/output.txt}}
```

Poate părea surprinzător că acest cod generează o eroare, deoarece te-ai putea întreba de ce o referință la primul element ar fi afectată de modificările produse la capătul vectorului. Motivul erorii este legat de cum vectorii își gestionează memoria: adăugând un nou element, poate fi necesară alocarea unui nou segment de memorie și copierea elementelor vechi în acesta, dacă spațiul actual nu este suficient. Într-o astfel de situație, referința la primul element ar indica spre memorie care a fost eliberată. Regulile de împrumut împiedică astfel de situații neplăcute.

> Notă: Pentru mai multe detalii despre implementarea tipului `Vec<T>`, poți consulta [“The Rustonomicon”][nomicon].

### Iterând prin elementele unui vector

Pentru a accesa elementele unui vector rând pe rând, cel mai eficient este să folosim o iterație completă, decât să accesăm elementele individual prin indici. Listarea 8-7 demonstrează modul în care putem utiliza un ciclu `for` pentru a parcurge un vector de valori de tip `i32`, obținând referințe imutabile la fiecare element și afișându-le.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

<span class="caption">Listarea 8-7: Afișarea fiecărui element al unui vector prin iterare cu ajutorul unui ciclu `for`</span>

Este posibil să iterăm și prin referințe mutabile ale elementelor unui vector mutabil, pentru a modifica toate elementele acestuia. Ciclul `for` din Listarea 8-8 adaugă `50` la valoarea fiecărui element.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

<span class="caption">Listarea 8-8: Iterarea prin referințe mutabile ale elementelor dintr-un vector</span>

Pentru a modifica valoarea la care se referă o referință mutabilă, folosim operatorul de dereferențiere `*` pentru a accesa valoarea efectivă din `i` înainte să aplicăm operatorul `+=`. Vom discuta mai detaliat despre operatorul de dereferențiere în secțiunea "Urmărind pointer-ul până la valoare cu ajutorul operatorului de dereferențiere"[deref]<!-- ignore --> din Capitolul 15.

Iterația printr-un vector, fie că face acces imutabil sau mutabil, este sigură datorită regulilor impuse de verificatorul de împrumuturi. Dacă încercăm să adăugăm sau să eliminăm elemente în timpul execuției unui ciclu `for`, așa cum se face în Listările 8-7 și 8-8, ne vom confrunta cu o eroare de compilare similară cu cea întâmpinată în Listarea 8-6. Referința la vector menținută de ciclul `for` previne orice modificare simultană asupra întregului vector.

### Folosirea unui enum pentru a combina mai multe tipuri într-un vector

Vectorii sunt limitați la stocarea valorilor de același tip, ceea ce poate fi restrictiv în anumite situații. Spre norocul nostru, variantele unui enum sunt grupate sub același tip de enum, permițându-ne să folosim un singur enum pentru a reprezenta elemente de tipuri diferite. Așadar, atunci când dorim să combinăm diverse tipuri într-o singură structură, putem apela la un enum!

Să presupunem că dorim să extragem valori dintr-un rând al unui tabel, unde coloanele acelui rând conțin întregi, numere în virgulă mobilă sau string-uri. Ne putem defini un enum cu variante pentru fiecare tip de valoare, iar aceste variante de enum vor fi considerate același tip: tipul enum-ului în sine. Putem crea apoi un vector care să păstreze acest enum și, în final, să cuprindă tipuri variate. Acest concept este ilustrat în Listarea 8-9.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

<span class="caption">Listarea 8-9: Crearea unui `enum` pentru stocarea valorilor de diferite tipuri într-un vector</span>

Rust trebuie să cunoască de la momentul compilării ce tipuri vor fi incluse în vector, astfel încât să aloce în mod corespunzător memoria necesară pe heap pentru fiecare element. De asemenea, trebuie să specificăm în mod explicit care tipuri sunt acceptate în vector. Dacă Rust ar permite unui vector să conțină orice tip, ar exista riscul ca unele tipuri să genereze erori în momentul operațiilor executate asupra elementelor. Utilizând un enum în combinare cu o expresie `match`, Rust garantează la compilare că fiecare caz posibil este luat în considerație, conform discuției din Capitolul 6.

Dacă nu știm dinainte toate tipurile de date cu care programul nostru va opera la executare, abordarea cu enum nu este aplicabilă. Ca alternativă, se poate folosi un obiect de tip trăsătură, subiect pe care îl vom aborda în Capitolul 17.

După ce am explorat unele dintre cele mai frecvente utilizări ale vectorilor, te încurajez să consulți [documentația API][vec-api]<!-- ignore --> pentru a descoperi multitudinea de metode utile definite pentru `Vec<T>` de către biblioteca standard. De exemplu, pe lângă metoda `push`, există și metoda `pop`, care elimină și returnează ultimul element din vector.

### Un vector își eliberează elementele la distrugere

Ca orice `struct`, un vector este eliberat automat când domeniul său de vizibilitate se încheie, după cum vedem în Listarea 8-10.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

<span class="caption">Listarea 8-10: Ilustrarea punctelor unde vectorul și elementele sale sunt eliberate</span>

Eliberarea unui vector implică și curățarea tuturor elementelor sale; în cazul nostru, numerele întregi vor fi de asemenea eliminate. Verificatorul de împrumuturi se asigură că referințele la elementele vectorului sunt folosite numai atât timp cât vectorul respectiv este în vigoare.

Acum să ne îndreptăm atenția spre următorul tip de colecție: `String`!

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: ../nomicon/vec/vec.html
[vec-api]: ../std/vec/struct.Vec.html
[deref]: ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator
