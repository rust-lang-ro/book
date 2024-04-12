## Utilizarea hash map-urilor pentru a asocia chei cu valori

Ultima dintre colecțiile fundamentale pe care le vom discuta este *hash map-ul*. Structura `HashMap<K, V>` permite stocarea unei asocieri între chei de tipul `K` și valori de tipul `V`, printr-o *funcție de hash* care stabilește cum sunt așezate aceste chei și valori în memorie. Această structură de date este cunoscută în multe limbaje de programare sub diverse nume precum hash, map, object, hash table, dictionary sau associative array.

Hash map-urile sunt de mare ajutor atunci când dorești să accesezi datele folosind o cheie de orice tip, nu un index, cum este cazul la vectori. De exemplu, într-un joc, poți urmări scorul diferitelor echipe folosind un hash map în care cheile sunt numele echipelor, iar valorile sunt scorurile acestora. Astfel, cu numele unei echipe, poți să obții rapid scorul aferent.

În această secțiune, vom explora funcționalitățile de bază ale hash map-urilor, dar biblioteca standard oferă multe alte capabilități interesante asociate cu `HashMap<K, V>`. Pentru detalii suplimentare, te încurajăm să consulți documentația bibliotecii standard.

### Cum să creezi un hash map nou

Pentru a inițializa un hash map gol, putem folosi metoda `new` și apoi adăugăm elemente prin metoda `insert`. În Listarea 8-20, monitorizăm scorurile a două echipe, denumite *Blue* și *Yellow*. Echipa Blue debutează cu 10 puncte, pe când Yellow începe jocul cu 50 de puncte.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

<span class="caption">Listarea 8-20: Inițializarea unui hash map nou și inserarea unor perechi cheie-valoare</span>

Este important de observat că, înainte de toate, trebuie să includem `HashMap` în sfera noastră de lucru prin intermediul instrucțiunii `use`, din secțiunea de colecții a bibliotecii standard. Dintre cele trei tipuri principale de colecții, hash map-ul este cel mai rar utilizat, motiv pentru care nu este importat implicit în preludiul Rust. Hash map-urile beneficiază, de asemenea, de un suport mai limitat din partea bibliotecii standard - nu există, de exemplu, un macro predefinit pentru construirea acestora.

Similar vectorilor, hash map-urile păstrează datele în heap. Acest `HashMap` utilizează chei de tip `String` și valori de tip `i32`. Și, precum în cazul vectorilor, has hmap-urile sunt colecții omogene - toate cheile trebuie să fie de același tip și toate valorile de alt tip, dar uniform între ele.

### Accesul la valorile dintr-un hash map

Noi putem extrage o valoare dintr-un hash map oferind cheia corespunzătoare metodei `get`, după cum se arată în Listarea 8-21.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

<span class="caption">Listarea 8-21: Obținerea scorului pentru echipa Albastră din hash map</span>

În acest exemplu, variabila `score` va prelua valoarea asociată echipei Albastre, care va fi `10`. Metoda `get` returnează `Option<&V>`, semnificând că dacă nu se află o valoare pentru cheia dată în hash map, `get` va oferi ca răspuns `None`. În acest program, opționalitatea este tratată prin folosirea metodei `copied`, care transformă `Option<&i32>` în `Option<i32>`, iar apoi, cu ajutorul metodei `unwrap_or`, setăm `score` la zero dacă hash map-ul `scores` nu conține o valoare pentru cheia respectivă.

Pentru a parcurge fiecare pereche cheie/valoare dintr-un hash map, noi putem folosi o abordare similară cu cea pentru vectori, aplicând o buclă `for`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

Acest fragment de cod va afișa fiecare pereche într-o ordine aleatoare:

```text
Galben: 50
Albastru: 10
```

### Hash map-uri și posesiunea datelor

Când inserăm date într-un hash map, ce se întâmplă cu acestea depinde de tipul lor. Dacă tipul de date implementează trăsătura `Copy`, cum este cazul lui `i32`, atunci valorile sunt duplicate și inserate în hash map fără a-și pierde originalul. Pe de altă parte, pentru tipuri de date cu posesiune unică, precum `String`, inserarea înseamnă transferul posesiunii: hash map-ul devine noul proprietar al acestor valori. Următoarea listare ilustrează această comportare:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

<span class="caption">Listarea 8-22: Ilustrarea modului în care hash map-ul devine posesorul cheilor și valorilor după ce sunt inserate</span>

După această operațiune de inserare, nu mai putem utiliza variabilele `field_name` și `field_value`; ele au fost permutate în hash map și acum hash map-ul deține drepturile asupra lor.

Dacă dorim să păstrăm drepturile asupra datelor și să inserăm doar referințe în hash map, va trebui să ne asigurăm că acele date originale rămân valabile pentru toată durata de viață a hash map-ului. Această temă este aprofundată în secțiunea despre [„Validarea referințelor prin durate de viață”][validating-references-with-lifetimes]<!-- ignore -->, pe care o vom explora în Capitolul 10.

### Cum actualizăm un hash map

Deși numărul de perechi cheie-valoare dintr-un hash map poate crește, fiecare cheie unică poate avea atribuită o singură valoare în același timp (ceea ce nu este valabil și invers; spre exemplu, echipa Blue și echipa Yellow ar putea ambele să aibă valoarea 10 în hash map-ul `scores`).

Când dorești să schimbi datele dintr-un hash map, trebuie să alegi cum să procedezi atunci când o cheie are deja atribuită o valoare. Există câteva opțiuni: poți înlocui valoarea veche cu cea nouă, ignorând complet valoarea anterioară. Poți păstra valoarea veche și să ignori pe cea nouă, adăugându-o doar dacă cheia nu are încă o valoare asignată. Sau, poți combina valoarea veche cu cea nouă. Să explorăm cum putem aplica fiecare dintre aceste metode!

#### Înlocuirea unei valori

Când adăugăm într-un hash map o pereche de cheie și valoare, iar mai apoi inserăm din nou aceeași cheie cu o valoare diferită, valoarea asociată cu cheia respectivă va fi înlocuită. Chiar dacă în codul prezentat în Listarea 8-23 executăm funcția `insert` de două ori, hash map-ul va conține o singură pereche de cheie și valoare, deoarece ambele inserări se referă la cheia echipei Blue.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

<span class="caption">Listarea 8-23: Înlocuirea unei valori asociate cu o anumită cheie</span>

Acest cod va afișa rezultatul `{"Blue": 25}`. Prima valoare, `10`, a fost înlocuită cu noua valoare, `25`.

<!-- Old headings. Do not remove or links may break. -->
<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### Adăugarea unei chei și valori numai dacă cheia lipsește

Un scenariu frecvent în lucrul cu hash map-uri este verificarea prezenței unei chei specifice înainte de a adăuga o valoare. Dacă cheia este deja prezentă, valoarea existentă trebuie să rămână neschimbată. În caz contrar, cheia și valoarea ei trebuie inserate în map.

Pentru aceasta, hash map-urile oferă o metodă specială numită `entry`, care primește ca parametru cheia de verificat. Rezultatul metodei `entry` este o enumerare `Entry` care indică dacă o valoare există sau nu pentru acea cheie. De exemplu, dacă vrem să verificăm dacă echipa Yellow are o valoare atribuită cheii sale și, în caz negativ, să inserăm valoarea 50; procedăm similar pentru echipa Blue. Utilizând API-ul `entry`, codul arată ca în Listarea 8-24.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

<span class="caption">Listarea 8-24: Utilizarea metodei `entry` pentru a insera valori doar când cheia nu există</span>

Metoda `or_insert` de pe `Entry` returnează o referință mutabilă la valoarea asociată cheii respective, dacă cheia există. Dacă nu, aceasta inserează valoarea primită ca parametru pentru cheia respectivă și returnează o referință mutabilă la noua valoare inserată. Această abordare este mult mai elegantă și mai eficientă decât să implementăm manual această logică, având și avantajul unei mai bune compatibilități cu verificatorul de împrumuturi.

Execuția codului din Listarea 8-24 va genera rezultatul `{"Yellow": 50, "Blue": 10}`. Prima invocare a metodei `entry` va adăuga cheia pentru echipa Yellow cu valoarea 50, deoarece aceasta nu are o valoare predefinită. Cea de-a doua invocare nu va modifica hash map-ul, deoarece echipa Blue dispune deja de valoarea 10.

#### Actualizarea unei valori în funcție de valoarea precedentă

Folosirea hash map-urilor pentru a actualiza valori pe baza celor anterioare este o practică obișnuită. De exemplu, în Listarea 8-25, codul prezentat numără cât de des apare fiecare cuvânt într-un text dat. Folosim un hash map cu cuvintele drept chei și creștem valoarea asociată pentru a contoriza numărul de apariții ale fiecărui cuvânt. Dacă întâlnim un cuvânt pentru prima dată, inserăm inițial valoarea 0.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

<span class="caption">Listarea 8-25: Numărarea aparițiilor cuvintelor cu ajutorul unui hash map care stochează cuvinte și contoare</span>

Acest cod va rezulta în afișarea `{"world": 2, "hello": 1, "wonderful": 1}`. Însă, ordinea acestor perechi cheie/valoare ar putea fi diferită, deoarece, așa cum am menționat în secțiunea [“Accesarea valorilor într-un hash map”][access]<!-- ignore -->, ordinea de iterație a unui hash map este întâmplătoare.

Metoda `split_whitespace` generează un iterator care oferă acces la sub-șirurile de caractere, separate prin spații, din `text`. Metoda `or_insert` furnizează o referință mutabilă (`&mut V`) la valoarea corespunzătoare cheii indicate. Stocăm această referință mutabilă în variabila `count` și, pentru a-i modifica valoarea, folosim dereferențierea cu ajutorul simbolului asterisc (`*`). Referința mutabilă își încheie domeniul de vizibilitate la finalul buclei `for`, astfel încât toate modificările sunt sigure și respectă regulile de împrumutare ale limbajului Rust.

### Funcția de hash în HashMap

Implicit, `HashMap` folosește o funcție de hash denumită *SipHash*. Aceasta oferă protecție împotriva atacurilor cibernetice de tip Denial of Service (DoS) care vizează tabelele de hash. *SipHash* nu este neapărat cel mai rapid algoritm de hash, însă avantajele în ceea ce privește securitatea compensează pentru scăderea în performanță. Dacă, în urma analizei performanței codului tău, descoperi că funcția de hash standard este prea înceată, poți opta pentru un alt algoritm de hash alegând un hasher diferit. Un *hasher* este un tip de date ce implementează trăsătura `BuildHasher`. Vom explora conceptul de trăsături și implementarea acestora mai detaliat în Capitolul 10. Nu este obligatoriu să scrii un hasher de la zero; pe [crates.io](https://crates.io/) găsești biblioteci create de comunitatea Rust care includ hasheri cu o varietate de algoritmi de hash cunoscuți.

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## Sumar

Aflăm că vectorii, string-urile și hash map-urile sunt esențiali pentru manipularea și gestionarea datelor în programe. În continuare, vei avea prilejul să aplici cunoștințele dobândite prin rezolvarea următoarelor sarcini:

* În cazul unei liste de numere întregi, folosește un vector pentru a determina mediana (elementul din mijlocul listei sortate) și moda (elementul care apare de cele mai multe ori; un hash map va fi de ajutor aici).
* Transformă string-urile în pig latin în felul următor: mută prima consoană a fiecărui cuvânt la sfârșitul acestuia și adaugă sufixul "ay", ca în "first" care devine "irst-fay". Cuvintele ce încep cu vocală primesc sufixul "hay" la final ("apple" se transformă în "apple-hay"), având în vedere specificitățile codificării UTF-8.
* Utilizând un hash map și vectori, dezvoltă o interfață text prin care un utilizator poate adăuga nume de angajați la un anumit departament într-o firmă, de exemplu "Adaugă pe Sally în departamentul de Inginerie" sau "Adaugă pe Amir în departamentul de Vânzări". În plus, asigură posibilitatea de a obține lista angajaților unui departament sau lista completă a angajaților companiei, împărțită pe departamente, organizată alfabetic.

Documentația API a bibliotecii standard detaliază metodele disponibile pentru vectori, string-uri și hash map-uri, care îți vor fi utile pentru a completa aceste exerciții.

Pe măsură ce programarea se complică și operațiunile pot eșua, este esențial să învățăm despre gestionarea erorilor. Este subiectul pe care îl vom aborda în continuare!

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
