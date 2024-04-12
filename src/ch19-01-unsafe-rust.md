## Rust unsafe

Am discutat până acum despre codul Rust care vine cu garanții de securitate a memoriei impuse în timpul compilării. Cu toate acestea, există în Rust și o altă lume ascunsă, cunoscută sub numele de *Rust unsafe* (Rust nesigur), aceasta funcționează similar cu varianta obișnuită, dar cu avantajul unor capabilități suplimentare.

Motivul pentru care există Rust unsafe este că analiza statică, prin natura sa, tinde să fie conservatoare. Când compilatorul încearcă să verifice conformitatea codului cu anumite garanții, e preferabil să respingă programe valide decât să accepte programe invalide. În cazuri în care codul *ar putea* fi sigur, dar Rust nu poate fi sigur din cauza lipsei de informații, codul va fi respins. În astfel de situații, putem folosi codul unsafe pentru a-i spune compilatorului, „Crede-mă, cunosc consecințele acțiunilor mele”. Cu toate acestea, să fii conștient că utilizarea lui Rust unsafe vine cu propriile riscuri: dacă este folosit greșit, pot apărea probleme legate de siguranța memoriei, precum dereferențierea pointerilor null.

În plus, Rust are un aspect unsafe pentru că hardware-ul calculatorului în sine nu este sigur prin natura sa. Fără posibilitatea de a efectua operații unsafe, anumite programe nu ar putea fi realizate. Rust trebuie să permită programarea la nivel de sistem, ca de exemplu interacțiunea directă cu sistemul de operare sau crearea unui sistem de operare de la zero. Aceste posibilități de programare la nivel scăzut reprezintă unul dintre scopurile principale ale limbajului Rust. Să examinăm ce putem face cu Rust unsafe și cum anume o facem.

### Superputeri unsafe

Pentru a activa modul unsafe Rust, folosește cuvântul cheie `unsafe` și începe un nou bloc care include codul unsafe. În Rust unsafe ai posibilitatea de a efectua cinci acțiuni care nu sunt permise în Rust obișnuit, acțiuni pe care le denumim *superputeri unsafe*. Aceste superputeri sunt:

* Dereferențierea unui pointer raw
* Apelarea unei funcții sau metode unsafe
* Accesarea sau modificarea unei variabile statice mutabile
* Implementarea unei trăsături unsafe
* Accesarea câmpurilor unui `union`

Este esențial să înțelegi că utilizarea `unsafe` nu dezactivează verificatorul de împrumut sau vreo altă verificare de siguranță implementată de Rust: dacă folosești o referință în cadrul codului unsafe, aceasta oricum va fi supusă verificărilor. Cuvântul cheie `unsafe` îți permite doar să accesezi aceste cinci caracteristici care nu sunt verificate de către compilator în ceea ce privește siguranța memoriei, dar totuși vei păstra un anumit nivel de siguranță în interiorul blocurilor unsafe.

Mai mult, `unsafe` nu înseamnă că, în mod necesar, codul din blocul respectiv este periculos sau că va genera probleme de siguranță a memoriei; intenția este că, în calitate de programator, deja *tu* te vei asigura că codul din blocul `unsafe` va accesa memoria într-un mod corect.

Fiindcă omul este supus erorilor și greșelile sunt inevitabile, insistența ca aceste cinci operațiuni unsafe să fie plasate în interiorul blocurilor marcate cu `unsafe` te ajută să conștientizezi că eventualele erori legate de siguranța memoriei le vei găsi anume în cadrul unui bloc `unsafe`. Încearcă să menții blocurile `unsafe` cât mai compacte; îți vei mulțumi ție însuți mai târziu, când vei fi nevoit să investighezi probleme legate de memorie.

Pentru a limita cât mai mult codul unsafe, este indicat să încapsulezi acest tip de cod în cadrul unei abstracțiuni sigure și să oferi o interfață API care să fie de asemenea sigură, aspecte pe care le vom discuta mai târziu în acest capitol când voi aborda funcțiile și metodele unsafe. Unele părți din biblioteca standard sunt implementate ca abstracțiuni sigure peste codul unsafe care a fost verificat. Împachetarea codului unsafe într-o abstracție sigură împiedică propagarea utilizării `unsafe` la toți cei care doresc să folosească funcționalitatea realizată cu ajutorul codului unsafe, deoarece folosirea unei abstracțiuni sigure este, prin definiție, sigură.

Să examinăm pe rând fiecare dintre cele cinci superputeri unsafe. Voi analiza și unele abstracțiuni care oferă o interfață sigură la codul unsafe.

### Dereferențierea Unui Pointer Brut

În Capitolul 4, la secțiunea [“Referințe suspendate”][dangling-references]<!-- ignore
-->, am evidențiat că compilatorul asigură întotdeauna că referințele sunt valide. Unsafe Rust introduce două tipuri noi denumite *pointeri bruți* care sunt asemănători cu referințele. La fel ca referințele, pointerii bruți pot fi imutabili sau mutabili și sunt reprezentați ca `*const T` respectiv `*mut T`. Asteriscul nu este operatorul de dereferențiere; este parte integrantă a denumirii tipului. În contextul pointerilor bruți, *imutabil* semnifică faptul că pointerul nu poate fi atribuit direct după ce a fost dereferențiat.

Diferența față de referințe și pointerii inteligenți, pointerii bruți:

* Au permisiunea de a ignora regulile de împrumut prin existența simultană a pointerilor imutabili și mutabili, sau a multiplelor pointere mutabile către aceeași adresă de memorie
* Nu este garantat că indică spre memorie valida
* Pot fi null
* Nu beneficiază de curățare automată

Renunțând la aceste siguranțe asigurate de Rust, puteți opta pentru performanță mai bună sau capacitatea de a interacționa cu alt limbaj sau hardware unde garanțiile Rust nu se aplică.

Listarea 19-1 demonstrează cum se pot crea un pointer brut imutabil și unul mutabil din referințe.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-01/src/main.rs:here}}
```

<span class="caption">Listarea 19-1: Crearea pointerilor bruți din referințe</span>

Remarcați faptul că nu am folosit cuvântul cheie `unsafe` în acest cod. Este posibil să creăm pointeri bruți în codul sigur; nu putem însă dereferenția pointeri bruți decât într-un bloc `unsafe`, aspect pe care îl veți observa în continuare.

Am generat pointeri bruți utilizând `as` pentru a converti o referință imutabilă și una mutabilă în tipurile lor corespunzătoare de pointeri bruți. Fiind creați direct din referințe care sunt garantate a fi valide, știm că acești pointeri bruți particulari sunt valizi, însă nu putem extinde această premisă asupra oricărui pointer brut.

Pentru a ilustra acest lucru, vom crea un pointer brut al cărui validitate nu poate fi garantată. Listarea 19-2 arată cum să creezi un pointer brut spre o locație arbitrară de memorie. Tentativa de utilizare a memoriei arbitrară este nedefinită: ar putea exista date la acea adresă sau nu, compilatorul ar putea optimiza codul astfel încât să nu existe acces la memorie, sau programul ar putea se încheie cu o eroare de tip segmentation fault. De obicei, nu există motive întemeiate pentru a scrie cod în acest mod, dar este posibil.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-02/src/main.rs:here}}
```

<span class="caption">Listarea 19-2: Crearea unui pointer brut către o adresă de memorie arbitrară</span>

Să ne reamintim că putem crea pointeri bruți în codurile sigure, dar nu îi putem *dereferenția* pentru a citi datele la care acesta indică. În Listarea 19-3, utilizăm operatorul de dereferențiere `*` pe un pointer brut, acțiune care necesită un bloc `unsafe`.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-03/src/main.rs:here}}
```

<span class="caption">Listarea 19-3: Dereferențierea pointerilor bruți într-un bloc `unsafe`</span>

Simpla creare a unui pointer nu produce daune; pericolele apar atunci când încercăm să accesăm valoarea indicată de acesta, situație în care ne-am putea confrunta cu o valoare invalidă.

De asemenea, notăm că în Listarea 19-1 și 19-3 am creat pointeri bruți `*const i32` și `*mut i32`, ambii indicând aceeași locație de memorie unde este stocat `num`. Dacă am încerca în schimb să creăm o referință imutabilă și una mutabilă la `num`, codul nu ar compila, deoarece regulile de posesiune din Rust nu permit existența unei referințe mutabile în același timp cu referințe imutabile. Cu pointerii bruți, avem posibilitatea de a crea un pointer mutabil și unul imutabil spre aceeași locație și de a modifica datele prin intermediul pointerului mutabil, putând astfel crea o posibilă cursă a datelor. Fii prudent!

Cu toate aceste riscuri, de ce am folosi vreodată pointerii bruți? Un caz de folosire principal este atunci când lucrăm cu cod C, cum vom vedea în secțiunea următoare, [„Apelarea unei funcții sau metode unsafe.”](#calling-an-unsafe-function-or-method)<!-- ignore --> Un alt motiv este crearea de abstracțiuni sigure ce nu sunt înțelese de verificatorul de împrumut. Vom aborda funcții nesigure și apoi vom analiza un exemplu de abstracție sigură bazată pe cod nesigur.

### Apelarea unei funcții sau metode unsafe

Cel de-al doilea tip de operațiune ce poate fi efectuată într-un bloc `unsafe` este invocarea funcțiilor unsafe. Aspectul funcțiilor și metodelor unsafe este identic cu al funcțiilor și metodelor regulare, doar că includ cuvântul cheie `unsafe` înainte de definiția propriu-zisă. Prezența `unsafe` semnalează că funcția impune anumite obligații de respectat atunci când o invocăm, deoarece Rust nu poate asigura că am îndeplinit aceste condiții. Invocând o funcție unsafe într-un bloc `unsafe`, transmitem că am citit documentația acesteia și ne asumăm responsabilitatea de a onora obligațiile impuse de funcție.

Iată cum arată o funcție unsafe denumită `dangerous`, care nu execută nicio operațiune în corpul ei:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Trebuie să invocăm funcția `dangerous` în interiorul unui bloc `unsafe` separat. Încercarea de a invoca `dangerous` fără utilizarea unui bloc `unsafe` va rezulta într-o eroare:

```console
{{#include ../listings/ch19-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

Prin folosirea blocului `unsafe`, declarăm că am consultat documentația funcției, că înțelegem modul corect de utilizare și că am verificat îndeplinirea obligațiilor impuse de funcție.

Corpurile funcțiilor unsafe sunt echivalentul unui bloc `unsafe`, așadar, pentru a executa alte acțiuni unsafe în cadrul unei astfel de funcții, nu este nevoie să includem un alt bloc `unsafe`.

#### Crearea unei abstracții sigure peste cod nesigur

Prezența codului nesigur într-o funcție nu impune ca întreaga funcție să fie marcată drept nesigură. De fapt, este o practică obișnuită să încapsulăm codul nesigur într-o funcție sigură. Ca exemplu, să analizăm funcția `split_at_mut` din biblioteca standard, care implică utilizarea unui cod nesigur. Vom explora cum am putea să implementăm aceasta. Această metodă sigură este definită pentru secțiuni mutabile și presupune divizarea unei secțiuni în două la indexul specificat ca argument. Listarea 19-4 ne ilustrează modul în care utilizăm funcția sigură `split_at_mut`.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-04/src/main.rs:here}}
```

<span class="caption">Listarea 19-4: Utilizarea funcției sigure `split_at_mut`</span>

Implementarea acestei funcții exclusiv prin intermediul Rust-ului sigur nu este posibilă. O încercare de implementare ar putea arăta așa cum este în Listarea 19-5, care însă nu va compila. Pentru simplificare, vom implementa `split_at_mut` ca și funcție și nu ca metodă, și doar pentru secțiuni de valori `i32` în loc de un tip generic `T`.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-05/src/main.rs:here}}
```

<span class="caption">Listarea 19-5: O tentativă eșuată de implementare a `split_at_mut` folosind doar Rust sigur</span>

Funcția calculează inițial lungimea totală a secțiunii. Apoi confirmă că indexul oferit ca parametru este în interiorul limitelor secțiunii, verificând dacă acesta este mai mic sau egal cu lungimea. Această verificare va determina ca, în cazul în care transmitem un index care este mai mare decât lungimea la care dorim să divizăm secțiunea, funcția va genera panică înainte de a încerca să utilizeze acel index.

Apoi funcția returnează două secțiuni mutabile ca elemente ale unei tuple: prima de la începutul secțiunii originale până la indexul `mid` și cel de-al doilea de la `mid` până la sfârșitul secțiunii.

Încercând să compilăm codul prezentat în Listarea 19-5, vom întâmpina o eroare.

```console
{{#include ../listings/ch19-advanced-features/listing-19-05/output.txt}}
```

Verificatorul de împrumut din Rust nu poate înțelege că împrumutăm părți diferite ale secțiunii; el recunoaște doar că se face împrumutul din aceeași secțiune de două ori. De fapt, împrumutarea diferitelor segmente ale unei secțiuni este acceptabilă din moment ce cele două secțiuni nu se suprapun, însă Rust nu este destul de evoluat pentru a detecta acest lucru. În momentul în care știm că un cod este corect, dar Rust nu, este vremea să apelăm la utilizarea codului `unsafe`.

Listarea 19-6 prezintă modul în care putem folosi un bloc `unsafe`, un pointer brut și câteva apeluri la funcții nesigure pentru a implementa funcția `split_at_mut`.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-06/src/main.rs:here}}
```

<span class="caption">Listarea 19-6: Utilizarea codului nesigur în cadrul implementării funcției `split_at_mut`</span>

Reamintim din Capitolul 4 [„Tipul secțiune”][the-slice-type]<!-- ignore -->  că secțiunile reprezintă un pointer către anumite date și lungimea respectivei secțiuni. Apelăm metoda `len` pentru a afla lungimea unei secțiuni și metoda `as_mut_ptr` pentru a accesa pointerul brut al secțiunii. În cazul nostru, având o secțiune mutabilă de valori `i32`, `as_mut_ptr` ne asigură un pointer brut cu tipul `*mut i32`, pe care l-am salvat în variabila `ptr`.

Păstrăm afirmația că indexul `mid` se găsește în interiorul secțiunii. Apoi trecem la codul nesigur: funcția `slice::from_raw_parts_mut` primește un pointer brut și o lungime pentru a genera o nouă secțiune. Această funcție este folosită pentru a crea o secțiune începând de la `ptr` și care se întinde pe `mid` elemente. Ulterior invocăm metoda `add` pe `ptr` cu argumentul `mid` pentru a câștiga un pointer brut ce debutează de la `mid`, realizând o nouă secțiune utilizând acel pointer și numărul de elemente rămase după `mid` drept lungime.

Funcția `slice::from_raw_parts_mut` este considerată nesigură deoarece presupune utilizarea unui pointer brut și necesită încrederea că acest pointer este valid. De asemenea, metoda `add` aplicată pointerilor bruți este nesigură, având în vedere că presupune ca și locația de decalare să fie un pointer valid. Așadar, am înconjurat folosirea `slice::from_raw_parts_mut` și `add` cu un bloc `unsafe`, ceea ce ne permite apelarea lor. Analizând codul și asigurându-ne prin cerința că `mid` este mai mic sau egal cu `len`, putem concluziona că toți pointerii bruți folosiți în blocul `unsafe` sunt pointeri valizi către date din secțiune. Aceasta constituie o aplicare corespunzătoare a conceptului `unsafe`.

Este important de notat că funcția `split_at_mut` rezultată nu necesită marcarea ca `unsafe`, iar apelarea acesteia se poate face din Rust sigur. Prin urmare, am elaborat o abstractizare sigură pentru codul `unsafe` prin implementarea funcției ce utilizează cod nesigur într-o manieră sigură, formând astfel doar pointeri validați din datele disponibile funcției.

În contrast, folosirea funcției `slice::from_raw_parts_mut` prezentată în Listarea 19-7 ar putea duce la prăbușirea aplicației atunci când secțiunea este accesată. Codul specificat generează o secțiune de 10.000 de elemente începând de la o locație arbitrar aleasă în memorie.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-07/src/main.rs:here}}
```

<span class="caption">Listarea 19-7: Generarea unei secțiuni de la o locație arbitrară în memorie</span>

Nu deținem control asupra memoriei din această locație aleatoare, și nu există niciun fel de garanție că secțiunea generată conține valori `i32` valide. Încercarea de a utiliza `values` de parcă ar fi o secțiune adevărată duce la comportament nedefinit.

#### Folosirea funcțiilor `extern` pentru apelarea codului extern

Câteodată, codul Rust pe care îl scriem ar putea avea nevoie să interacționeze cu cod scris într-un alt limbaj. În acest scop, Rust oferă cuvântul cheie `extern`, care facilitează crearea și utilizarea unei *Interfețe Funcționale Externe* (Foreign Function Interface, FFI). Un FFI constituie o modalitate prin care un limbaj de programare poate defini funcții, permițând ca un alt limbaj de programare (străin) să cheme acele funcții.

Listarea 19-8 ilustrează cum se realizează o integrare cu funcția `abs` din biblioteca standard a limbajului C. Funcțiile declarate în interiorul blocurilor `extern` sunt întotdeauna considerate nesigure când sunt chemate din codul Rust. Acest lucru se datorează faptului că alte limbaje nu impun regulile și garanțiile specifice lui Rust și, deoarece Rust nu le poate verifica, responsabilitatea asigurării siguranței revine programatorului.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-08/src/main.rs}}
```

<span class="caption">Listarea 19-8: Declararea și apelarea unei funcții `extern` definită într-un alt limbaj</span>

În cadrul blocului `extern "C"`, enumerăm numele și semnăturile funcțiilor externe din alt limbaj pe care dorim să le folosim. Secțiunea `"C"` specifică ce *interfață binară a aplicației* (application binary interface, ABI) este folosită de funcția respectivă: ABI definește cum se realizează apelul funcției la nivelul limbajului de asamblare. ABI-ul de tip `"C"` este cel mai frecvent utilizat și este în concordanță cu ABI-ul limbajului de programare C.

> #### Invocarea funcțiilor Rust din alte limbaje
>
> Putem de asemenea să utilizăm `extern` pentru a crea o interfață care permite
> altor limbaje să invoce funcții Rust. În loc să generăm un bloc `extern`
> complet, adăugăm cuvântul cheie `extern` și specificăm ABI-ul de utilizat
> exact înaintea cuvântului cheie `fn` pentru funcția în cauză. Trebuie să
> includem și adnotarea `#[no_mangle]` pentru a împiedica compilatorul Rust să
> modifice numele acestei funcții. *Mangling* înseamnă modificarea de către un
> compilator a numelui dat de noi unei funcții într-un nume diferit care
> include mai multe informații pentru diversele părți ale procesului de
> compilare, dar care este mai puțin lizibil pentru oameni. Deoarece fiecare
> compilator al unui limbaj de programare mâzgălește numele într-o manieră ușor
> diferită, pentru ca o funcție Rust să fie identificabilă de alte limbaje,
> trebuie să dezactivăm această mâzgăleală a numelui efectuată de compilatorul
> Rust.
>
> În exemplul de mai jos, facem funcția `call_from_c` accesibilă din cod C,
> după ce aceasta a fost compilată într-o bibliotecă dinamică și link-uită din
> C:
>
> ```rust
> #[no_mangle]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ```
>
> Utilizarea `extern` în acest context nu implică utilizarea `unsafe`.

### Accesarea sau modificarea unei variabile statice mutabile

Nu am discutat încă despre *variabile globale* în această carte, care sunt suportate de Rust, dar pot contraveni regulilor de posesiune Rust atunci când sunt modificate. Acest lucru poate duce la curse de date dacă două fire de execuție accesează simultan aceeași variabilă globală mutabilă.

În Rust, variabilele globale se numesc *variabile statice*. Listarea 19-9 ilustrează cum să declari și să utilizezi o variabilă statică cu o valoare de tip secțiune de string.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
...
```

<span class="caption">Listarea 19-9: Definirea și utilizarea unei variabile statice imutabile</span>

Variabilele statice sunt similare constantelor, discutate în secțiunea [“Diferențele între variabile și constante”][differences-between-variables-and-constants]<!-- ignore --> din Capitolul 3. Numele variabilelor statice sunt, prin convenție, în `SCREAMING_SNAKE_CASE`. Variabilele statice pot deține doar referințe cu durata de viață `'static`, ceea ce înseamnă că compilatorul Rust poate rezolva durata de viață fără să fie nevoie de adnotarea explicită a acesteia. Accesul la o variabilă statică imutabilă este sigur.

O diferență subtilă între constante și variabilele statice imutabile este că valorile unei variabile statice sunt alocate la o adresă fixă în memorie. Fiecare utilizare a valorii va accesa mereu datele din aceeași locație. Pe de altă parte, constantele pot duplica valorile ori de câte ori sunt utilizate. Variabilele statice pot fi, de asemenea, mutabile, dar accesarea și modificarea lor este considerată *nesigură*. Listarea 19-10 arată cum să declari, să accesezi și să modifici o variabilă statică mutabilă numită `COUNTER`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-10/src/main.rs}}
```

<span class="caption">Listarea 19-10: Citirea sau scrierea într-o variabilă statică mutabilă este nesigură</span>

Similar cu variabilele obișnuite, folosim cuvântul cheie `mut` pentru a indica mutabilitatea. Orice cod care interacționează cu `COUNTER` trebuie să fie înconjurat de un bloc `unsafe`. Acest cod se compilează și va afișa `COUNTER: 3` cum ne-am aștepta, întrucât rulează pe un singur fir de execuție. Folosirea `COUNTER` în scenarii cu mai multe fire de execuție ar putea cauza cu ușurință curse de date.

Este dificil să garantăm evitarea curselor de date când lucrăm cu date mutable accesibile global, ceea ce face Rust să considere variabilele statice mutabile ca fiind nesigure. Acolo unde e posibil, este de dorit să utilizăm tehnicile de concurență și pointerii inteligenți thread-safe descrise în Capitolul 16, astfel încât compilatorul să confirme că accesul la date din diferite fire de execuție este realizat în condiții de siguranță.

### Implementarea unei trăsături nesigure

Este posibil să utilizăm `unsafe` pentru a implementa o trăsătură nesigură. O trăsătură este considerată nesigură când are cel puțin una dintre metodele sale cu anumite invariante ce nu pot fi verificate de către compilator. O trăsătură se declară `unsafe` adăugând cuvântul cheie `unsafe` înainte de `trait` și marcând totodată implementarea trăsăturii ca `unsafe`, așa cum se arată în Listarea 19-11.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-11/src/main.rs}}
```

<span class="caption">Listarea 19-11: Definirea și implementarea unei trăsături nesigure</span>

Prin utilizarea `unsafe impl`, ne asumăm responsabilitatea de a respecta invariantele pe care compilatorul nu le poate confirma.

De exemplu, să ne aducem aminte de trăsăturile de marcaj `Sync` și `Send` discutate în secțiunea [“Concurență extensibilă cu trăsăturile `Sync` și `Send`”][extensible-concurrency-with-the-sync-and-send-traits]<!-- ignore --> din Capitolul 16: compilatorul implementează automat aceste trăsături dacă tipurile noastre sunt alcătuite exclusiv din tipuri `Send` și `Sync`. În cazul în care creăm un tip ce include un tip care nu este `Send` sau `Sync`, cum ar fi pointerii bruți, și dorim să considerăm acel tip ca fiind `Send` sau `Sync`, trebuie să recurgem la `unsafe`. Rust nu poate asigura că tipul nostru respectă garanțiile de a fi transmis în siguranță între fire de execuție sau de a fi accesat concomitent de mai multe fire de execuție; prin urmare, este necesar să verificăm manual aceste garanții și să indicăm aceasta prin utilizarea `unsafe`.

### Accesarea câmpurilor unei uniuni

Ultima operațiune care necesită folosirea `unsafe` este accesarea câmpurilor unei *uniuni*. Un `union` este similar cu un `struct`, dar doar un singur câmp declarat este utilizat într-o anumită instanță, la un moment dat. Uniunile sunt folosite în principal pentru interoperarea cu uniunile din codul C. Accesul la câmpurile unei uniuni este nesigur pentru că Rust nu poate asigura tipul datelor care sunt stocate curent în instanța uniunii. Poți învăța mai multe despre uniuni în [Referința Rust][reference].

### Când utilizăm cod nesigur

Utilizarea `unsafe` pentru a lua una dintre cele cinci acțiuni (superputeri) menționate mai sus nu este greșită sau văzută cu reticență. Totuși, este mai dificil să scrii cod `unsafe` corect, deoarece compilatorul nu poate contribui la asigurarea securității memoriei. Dacă ai un motiv întemeiat să folosești cod `unsafe`, ai posibilitatea de a face asta, iar utilizarea explicită a adnotării `unsafe` ușurează identificarea sursei problemelor atunci când acestea apar.

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references [differences-between-variables-and-constants]: ch03-01-variables-and-mutability.html#constants [extensible-concurrency-with-the-sync-and-send-traits]: ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits [the-slice-type]: ch04-03-slices.html#the-slice-type [reference]: ../reference/items/unions.html
