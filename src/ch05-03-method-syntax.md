## Sintaxa pentru metode

*Metodele* sunt similare cu funcțiile. Le definim folosind cuvântul-cheie `fn`, urmat de un nume. Metodele pot avea parametri și pot returna o valoare, exact ca funcțiile. De asemenea, ele conțin un bloc de cod care este executat atunci când metoda este invocată de undeva. 

Totuși, există o diferență importantă: spre deosebire de funcții, metodele sunt definite în cadrul unei structuri (sau a unei enumerări ori a unui obiect de tip trăsătură, subiecte pe care le vom aborda în [Capitolul 6][enums]<!-- ignore --> și [Capitolul
17][trait-objects]<!-- ignore -->, respectiv). Primul lor parametru este întotdeauna `self`, care reprezintă instanța curentă a structurii pe care este invocată metoda.

### Definirea metodelor

Vom modifica funcția `area` care primește o instanță `Rectangle` ca parametru și vom crea o metodă `area`, definită direct pe structura `Rectangle`. Această modificare este prezentată în Listarea 5-13.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

<span class="caption">Listarea 5-13: Definirea metodei `area` asociate cu structura `Rectangle`</span>

Pentru a adăuga funcția în cadrul `Rectangle`, vom începe prin a iniția un bloc de implementare `impl` specific pentru `Rectangle`. Orice element care va fi inclus în acest bloc `impl` va fi asociat direct cu tipul `Rectangle`. Apoi, funcția `area` trebuie să fie mutată în interiorul acoladelor blocului `impl`. Primul parametru (unicul, în acest caz) va deveni `self`, atât în semnătura funcției, cât și în corpul acesteia. În funcția `main`, unde anterior funcția `area` era apelată cu `rect1` ca argument, vom utiliza în schimb *sintaxa metodei*. Astfel, metoda `area` va fi apelată direct pe instanța `Rectangle`. Sintaxa metodei intervine după instanță: adăugăm un punct, urmat de numele metodei, paranteze și oricare argumente necesare.

În semnătura pentru funcția `area`, am preferat să folosim `&self` în loc de `rectangle: &Rectangle`. De fapt, `&self` nu este altceva decât o formă prescurtată a expresiei `self: &Self`. În cadrul unui bloc `impl`, `Self` denotă tipul pentru care este destinat blocul `impl`. Orice metodă trebuie să aibă un parametru denumit `self` de tipul `Self` ca primul său parametru. Rust ne permite să simplificăm aceasta folosind doar termenul `self` ca prim parametru. Este important de remarcat că `&` trebuie să precedă scurtătura `self` pentru a indica faptul că această metodă împrumută instanța `Self`, exact cum am procedat în `rectangle: &Rectangle`. Metodele pot lua posesia `self`, pot împrumuta `self` în mod imutabil, cum este cazul de față, sau pot împrumuta `self` în mod mutabil, exact ca în cazul oricărui alt parametru.

Am optat pentru `&self` din același motiv pentru care am folosit `&Rectangle` în cazul funcției: nu dorim să preluăm posesiunea și aspirăm să citim doar datele din structură, nu și să scriem în aceasta. Dacă intenția noastră ar fi fost să modificăm instanța pe care am invocat-o în cadrul metodei, am fi folosit `&mut self` ca prim parametru. Este destul de neobișnuit să avem o metodă care preia posesiunea instanței folosind doar `self` ca prim parametru. În mod obișnuit, ne folosim de acest procedeu când metoda transformă `self` în ceva diferit, iar scopul nostru este de a împiedica autorul apelului să folosească instanța originală după finalizarea transformării.

Motivul principal de a opta pentru metode în detrimentul funcțiilor, dincolo de a oferi sintaxa metodei și de a evita repetarea tipului `self` în fiecare semnătură a metodei, este legat de organizare. Practic, am concentrat într-un singur bloc `impl` tot ceea ce putem realiza cu o instanță a unui anumit tip. Astfel, nu obligăm viitorii utilizatori ai codului nostru să caute capacitățile `Rectangle` împrăștiate în diverse colțuri ale bibliotecii pe care o furnizăm.

Trebuie să reții că avem libertatea de a denumi o metodă identic cu unul dintre câmpurile structurii. De exemplu, putem defini o metodă în structura `Rectangle` și o putem numi tot `width`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

În acest caz, optăm să configurăm metoda `width` astfel încât să returneze `true` dacă valoarea din câmpul `width` al instanței este mai mare decât `0` și `false` în caz contrar, când valoarea este `0`. Este important de subliniat faptul că avem libertatea de a utiliza un câmp în cadrul unei metode cu același nume, în funcție de necesități. De exemplu, în funcția `main`, atunci când denumirea `rect1.width` este urmată de paranteze, compilatorul Rust înțelege că ne referim la metoda `width`. În schimb, în absența parantezelor, Rust interpretează că ne referim la câmpul `width`.

Deși nu este întotdeauna cazul, frecvent, atunci când atribuim unei metode același nume cu un câmp, intenția este ca aceasta să returneze exclusiv valoarea din acel câmp, fără a efectua alte operații. Asemenea metode sunt denumite *getteri*. Este important de subliniat faptul că, spre deosebire de unele limbaje de programare, Rust nu generează în mod automat asemenea getteri pentru câmpurile structurilor. Getterii se dovedesc a fi utili deoarece permit transformarea câmpului într-unul privat și a metodei într-una publică, oferind astfel accesul în regim doar-de-citire a acelui câmp, rol ce intră în alcătuirea API-ului public al tipului respectiv. Conceptele de public și privat, precum și modalitatea de a desemna un câmp sau o metodă ca fiind publice sau private, vor fi detaliate în [Capitolul 7][public]<!-- ignore -->.

> ### Care este echivalentul operatorului `->` în Rust?

> În limbajele de programare C și C++, sunt folosiți doi operatori diferiți
> pentru apelarea metodelor: se utilizează `.` atunci când metoda este apelată
> direct pe obiect, iar `->` este folosit dacă metoda este apelată pe un pointer
> către obiect, caz în care este necesar să se realizeze o dereferențiere a
> pointerului. Adică, dacă `object` este un pointer, atunci
> `object->something()` este similar cu `(*object).something()`.
>
> Rust nu are un echivalent direct pentru operatorul `->`. În schimb, oferă o
> caracteristică numită *referențiere și dereferențiere automată*. Această
> caracteristică este aplicată în contextul apelării metodelor, unul dintre
> puținele locuri în Rust unde se întâmplă acest lucru.
>
> Iată cum funcționează: când apelezi o metodă cu `object.something()`, Rust
> include automat operatorii `&`, `&mut`, sau `*` pentru a se asigura că
> `object` corespunde semnăturii metodei. Așadar, următoarele două apeluri de
> metode sunt echivalente:
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> Primul mod de apelare pare mult mai elegant. Acest procedeu de referențiere
> automată este posibil datorită faptului că metodele au un receptor distinct -
> tipul `self`. Cunoscând receptorul și numele unei metode, Rust poate deduce cu
> precizie dacă metoda este de citire (`&self`), modificare (`&mut self`), sau
> consumă (`self`) receptorul. Aceasta abordare implicită de împrumutare pentru
> receptorii de metode contribuie semnificativ la eficiența posesiunii în
> practică.

### Metode cu parametri multipli

Să ne familiarizăm mai bine cu metodele prin crearea unei a doua metode în structura `Rectangle`. În acest caz, vrem ca o instanță a `Rectangle` să poată primi o altă instanță de `Rectangle` și să returneze `true` dacă al doilea `Rectangle` poate fi încorporat integral în interiorul primului (`self`); în caz contrar, ar trebui să returneze `false`. Mai pe scurt, după ce am definit metoda `can_hold`, dorim să fim în măsură să scriem programul prezentat în Listarea 5-14.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

<span class="caption">Listarea 5-14: Apelarea metodei încă nedefinite `can_hold`</span>

Avem următoarea ieșire anticipată, deoarece dimensiunile `rect2` sunt mai mici decât cele ale lui `rect1`, pe când `rect3` este mai lat decât `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Știm că intenționăm să definim o metodă, așadar aceasta se va găsi în blocul `impl Rectangle`. Metoda se va numi `can_hold`, iar ca parametru va prelua un împrumut imutabil către un alt `Rectangle`. Tipul parametrului este deducibil inspectând codul care apelează metoda: `rect1.can_hold(&rect2)` transmite `&rect2`, adică un împrumut imutabil către `rect2`, o instanță a `Rectangle`. Acest lucru este logic, de vreme ce avem nevoie doar să citim `rect2`, nu să-l și scriem (în acest ultim caz, am avea nevoie de un împrumut mutabil). Mai mult, dorim ca `main` să-și păstreze posesiunea asupra `rect2`, pentru a putea folosi din nou instanța după ce apelăm metoda `can_hold`. Valoarea returnată de `can_hold` va fi de tip Boolean, iar implementarea va verifica dacă atât lățimea, cât și înălțimea instanței `self`, sunt mai mari decât cele corespunzătoare altui `Rectangle`. Acum putem adăuga noua metodă `can_hold` în blocul `impl` extras din Lista 5-13, și prezentat în Lista 5-15.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

<span class="caption">Listarea 5-15: Implementăm metoda `can_hold` pentru `Rectangle`, aceasta acceptând o altă instanță `Rectangle` ca argument</span>

Atunci când rulăm acest cod împreună cu funcția `main` prezentată în Listarea 5-14, obținem rezultatul scontat. Metodele pot accepta mai mulți parametri, pe care îi adăugăm în semnătură după parametrul `self`. Acești parametri se comportă exact ca parametrii standard din funcții.

### Funcții asociate

Toate funcțiile care sunt definite în interiorul unui bloc `impl` poartă denumirea de *funcții asociate*, deoarece sunt legate de tipul ce este indicat după `impl`. Avem posibilitatea de a defini funcții asociate care nu au `self` drept prim parametru (acestea, prin urmare, nu sunt metode), întrucât nu necesită o instanță a respectivului tip pentru a putea funcționa. Deja am întâlnit o funcție de acest fel: funcția `String::from`, care este definită pentru tipul `String`.

Funcțiile asociate care nu sunt metode sunt frecvent utilizate în rol de constructori, aceștia având menirea de a returna o nouă instanță a unei structuri. În mod obișnuit, acestea poartă numele de `new`, dar `new` nu este un nume special sau implicit în limbaj. De exemplu, am putea opta să oferim o funcție asociată sub numele de `square` (pătrat), care să aibă un singur parametru ce reprezintă dimensiunea, utilizând-o atât pentru lățime, cât și pentru înălțime, simplificând astfel procesul de creare a unui dreptunghi `Rectangle` pătrat, fără a fi nevoie să introducem aceeași valoare de două ori.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

Cuvintele cheie `Self`, prezente atât în tipul de retur, cât și în corpul funcției, reprezintă alias-uri pentru tipul care urmează după cuvântul cheie `impl`. În acest context, acest tip este `Rectangle`.

Pentru a invoca această funcție asociată, utilizăm sintaxa `::` împreună cu numele structurii. De exemplu, `let sq = Rectangle::square(3);`. Această funcție este asociată unui spațiu de nume definit de structura respectivă: sintaxa `::` este folosită în ambele situații, atât pentru funcțiile asociate, cât și pentru spațiile de nume generate de module. Vom aborda subiectul modulelor în detaliu în [Capitolul 7][modules]<!-- ignore -->.

### Multiple blocuri `impl`

Fiecare structură are permisiunea de a avea mai multe blocuri `impl`. De exemplu, codul din Listarea 5-15 corespunde cu cel prezentat în Listarea 5-16, unde fiecare metodă este separată în propriul său bloc `impl`.


```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```


<span class="caption">Listarea 5-16: Rescrierea Listării 5-15 utilizând mai multe blocuri `impl` </span>

Deși în acest context nu există un motiv anume pentru a separa aceste metode în diverse blocuri `impl`, este important de menționat că aceasta este o sintaxă validă. Vom întâlni un caz de utilizare util pentru multiple blocuri `impl` în Capitolul 10, unde vom discuta despre tipurile generice și trăsături.

## Sumar

Structurile permit crearea tipurilor personalizate relevante pentru domeniul în care lucrezi. Folosind structurile, poți menține conexiuni între diverse date asociate și le poți numi pe fiecare în parte, astfel încât codul tău să fie mai clar. În blocurile `impl`, poți defini funcții care sunt legate de tipul tău și metodele, care sunt un fel de funcții asociate și care îți permit să definești comportamentul pe care instanțele structurilor tale îl au.

Totuși, structurile nu sunt singura metodă de a crea tipuri personalizate: să ne îndreptăm atenția către caracteristica `enum` a limbajului Rust pentru a adăuga un nou instrument în trusa ta de unelte.

[enums]: ch06-00-enums.html
[trait-objects]: ch17-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
