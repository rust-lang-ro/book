## Utilizarea pointerilor inteligenți la fel ca referințele obișnuite cu trăsătura `Deref`

Când implementăm trăsătura `Deref`, personalizăm comportamentul *operatorului de dereferențiere* `*` (care nu trebuie confundat cu operatorul de înmulțire sau cu operatorul glob). Prin implementarea `Deref` în așa fel încât un pointer inteligent să poată fi tratat ca o referință obișnuită, poți scrie cod care operează pe referințe și să folosești același cod cu pointeri inteligenți.

Să începem prin a examina modul în care operatorul de dereferențiere funcționează cu referințe convenționale. Apoi, vom *defini* un nou tip care se comportă similar cu `Box<T>`, observând de ce operatorul de dereferențiere nu își păstrează comportamentul obișnuit asupra acestui tip proaspăt definit. Vom descoperi cum implementarea trăsăturii `Deref` permite pointerilor inteligenți să funcționeze similar cu referințele. În continuare, vom explora funcția de *coerciție deref* oferită de Rust și modul în care aceasta ne facilitează lucrul fie cu referințe, fie cu pointeri inteligenți.

> Notă: există o diferență majoră între tipul `MyBox<T>` ce urmează să-l
> construim și `Box<T>` autentic: varianta noastră nu va plasa datele pe heap.
> Concentrându-ne pe exemplificarea `Deref`, locația exactă unde datele sunt
> stocate devine secundară în fața comportamentului tipic de pointer.

<!-- Old link, do not remove --> <a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>

### Urmărind Pointer-ul până la Valoarea sa

Un referință obișnuită este un fel de pointer, și putem gândi un pointer ca fiind o săgeată ce arată spre o valoare păstrată altundeva. În Listarea 15-6, inițiem o referință la o valoare `i32` și apoi aplicăm operatorul de dereferențiere pentru a ajunge la valoarea la care face referința:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

<span class="caption">Listarea 15-6: Utilizarea operatorului de dereferențiere pentru a urma o referință către o valoare `i32`</span>

Variabila `x` are deține o valoare `i32`, mai exact `5`. Alocăm `y` să fie o referință la `x`. Ne putem asigura că `x` este egal cu `5`. Însă, dacă dorim să argumentăm ceva despre valoarea către care `y` face referință, trebuie să folosim `*y` pentru a urmări referința până la valoarea respectivă (de aici termenul *dereferențiere*), ceea ce îi va permite compilatorului să compare valoarea actuală. Odată ce dereferențiem `y`, obținem accesul la valoarea numerică la care `y` face referință și pe care o putem compara cu `5`.

Dacă am opta să folosim `assert_eq!(5, y);`, am întâmpina această eroare de compilare:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

Nu este permis să comparam un număr cu o referință la un număr, deoarece sunt de tipuri diferite. Trebuie să utilizăm operatorul de dereferențiere pentru a accesa valoarea la care face referința.

### Utilizarea `Box<T>` la fel ca o referință

Putem rescrie codul din Listarea 15-6 astfel încât să folosim `Box<T>` în loc de o referință; operatorul de dereferențiere aplicat asupra lui `Box<T>` în Listarea 15-7 funcționează exact ca și operatorul de dereferențiere folosit asupra referinței în Listarea 15-6:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

<span class="caption">Listarea 15-7: Utilizarea operatorului de dereferențiere pe un `Box<i32>`</span>

Principala diferență între Listarea 15-7 și Listarea 15-6 este că aici `y` este inițializat ca o instanță de `Box<T>` care conține o copie a valorii lui `x`, nu ca o referință la valoarea lui `x`. În aserțiunea finală, folosim operatorul de dereferențiere pentru a accesa pointerul din `Box<T>` la fel cum am făcut când `y` era o referință. În continuare, vom descoperi ce anume face `Box<T>` special și ne permite să folosim operatorul de dereferențiere, prin definirea propriului nostru tip de date.

### Definirea propriului nostru pointer inteligent

Să dezvoltăm un pointer inteligent asemănător cu tipul `Box<T>` oferit de biblioteca standard, pentru a înțelege cum, în mod implicit, se comportă pointerii inteligenți diferit de referințe. Mai apoi, vom explora cum adăugăm funcționalitatea pentru a folosi operatorul de dereferențiere.

Tipul `Box<T>` este în esență definit ca un struct-tuplă cu un singur element, așadar Listarea 15-8 definește un tip `MyBox<T>` într-un mod similar. De asemenea, vom defini o funcție `new`, ca să corespundă funcției `new` definită pentru `Box<T>`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

<span class="caption">Listarea 15-8: Definirea tipului `MyBox<T>`</span>

Definim un struct cu numele `MyBox` și declarăm un parametru generic `T`, deoarece dorim ca acest tip să poată conține valori de diverse tipuri. Tipul `MyBox` este un struct-tuplă ce are un singur element de tipul `T`. Funcția `MyBox::new` acceptă un parametru de tip `T` și returnează o instanță de `MyBox` care conține valoarea primită.

Să încercăm să adăugăm funcția `main` din Listarea 15-7 la Listarea 15-8 și să o schimbăm pentru a folosi tipul `MyBox<T>` definiț de noi, în loc de `Box<T>`. Codul din Listarea 15-9 nu va compila deoarece Rust nu recunoaște cum să dereferențieze `MyBox`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

<span class="caption">Listarea 15-9: Tentativa de a utiliza `MyBox<T>` așa cum am folosit referințele și `Box<T>`</span>

Iată eroarea de compilare care rezultă:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

Tipul nostru `MyBox<T>` nu poate fi dereferențiat deoarece nu am implementat încă această capabilitate pe el. Pentru a activa posibilitatea dereferențierii cu operatorul `*`, trebuie să implementăm trăsătura `Deref`.

### Tratarea unui tip ca o referință prin implementarea trăsăturii `Deref`

Așa cum am discutat în secțiunea [„Implementarea unei trăsături pe un tip”][impl-trait]<!-- ignore --> din Capitolul 10, pentru a implementa o trăsătură, trebuie să oferim implementări pentru toate metodele necesare ale acelei trăsături. Trăsătura `Deref`, furnizată de biblioteca standard, ne solicită implementarea unei metode numite `deref` care împrumută `self` și returnează o referință către datele interne. Listarea 15-10 include o implementare a `Deref` pe care o adăugăm la definiția `MyBox`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

<span class="caption">Listarea 15-10: Implementarea `Deref` pentru `MyBox<T>`</span>

Sintaxa `type Target = T;` definește un tip asociat pe care trăsătura `Deref` îl va folosi. Tipurile asociate sunt un alt mod de a declara un parametru generic, însă nu este necesar să ne concentrăm asupra lor în acest moment, deoarece vor fi discutate în detaliu în Capitolul 19.

Completăm corpul metodei `deref` cu `&self.0`, astfel încât metoda `deref` să returneze o referință la valoarea la care vrem să avem acces utilizând operatorul `*`. Așa cum am revăzut în secțiunea [„Folosirea structurilor tuple fără câmpuri nominale pentru a crea tipuri diferite”][tuple-structs]<!-- ignore --> din Capitolul 5, `.0` permite accesul la prima valoare dintr-o structură de tip tuple. Funcția `main` din Listarea 15-9, care aplică operatorul `*` asupra unei valori `MyBox<T>`, acum se compilează corect și aserțiunile trec cu succes.

Fără trăsătura `Deref`, compilatorul este capabil să dereferențieze doar referințe de tip `&`. Metoda `deref` îi permite compilatorului să preia o valoare de orice tip care implementează `Deref` și să invoce metoda `deref` pentru a obține o referință de tip `&`, cu care știe să opereze prin dereferențiere.

Atunci când am folosit `*y` în Listarea 15-9, Rust a efectuat în spate această instrucțiune:

```rust,ignore
*(y.deref())
```

Limbajul Rust înlocuiește operatorul `*` cu un apel la metoda `deref`, urmat de o dereferențiere simplă, permițându-ne astfel să nu ne îngrijorăm despre necesitatea apelării metodei `deref`. Aceasta este o facilitate a Rust care ne îngăduie să scriem cod care se comportă la fel, fie că avem o referință obișnuită sau un tip care implementează `Deref`.

Motivul pentru care metoda `deref` returnează o referință spre o valoare, și faptul că este încă necesară dereferențierea simplă în afara parantezelor în `*(y.deref())`, este legat de sistemul de posesiune. În cazul în care metoda `deref` ar elibera valoarea în mod direct, în loc de o referință la aceasta, valoarea ar fi permutată din `self`. Nu ne dorim să preluăm posesiunea valorii interne din `MyBox<T>` în acest caz, nici în majoritatea situațiilor când folosim operatorul de dereferențiere.

Este important de reținut faptul că operatorul `*` este substituit cu un apel la metoda `deref`, urmat de utilizarea a doar o singură dată a operatorului `*`, ori de câte ori introducem un `*` în codul nostru. Datorită faptului că înlocuirea operatorului `*` nu se repetă la infinit, obținem în final date de tipul `i32`, care corespunde cu valoarea `5` folosită în `assert_eq!` din Listarea 15-9.

### Coerciția implicită Deref în funcții și metode

Coerciția Deref convertește o referință la un tip ce implementează trăsătura `Deref` într-o referință la alt tip. De exemplu, coerciția Deref poate converti `&String` în `&str` datorită faptului că `String` implementează trăsătura `Deref` astfel încât să returneze `&str`. Rust aplică automat coerciția Deref la argumentele funcțiilor și metodelor, dar numai pe acele tipuri care implementează trăsătura `Deref`. Aceasta are loc când pasăm o referință către o valoare de un anumit tip ca argument unei funcții sau metode ale cărei tip de parametru nu corespunde cu cel din definiția funcției sau a metodei. O succesiune de apelări ale metodei `deref` transformă tipul introdus în cel necesar parametrului.

Coerciția Deref a fost adăugată în Rust pentru a-i scuti pe programatori de necesitatea de a adăuga un număr mare de referințe și dereferințe explicite prin utilizarea `&` și `*`. Anume această funcționalitate ne oferă și posibilitatea de a scrie cod care este compatibil atât cu referințele, cât și cu pointerii inteligenți.

Pentru a vedea coerciția Deref în acțiune, vom folosi tipul `MyBox<T>` pe care l-am definit în Listarea 15-8 și implementarea `Deref` adăugată în Listarea 15-10. Listarea 15-11 ne arată cum să definim o funcție cu un parametru care este o secțiune de string:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

<span class="caption">Listarea 15-11: Funcția `hello` cu parametrul `name` de tip `&str`</span>

Putem apela funcția `hello` cu o secțiune de string ca argument, de pildă `hello("Rust");`. Coerciția Deref ne face posibilă apelarea funcției `hello` cu o referință la o valoare de tip `MyBox<String>`, așa cum este ilustrat în Listarea 15-12:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

<span class="caption">Listarea 15-12: Apelul funcției `hello` cu o referință la o valoare `MyBox<String>`, posibil datorită coerciției deref</span>

În acest context, invocăm funcția `hello` cu argumentul `&m`, care este o referință la o valoare `MyBox<String>`. Având implementată trăsătura `Deref` pentru `MyBox<T>` în Listarea 15-10, Rust poate să convertească `&MyBox<String>` în `&String` prin apelarea lui `deref`. Biblioteca standard conține o implementare pentru `Deref` aplicată la `String`, care returnează o secțiune de string, detaliu prezent în documentația pentru API-ul `Deref`. Rust folosește `deref` încă o dată pentru a schimba `&String` în `&str`, potrivindu-se astfel cu parametrii funcției `hello`.

Fără implementarea coerciției deref de către Rust, ar trebui să utilizăm codul din Listarea 15-13, în locul celui din Listarea 15-12, pentru a apela funcția `hello` cu un argument de tip `&MyBox<String>`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

<span class="caption">Listarea 15-13: Codul necesar în lipsa coerciției deref în Rust</span>

Utilizând `(*m)` se realizează dereferențierea lui `MyBox<String>` într-un `String`. După aceea, `&` și `[..]` extrag o secțiune din `String` corespunzătoare întregului string, pentru a se potrivi cu definiția funcției `hello`. Fără utilizarea coercițiilor deref, acest cod este mai greu de citit, scris și înțeles, având în vedere multiplele simboluri implicate. Coerciția deref permite Rust să efectueze aceste transformări automat.

Atunci când trăsătura `Deref` este definită pentru tipurile în discuție, Rust va analiza tipurile și va aplica metoda `Deref::deref` atât de des cât este necesar pentru a obține o referință care se potrivește cu tipul parametrului. Determinarea numărului de aplicări ale `Deref::deref` are loc în timpul compilării, eliminând astfel orice penalitate asupra timpului de execuție pentru a beneficia de avantajele coerciției deref!

### Cum interacționează coerciția Deref cu mutabilitatea

Similar cu modul în care utilizăm trăsătura `Deref` pentru a redefini operatorul `*` pe referințele imutabile, putem folosi trăsătura `DerefMut` pentru a redefini operatorul `*` pe referințele mutabile.

Rust aplică coerciția Deref când întâlnește tipuri și implementări de trăsături în trei situații:

* De la `&T` la `&U` atunci când `T: Deref<Target=U>`
* De la `&mut T` la `&mut U` atunci când `T: DerefMut<Target=U>`
* De la `&mut T` la `&U` atunci când `T: Deref<Target=U>`

Primele două situații sunt practic identice, cu excepția faptului că a doua situație implică mutabilitate. În primul caz, este specificat că dacă ai un `&T`, și `T` implementează `Deref` spre un tip `U`, poți obține un `&U` într-o manieră transparentă. În al doilea caz, se precizează că același proces de coerciție Deref este valabil și pentru referințele mutabile.

Al treilea caz este mai subtil: Rust va converti, de asemenea, o referință mutabilă într-una imutabilă. Totuși, inversul nu este posibil: referințele imutabile nu vor deveni niciodată referințe mutabile. Conform regulilor de împrumut, dacă deținem o referință mutabilă, aceasta trebuie să fie unicul indicator către datele respective (altfel, programul nu s-ar compila). Transformarea unei referințe mutabile în una imutabilă nu încalcă regulile de împrumut. Pe de altă parte, transformarea unei referințe imutabile în una mutabilă ar implica că referința imutabilă inițială este singura de acest tip către date, însă regulile de împrumut nu asigură această situație. De aceea, Rust nu poate presupune că transformarea unei referințe imutabile într-una mutabilă este fezabilă.

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type [tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
