## Utilizarea `Box<T>` pentru a arăta spre date situate pe heap

Cel mai simplu smart pointer este *boxa*, ale cărei tip este notat ca `Box<T>`. Boxele permit stocarea datelor pe heap în loc de pe stivă. Ce rămâne pe stivă este pointerul către datele de pe heap. Pentru a revizui diferența dintre stivă și heap, putem reveni la Capitolul 4.

Boxele nu aduc un supra-cost de performanță decât în ceea ce privește stocarea datelor pe heap în locul stivei. Dar, ele nu oferă nici multe funcționalități extra. Vom utiliza boxele cel mai frecvent în următoarele situații:

* Atunci când deținem un tip a cărui dimensiune nu poate fi stabilită în timpul compilării și dorim să utilizăm o valoare de acest tip într-un context care necesită o dimensiune exactă
* Atunci când avem o cantitate mare de date și dorim să transferăm posesiunea, dar să ne asigurăm că datele nu vor fi copiate în acest proces
* Atunci când dorim să deținem o valoare și ne interesează doar ca aceasta să implementeze o anumită trăsătură, mai degrabă decât să fie de un tip anume

Primul caz va fi demonstrat în secțiunea [“Activarea Tipurilor Recursive folosind Boxe”](#enabling-recursive-types-with-boxes)<!-- ignore -->. În al doilea caz, transferul posesiunii asupra unei cantități mari de date poate dura mult deoarece datele sunt copiate pe stivă. Pentru îmbunătățirea performanței în această situație, putem stoca acele date pe heap într-o boxă. Astfel, doar o mică parte din datele pointerului sunt copiate pe stivă, pe când datele la care se referă rămân într-un singur loc pe heap. Al treilea caz este cunoscut sub numele de *obiect-trăsătură*, iar Capitolul 17 consacră o întreagă secțiune, [“Utilizarea obiectelor-trăsătură ce permit valori de tipuri diverse,”][trait-objects]<!-- ignore --> exclusiv acestui subiect. Deci, tot ce învățăm aici va fi aplicat din nou în Capitolul 17!

### Utilizarea unei `Box<T>` pentru a stoca date pe heap

Înainte de a discuta cazul de utilizare a `Box<T>` pentru stocare pe heap, vom revizui sintaxa și cum interacționăm cu valorile închise într-o `Box<T>`.

Listarea 15-1 ilustrează cum să folosești o boxă pentru a păstra o valoare `i32` pe heap:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

<span class="caption">Listarea 15-1: Stocarea unei valori `i32` pe heap utilizând o boxă</span>

Definim variabila `b` cu valoarea unui `Box` care indică spre valoarea `5`, alocată pe heap. Acest program va afișa `b = 5`; în acest context, putem accesa datele din boxă într-un mod asemănător cu cel în care am accesa datele dacă ar fi pe stivă. Așa cum se întâmplă cu orice valoare deținută, când un `Box` iese din domeniul de vizibilitate, cum se întâmplă pentru `b` la finalul `main` function, se va proceda la dealocarea acestuia. Dealocarea are loc atât pentru boxă (pe stivă), cât și pentru datele la care face referire (pe heap).

A așeza o singură valoare pe heap nu este frecvent utilă, prin urmare utilizarea în izolare a boxelor în acest mod nu este des întâlnită. În majoritatea situațiilor, este mai potrivit să avem valori precum o instanță `i32` pe stivă, unde în mod implicit sunt stocate. Să examinăm acum o situație în care boxele ne permit să definim tipuri care în alt mod n-ar fi posibile fără existența boxelor.

### Permiterea tipurilor recursive cu boxe

O valoare de tip recursiv poate include în ea însăși o altă valoare de același tip. Tipurile recursive creează o problemă în Rust deoarece, la momentul compilării, este necesar să se știe cât spațiu ocupă fiecare tip. Însă, întrepătrunderea valorilor de tipuri recursive teoretic nu are sfârșit, astfel Rust nu poate deduce cât spațiu va fi necesar. Utilizarea boxelor, care au o dimensiune fixă cunoscută, ne permite să activăm tipurile recursive prin introducerea unei boxe în definiția tipului recursiv.

Drept exemplu de tip recursiv, să analizăm *lista cons*. Aceasta este un tip de date des întâlnit în limbajele de programare funcțională. Tipul listei cons pe care îl vom defini este simplu, mai puțin partea recursivă; deci, conceptele din exemplul cu care vom lucra ne vor fi de folos în orice moment când ne confruntăm cu situații mai complexe ce implică tipuri recursive.

#### Mai multe informații despre lista cons

O *cons list* (listă cons) este o structură de date originară din limbajul de programare Lisp și din dialectele acestuia, alcătuită din perechi încapsulate și reprezintă versiunea în Lisp a unei liste înlănțuite. Numele provine de la funcția `cons` (prescurtarea pentru “construct function”) din Lisp, care construiește o nouă pereche pe baza celor doi parametri. Prin apelarea recursivă a funcției `cons` pe o pereche formată dintr-o valoare și o altă pereche, putem crea liste cons alcătuite din perechi recursive.
  Iată un exemplu de reprezentare în pseudocod a unei liste cons care include secvența 1, 2, 3; fiecare pereche fiind închisă în paranteze:
 
```text
(1, (2, (3, Nil)))
```
  Fiecare element al unei liste cons cuprinde două componente: valoarea actuală și următorul element din listă. Ultimul element conține numai o valoare numită `Nil`, fără un succesor. O listă cons este generată prin apeluri recursive ale funcției `cons`. Termenul recunoscut universal pentru a desemna situația inițială a recursivității este `Nil`. Este important de subliniat că acesta nu coincide cu noțiunea de „null” sau „nil” menționată în Capitolul 6, ce se referă la o valoare nevalidă sau lipsă.

Lista cons nu este o structură de date des întâlnită în Rust. Majoritatea timpului, când avem de-a face cu o listă de elemente în Rust, opțiunea `Vec<T>` se dovedește a fi mai practică. Alte structuri de date recursive mai complexe *sunt* utile în situații variate, dar introducerea noțiunii de listă cons în acest capitol ne permite să explorăm cum boxele ne ajută să definim un tip de date recursiv, fără alte distrageri.

Listarea 15-2 conține definiția unui enum pentru o listă de tip cons. Observăm că acest cod nu o să se compileze încă, pentru că tipul `List` nu are o mărime cunoscută, lucru pe care îl vom demonstra.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

<span class="caption">Listarea 15-2: Prima încercare de definire a unui enum pentru a reprezenta o structură de date listă cons de valori `i32`</span>

> Notă: Implementăm o listă cons care conține numai valori `i32` pentru acest
> exemplu. Am fi putut să utilizăm generici, așa cum am discutat în Capitolul
> 10, pentru a defini un tip de listă cons capabil să stocheze valori de orice
> tip.

Utilizând tipul `List` pentru a stoca lista `1, 2, 3` arată ca și codul din Listarea 15-3:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

<span class="caption">Listarea 15-3: Folosind enum-ul `List` pentru a stoca lista `1, 2, 3`</span>

Prima valoare `Cons` păstrează `1` și încă o valoare `List`. Această valoare `List` este alte o valoare `Cons` care păstrează `2` și încă o valoare `List`. Ultima valoare `List` este încă o valoare `Cons` care păstrează `3` și o valoare `List`, care până la urmă este `Nil`, varianta non-recursivă ce semnalează finalul listei.

Dacă încercăm să compilăm codul din Listarea 15-3, vom întâmpina eroarea afișată în Listarea 15-4:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

<span class="caption">Listarea 15-4: Eroarea întâlnită când încercăm să definim un enum recursiv</span>

Eroarea indică că acest tip "are dimensiune infinită". Acest lucru se întâmplă deoarece am definit `List` cu o variantă care este recursivă, aceasta conținând direct altă valoare de tipul său. Drept rezultat, Rust nu poate stabili cât spațiu e necesar pentru a stoca o valoare de tip `List`. Să analizăm de ce apare această eroare. În primul rând, să vedem cum Rust decide cât spațiu e necesar pentru a stoca o valoare de tip non-recursiv.

#### Calcularea dimensiunii unui tip non-recursiv

Reamintim enum-ul `Message` definit în Listarea 6-2, unde am analizat definițiile enum-urilor în Capitolul 6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

Pentru a calcula cât spațiu este necesar pentru o valoare de tip `Message`, Rust inspectează fiecare variantă pentru a determina care variază cel mai mult în dimensiune. Rust observă că `Message::Quit` nu ocupă spațiu, `Message::Move` necesită destul spațiu pentru două valori `i32`, etc. Cum doar una dintre variante va fi utilizată, cantitatea maximă de spațiu pe care o valoare `Message` o poate ocupa este dată de dimensiunea celei mai mari variante.

Prin contrast, observăm ce se întâmplă când Rust încearcă să determine cât spațiu este necesar pentru un tip recursiv, cum ar fi enum-ul `List` din Listarea 15-2. Compilatorul începe analiza cu varianta `Cons`, care include o valoare `i32` și una de tip `List`. Astfel, `Cons` necesită un spațiu egal cu dimensiunea unui `i32` adăugată la dimensiunea unui `List`. Pentru a deduce cât spațiu îi trebuie tipului `List`, compilatorul se uită la variante, pornind de la `Cons`. Aceasta conține o valoare `i32` și una `List`, iar această recursivitate continuă ad infinitum, așa cum e prezentat în Figura 15-1.

<span class="caption">Figura 15-1: O listă `List` infinită compusă din variante `Cons` la infinit</span>

#### Folosirea lui `Box<T>` pentru a realiza un tip recursiv cu dimensiunea cunoscută

Pentru că Rust nu este capabil să calculeze automat cât spațiu de memorie trebuie alocat pentru tipurile definite recursiv, compilatorul va arăta o eroare cu următoarea sugestie utilă:

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to make `List` representable
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

Aici, „un nivel de indirecție” sugerează că în loc să stocăm direct o valoare, ar trebui să modificăm structura de date astfel încât valoarea să fie stocată în mod indirect, prin intermediul unui pointer care să indice către acea valoare.

Din moment ce un `Box<T>` este un pointer, Rust întotdeauna va ști cât spațiu necesită un `Box<T>`: mărimea unui pointer rămâne constantă, indiferent de volumul de date la care face referire. Asta înseamnă că putem utiliza o boxă `Box<T>` în varianta `Cons` pe locul unei valori `List` directe. `Box<T>` va referi la următoarea intrare `List`, care va fi amplasată în heap și nu direct în varianta `Cons`. În esență, avem în continuare o listă, alcătuită din liste care conțin alte liste, însă această implementare este acum mai aproape de ideea de a așeza elementele unul lângă celălalt decât unul în altul.

Putem modifica definiția enum-ului `List` din Listarea 15-2 și utilizarea `List` din Listarea 15-3 cu codul din Listarea 15-5, care se va compila:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

<span class="caption">Listarea 15-5: Definiția lui `List` care folosește `Box<T>` pentru a asigura o dimensiune cunoscută</span>

Varianta `Cons` necesită dimensiunea unui `i32` plus spațiul necesar pentru a stoca datele pointer-ului boxei. Varianta `Nil` nu stochează nicio valoare, deci ocupă mai puțin spațiu decât varianta `Cons`. Acum știm că orice valoare de tip `List` va ocupa dimensiunea unui `i32` plus dimensiunea datelor pointer-ului unei boxe. Folosind o boxe am întrerupt lanțul infinit, recursiv, permițând astfel compilatorului să calculeze dimensiunea de care are nevoie pentru a stoca o valoare `List`. Figura 15-2 ilustrează aspectul curent al variantei `Cons`.

<img alt="O listă cons finită" src="img/trpl15-02.svg" class="center" />

<span class="caption">Figura 15-2: O listă `List` care nu este de dimensiuni infinite deoarece `Cons` conține un tip `Box`</span>

Boxele oferă doar indirectare și alocarea memoriei pe heap; ele nu dispun de alte capabilități speciale, asemenea celor pe care le vom examina la alte categorii de pointeri inteligenți. Totodată, ele nu implică o supraplată de performanță asociată acestor capabilități speciale, fiind astfel folositoare în situații precum lista cons, unde indirectarea este singurul atribut necesar. Vom analiza mai multe întrebuințări pentru boxe și în Capitolul 17.

Tipul `Box<T>` este considerat un pointer inteligent deoarece implementează trăsătura `Deref`, ceea ce îi permite lui `Box<T>` să fie tratat ca o referință. Când o valoare de tip `Box<T>` iese din domeniul de vizibilitate, datele de pe heap la care indică boxa sunt și ele eliberate, datorită implementării trăsăturii `Drop`. Aceste două trăsături sunt și mai importante pentru funcționalitățile oferite de celelalte tipuri de pointeri inteligenți pe care le vom discuta în restul capitolului. Să ne aprofundăm cunoștințele despre aceste două trăsături.

[trait-objects]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
