## Validarea referințelor cu ajutorul lifetimes

Lifetimes sunt un alt fel de generici pe care i-am utilizat deja. Aceștia nu doar că asigură faptul că un tip are comportamentul pe care îl dorim, ci și că referințele rămân valide pentru durata necesară.

Un aspect pe care nu l-am discutat în secțiunea [„Referințe și împrumutare”][references-and-borrowing]<!-- ignore --> din Capitolul 4 este acela că fiecare referință în Rust deține un *lifetime*, adică un domeniu de vizibilitate pentru care referința este validă. În cele mai multe situații, lifetimes sunt impliciți și inferați, așa cum sunt și tipurile, în mod obișnuit. Adnotarea tipurilor este necesară doar atunci când mai multe tipuri sunt posibile. Analog, adnotarea lifetimes este necesară atunci când duratele de viață ale referințelor pot fi interpretate diferit. Rust ne cere să definim aceste relații utilizând parametri de lifetime generici pentru a garanta că referințele utilizate în timpul execuției vor fi valide în mod cert.

Conceptul de adnotare a lifetimes nu există în multe alte limbaje de programare, ceea ce îl face să ni se pară neobișnuit. Deși nu vom trata lifetimes în totalitatea lor în acest capitol, vom explora modalitățile comune prin care este posibil să întâlnim sintaxa specifică lifetimes astfel încât să ne obișnuim cu conceptul.

### Combaterea referințelor suspendate prin intermediul duratelor de viață

Scopul esențial al duratelor de viață (lifetimes) este de a înlătura referințele suspendate (dangling references), care determină programul să referențieze alte date decât cele destinate. Analizează programul din Listarea 10-16, ce conține un domeniu de vizibilitate extern și unul intern.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

<span class="caption">Listarea 10-16: Tentativa de a folosi o referință a cărei valoare nu mai este în domeniul de vizibilitate</span>

> Notă: În Listările 10-16, 10-17 și 10-23 sunt declarate variabile fără
> valori inițiale, astfel numele lor sunt prezente în domeniul extern. La
> prima vedere, acest lucru poate părea că intră în conflict cu faptul că Rust
> nu permite valori null. Cu toate acestea, dacă încercăm să utilizăm o
> variabilă înainte să îi atribuim o valoare, vom întâlni o eroare la
> compilare, confirmând astfel că Rust nu acceptă valori null.

În domeniul extern este declarată o variabilă `r` fără valoare inițială, iar în domeniul intern este declarată o variabilă `x` cu valoarea inițială de 5. În domeniul intern, încercăm să stabilim `r` ca referință la `x`. La încheierea domeniului intern, încercăm să afișăm valoarea referită de `r`. Acest cod nu va compila deoarece valoarea la care `r` face referire a ieșit din domeniul de vizibilitate înainte de a fi folosită. Iată mesajul de eroare:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

Variabila `x` nu "durează" suficient de mult. Aceasta iese din domeniul de vizibilitate când domeniul intern se sfârșește la linia 7. Cu toate acestea, `r` rămâne în domeniul extern; deoarece domeniul său de vizibilitate este mai amplu, spunem că "are o durată de viață mai mare". Dacă Rust ar permite ca acest cod să funcționeze, `r` ar face referință la memorie care a fost dezalocată când `x` a ieșit din domeniu, și orice tentativă de interacțiune cu `r` nu ar avea rezultate corecte. Cum determină Rust că acest cod este nevalid? Prin utilizarea unui verificator de împrumut (borrow checker).

### Verificatorul de împrumut

Compilatorul Rust beneficiază de un *verificator de împrumut* care evaluează domeniile de vizibilitate și stabilește dacă toate împrumuturile sunt conforme. Listarea 10-17 îți prezintă același cod ca Listarea 10-16, dar cu adnotări ce indică durata de viață a variabilelor.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

<span class="caption">Listarea 10-17: Adnotările duratelor de viață ale `r` și `x`, denumite `'a` și `'b`, în ordine</span>

În exemplul de față, am însemnat durata de viață a `r` cu `'a` și pe cea a lui `x` cu `'b`. Remarcăm că blocul intern `'b` este considerabil mai restrâns decât blocul extern `'a`. În etapa de compilare, Rust analizează și compară extinderea celor două durate și identifică faptul că `r` există pentru `'a`, dar face referire la memorie ale cărei valori au durata `'b`. Programul este respins pentru că durata `'b` este inferioară lui `'a`: entitatea la care face referire nu persistă atât cât durează referința.

Listarea 10-18 rezolvă problema referinței suspendate și se compilează fără niciun fel de eroare.

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

<span class="caption">Listarea 10-18: O referință corectă, având în vedere că informațiile referite au o durată de viață mai lungă decât referința însăși</span>

În această situație, `x` deține durata de viață `'b`, care de această dată excede `'a`. Astfel, `r` are libertatea de a referi `x`, deoarece Rust confirmă că referința din `r` va fi întotdeauna în vigoare pe durata existenței lui `x`.

Acum, cunoscând localizarea duratelor de viață ale referințelor și metodologia prin care Rust le evaluează pentru a asigura validitatea neîntreruptă a acestora, să ne orientăm spre examinarea duratelor de viață generice pentru parametri și valori returnate în contextul funcțiilor.

### Durate de viață generice în funcții

Să dezvoltăm o funcție care identifică care dintre două secțiuni de string-uri este mai lungă. Această funcție va accepta două secțiuni și va oferi ca rezultat o singură secțiune de string. Implementarea corespunzătoare pentru funcția `longest` va face ca exemplul prezentat în Listarea 10-19 să genereze output-ul `The longest string is abcd`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

<span class="caption">Listarea 10-19: Funcția `main` apelează `longest` pentru a afla cea mai lungă secțiune dintre două secțiuni de string-uri</span>

Trebuie subliniat faptul că funcția trebuie să opereze cu secțiuni de string-uri, adică referințe, nu cu string-uri, pentru că nu ne dorim ca `longest` să preia posesiunea asupra parametrilor săi. Consultă secțiunea [“Secțiuni de string-uri ca
parametri”][string-slices-as-parameters]<!-- ignore --> din Capitolul 4 pentru mai multe explicații privind alegerea acestor parametri în Listarea 10-19.

Dacă vom încerca să implementăm `longest` așa cum e exemplificat în Listarea 10-20, vom observa că nu va compila.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

<span class="caption">Listarea 10-20: O încercare de implementare a `longest`, care urmează să returneze secțiunea de string mai lungă, dar care în prezent nu compilează</span>

Eroarea întâmpinată se referă la duratele de viață:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

Indicațiile de ajutor arată că tipul care se returnează necesită un parametru de durată de viață generic, pentru că Rust nu este capabil să determine dacă referința ce se returnează aparține lui `x` sau `y`. Și noi suntem în aceeași incertitudine, de vreme ce funcția poate returna fie o referință către `x`, fie una către `y`, în funcție de rezultatul condiției `if`.

Nu avem informațiile despre valorile exacte care vor fi introduse în funcție la definirea acesteia, deci nu putem prevedea care dintre cazurile `if` sau `else` se va întâmpla. Nu cunoaștem nici care vor fi duratele de viață concrete ale referințelor ce vor fi folosite, așa că nu putem estima domeniile lor de vizibilitate pentru a decide dacă referința returnată va fi mereu validă. Verificatorul de împrumut al lui Rust nu este capabil să facă aceste deducții singur; nu are cunoștințe despre cum se corelează duratele de viață ale lui `x` și `y` cu durata de viață a valorii ce urmează să fie returnată. Pentru a rectifica eroarea, trebuie să introducem parametri generici de durată de viață, care vor clarifica relația dintre referințele implicate, permițând astfel verificatorului de împrumut să realizeze analiza necesară.

### Sintaxa adnotărilor pentru duratele de Viață

Adnotările pentru duratele de viață nu influențează cât timp supraviețuiesc referințele. În realitate, ele stabilesc relații între duratele de viață ale mai multor referințe, neavând impact asupra acestora. Asemănător cu funcțiile ce acceptă orice tip când sunt definite cu parametri generici de tip, funcțiile pot accepta referințe cu orice durată de viață specificând un parametru generic de durată de viață.

Adnotările pentru duratele de viață utilizează o sintaxă ceva mai neobișnuită: numele acestor parametri încep cu apostrof (`'`) și sunt de regulă foarte scurte și scrise cu litere mici, în manieră similară tipurilor generice. Denumirea `'a` este adesea prima opțiune pentru o adnotare de durată de viață. Aceste adnotări se plasează după simbolul `&` al unei referințe, cu un spațiu între adnotarea și tipul referinței.

Iată nişte exemple: o referință către un `i32` fără un parametru de durată de viață, o referință către un `i32` cu un parametru de durată de viață `'a`, și o referință mutabilă către un `i32` care are de asemenea durata de viață `'a`.

```rust,ignore
&i32        // o referință
&'a i32     // o referință cu durată de viață explicită
&'a mut i32 // o referință mutabilă cu durată de viață explicită
```

O singură adnotare de durată de viață, luată individual, nu aduce multă claritate, deoarece aceste adnotări sunt proiectate să descrie pentru Rust cum relaționează între ele parametrii de durată de viață generici ai diferitelor referințe. Să analizăm cum aceste adnotări pentru duratele de viață interacționează reciproc în contextul funcției `longest`.

### Adnotarea duratelor de viață în semnăturile funcțiilor

Pentru a aplica adnotări ale duratelor de viață în semnăturile funcțiilor, este necesar să declarăm parametrii de *durată de viață* generici, plasându-i între parantezele unghiulare care se află între numele funcției și lista de parametri, procedând astfel ca în cazul parametrilor de *tip* generici.

Semnătura trebuie să exprime următoarea restricție: referința returnată rămâne validă pe durata de viață a ambilor parametri. Aceasta descrie relația dintre duratele de viață ale parametrilor și valoarea returnată. Durata de viață `'a` este numele pe care îl vom da acestei relații, pe care o vom adnota pe fiecare referință, așa cum este ilustrat în Listarea 10-21.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

<span class="caption">Listarea 10-21: Definirea funcției `longest`, indicând că toate referințele din semnătură trebuie să aibă durata de viață `'a`</span>

Acest cod ar trebui să compileze și să genereze rezultatul dorit atunci când este folosit odată cu 
funcția `main` din Listarea 10-19.

Semnătura funcției ne anunță că pentru o anumită durată de viață `'a`, aceasta primește doi parametri, ambii fiind secțiuni de string-uri care persistă minim pe durata de viață `'a`. De asemenea, semnătura precizează că secțiunea de string returnată de funcție va persista cel puțin pe durata de viață `'a`. În practică, acest lucru sugerează că durata de viață a referinței întoarse de funcția`longest` corespunde cu cea mai scurtă durată de viață a valorilor referite de argumentele funcției. Aceste legături sunt relațiile pe care dorim ca Rust să le folosească în evaluarea codului nostru.

Trebuie reținut că atunci când definim parametri de *durată de viață* în această semnătură, nu modificăm duratele de viață ale valorilor care sunt transmise sau întoarse de funcție. În schimb, stabilim condițiile pe care verificatorul de împrumut trebuie să le aplice, respingând valorile care nu se încadrează în aceste limite. Cu alte cuvinte, funcția `longest` nu trebuie să cunoască exact cât timp `x` și `y` vor exista, ci trebuie să fie garantat că va exista o durată de viață care poate fi atribuită lui `'a` și care va îndeplini cerințele acestei semnături.

Când vine vorba de adnotarea duratelor de viață în funcții, acestea sunt specificate în semnătura funcției, nu în corpul acesteia. Astfel adnotările devin parte integrantă a contractului funcției, pe picior de egalitate cu tipurile din semnătură. Includerea acestui contract de *durată de viață* în semnătura funcției simplifică analiza realizată de compilatorul Rust. Dacă întâmpinăm probleme legate de adnotările unei funcții sau de modul în care este invocată, erorile generate de compilator pot indica precis unde în codul nostru se află problemele și care sunt constrângerile nerespectate. Dacă ar exista o dependență mai mare pe inferențele Rust în ceea ce privește intențiile noastre legate de relațiile duratelor de viață, atunci compilatorul ar putea indica problemele abia după urmărirea utilizării codului la câteva niveluri de la cauza de bază.

Când folosim referințe specifice cu funcția `longest`, durata de viață concretă substituită pentru `'a` este acea parte din durata de viață a lui `x` care coincide cu durata de viață a lui `y`. Mai direct spus, durata de viață generică `'a` se va concretiza în cea mai scurtă durată de viață dintre cele ale lui `x` și `y`. Dat fiind că am adnotat referința returnată cu același parametru de *durată de viață* `'a`, și referința returnată va fi validă pentru aceeași perioadă de timp cât sunt valide `x` și `y`.

Evaluăm acum modul în care adnotările de durată de viață limitează funcția `longest` printr-o demonstrație unde sunt folosite referințe cu durate de viață concrete diferite. Listarea 10-22 ne furnizează un astfel de exemplu explicit.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

<span class="caption">Listarea 10-22: Utilizarea funcției `longest` cu referințe la valori `String` ce au durate de viață concrete diferite</span>

În acest caz, `string1` rămâne valid până la încheierea domeniului de vizibilitate exterior, `string2` este valid până la încheierea domeniului de vizibilitate interior, iar `result` indică o valoare care rămâne validă până la finalul domeniului de vizibilitate interior. Dacă executăm acest cod, vom constata că este acreditat de verificatorul de împrumut; va compila și va crea afișajul „The longest string is long string is long”.
```

Continuând, să analizăm un exemplu care ilustrează necesitatea ca durata de viață a referinței în `result` să fie mai scurtă decât duratele de viață ale celor două argumente. Declararea variabilei `result` va fi efectuată în afara domeniului de vizibilitate intern, în timp ce asignarea valorii pentru aceasta va rămâne în interiorul acelui domeniu împreună cu `string2`. Vom muta instrucțiunea `println!`, ce utilizează `result`, în afara domeniului intern, după ce acesta se încheie. Codul prezentat în Listarea 10-23 nu va fi compilabil.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

<span class="caption">Listarea 10-23: Tentativa de utilizare a `result` după ce `string2` nu se mai află în domeniul de vizibilitate</span>

La încercarea de a compila acest cod, întâmpinăm eroarea:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

Eroarea ne arată că, pentru a fi valid pentru instrucțiunea `println!`, `string2` ar necesita să fie disponibil până la finalul domeniului extern. Rust înțelege acest aspect deoarece am adnotat duratele de viață ale parametrilor funcției și ale valorii de retur utilizând același parametru `'a`.

Noi, oamenii, putem privi acest cod și înțelege că `string1` este disponibil mai mult timp decât `string2` și, prin urmare, `result` va conține o referință către `string1`, care, nefiind încă ieșită din domeniul de vizibilitate, rămâne validă pentru instrucțiunea `println!`. Cu toate acestea, compilatorul nu este capabil să deducă că referința este validă în acest caz. I-am indicat compilatorului Rust că durata de viață a referinței returnate de funcția `longest` coincide cu durata cea mai scurtă a referințelor primite. În consecință, verificatorul de împrumut respinge codul din Listarea 10-23 pentru că s-ar putea să aibă o referință invalidă.

Propune-ți să experimentezi cu diverse scenarii care alterează valorile și duratele de viață ale referințelor transmise funcției `longest`, precum și modul în care este utilizată referința retur. Înainte de compilare, formulează presupuneri referitoare la rezultatele verificatorului de împrumut și verifică ulterior dacă acestea sunt corecte!

### Gândirea în termeni de durate de viață

Alegerea parametrilor de durată de viață este strâns legată de lucrul efectuat de funcția în cauză. Dacă, spre exemplu, am decide ca funcția `longest` să returneze constant primul argument în locul celei mai lungi secțiuni de string, indicația de durată de viață pentru parametrul `y` nu ar fi necesară. Așadar, următorul cod va fi considerat valid de către compilator:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

Prin introducerea parametrului de durată de viață `'a` pentru argumentul `x` și pentru tipul de retur, dar omițându-l pe `y`, recunoaștem că durata de viață a lui `y` nu are conexiuni cu durata de viață a lui `x` ori cu valoarea returnată.

O funcție care returnează o referință trebuie să sincronizeze durata de viață a tipului de retur cu durata de viață a unuia dintre argumente. Dacă referința returnată *nu* corespunde niciunui argument, atunci ea trebuie să indică spre o valoare creată intern în funcție, ceea ce inevitabil va crea o referință suspendată, dat fiind că respectivele valori vor părăsi domeniul de vizibilitate când funcția se încheie. Să luăm spre analiză un caz de implementare eșuată a funcției `longest`, care va fi respins de compilator:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

Deși am definit parametrul de durată de viață `'a` pentru tipul returnat, codul nu trece de etapa de compilare, pentru că durata de viață a valorii returnate nu are nicio legătură cu duratele de viață ale argumentelor. Acesta este mesajul de eroare afișat:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

Complicația de aici provine din faptul că `result` nu mai există după închiderea funcției `longest`, încercând totodată să returneze o referință către aceasta. Nu putem remedia prin parametrii de durată de viață acest tip de referință suspendată, iar in sistemul Rust, acestea sunt inacceptabile. În situații similare, soluția cea mai potrivită ar fi să optăm pentru returnarea unei date cu posesiune completă, nu sub formă de referință, astfel încât funcția apelatoare să preia responsabilitatea gestiunii acesteia.

Concluzionând, aplicarea corectă a sintaxei duratelor de viață leagă duratele de viață ale diverselor argumente de cele ale valorilor returnate, permițându-i astfel limbajului Rust să asigure operațiuni de gestionare a memoriei în mod sigur și să interzică orice operațiune ce ar putea duce la apariția referințelor suspendate sau care ar putea pune în pericol siguranța manipulării memoriei.

### Adnotări de durată de viață în definițiile de structuri

Până acum, structurile pe care le-am descris includ tipuri de date cu posesiune proprie. Avem posibilitatea să definim structuri ce conțin referințe, dar pentru aceasta este necesar să adăugăm adnotări de durată de viață pentru toate referințele din definiția structurii. În Listarea 10-24, este prezentată structura `ImportantExcerpt`, care încorporează o secțiune de tip string.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

<span class="caption">Listarea 10-24: Structura care include o referință necesitând adnotare de durată de viață</span>

Structura posedă un unic câmp `part`, ce conține o secțiune dintr-un string, iar aceasta este o referință. În maniera tipurilor generice, numele parametrului generic al duratei de viață se declară în paranteze unghiulare după numele structurii, ceea ce ne permite să folosim parametrul în cadrul definiției structurii. Adnotarea indică faptul că o instanță de `ImportantExcerpt` nu are voie să depășească durata de viață a referinței din câmpul `part`.

Funcția `main` generează o instanță a structurii `ImportantExcerpt`, care înglobează o referință la prima sentință din `String` aparținând variabilei `novel`. Informația conținută în `novel` este disponibilă cu mult înainte de crearea instanței `ImportantExcerpt`. În plus, `novel` nu devine inaccesibil până după ce structura `ImportantExcerpt` este retrasă din domeniul de vizibilitate, făcând astfel ca referința din instanța `ImportantExcerpt` să fie validă.

### Eliziunea duratei de viață

Am învățat că fiecare referință are o durată de viață şi trebuie să specificăm
parametrii de durată de viață pentru funcții sau structuri care folosesc referințe. Totuși, în
Capitolul 4, am prezentat o funcție în Listarea 4-9, reafirmată în Listarea 10-25, ce
s-a compilat fără adnotări de durată de viață.

<span class="filename">Numele fişierului: src/lib.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

<span class="caption">Listarea 10-25: O funcție definită în Listarea 4-9 care
s-a compilat fără adnotările de durată de viață, deși parametrul și tipul de retur sunt referințe</span>

Funcția se compilează fără adnotări de durată de viață din motive istorice: în versiunile timpurii (pre-1.0) ale Rust, codul respectiv nu ar fi funcționat, fiind necesară o durată de viață explicită pentru fiecare referință. Semnătura funcției de atunci ar fi arătat astfel:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

După redactarea unei cantități considerabile de cod Rust, echipa a identificat situații specifice unde programatorii Rust repetă adnotări similare de durată de viață. Având un șablon previzibil și determinist, dezvoltatorii au înscris aceste modele în comportamentul compilatorului, astfel încât verificatorul de împrumut să poată înțelege duratele de viață implicit, eliminând necesitatea adnotărilor explicite.

Această parte din istoria Rust este relevantă, deoarece este posibil ca în viitor, pe măsură ce sunt identificate alte șabloane deterministe, să avem nevoie de și mai puține adnotări de durată de viață.

Regulile care guvernează această analiză a referințelor în Rust se numesc *regulile de eliziune a duratei de viață*. Acestea nu sunt directive pentru programatori, ci cazuri specifice pe care compilatorul le recunoaște, iar în prezența acestora, nu este nevoie să specifice durate de viață în cod.

Eliziunea nu implică inferență totală. Dacă, după aplicarea regulilor în mod deterministic, rămân ambiguități cu privire la duratele de viață ale anumitor referințe, compilatorul nu va specula asupra lor. În schimb, va emite o eroare ce poate fi rezolvată prin adăugarea explicită a adnotărilor necesare.

Duratele de viață asociate parametrilor de funcție sau metodei sunt cunoscute drept *durate de viață de intrare*, iar cele legate de valori returnate sunt *durate de viață de ieșire*.

Pentru deducerea duratelor de viață ale referințelor lipsite de adnotări explicite, compilatorul urmează trei reguli. Prima se aplică la duratele de viață de intrare, iar următoarele două la duratele de viață de ieșire. Dacă, după aceste trei reguli, există referințe având în continuare o durată de viață nedeterminată, compilatorul va întrerupe procesul și va raporta o eroare. Regulile sunt aplicabile atât definițiilor de `fn`, cât și blocurilor `impl`.

Prima regulă pe care compilatorul o impune este aceea că atribuie un parametru de durată de viață pentru fiecare parametru care este o referință. Concret, o funcție cu un singur parametru va avea un parametru de durată de viață: `fn foo<'a>(x: &'a i32)`. În cazul unei funcții cu doi parametri, vor exista doi parametri de durată de viață separați: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)` și așa mai departe.

Conform celei de-a doua reguli, dacă avem un singur parametru de intrare cu durată de viață, atunci aceasta se aplică întregii ieșiri: `fn foo<'a>(x: &'a i32) -> &'a i32`.

Când intervin mai mulți parametri de intrare cu durate de viață și unul dintre aceștia este `&self` sau `&mut self` – adică în contextul unei metode – durata de viață a `self` se va atribui la întreaga ieșire. Acest lucru simplifică scrierea și citirea metodelor, deoarece nu este nevoie de atât de multe simboluri.

Imaginându-ne în rolul compilatorului, aplicăm aceste reguli pentru a înțelege duratele de viață ale referințelor din semnătura funcției `first_word`, așa cum e prezentată în Listarea 10-25. Semnătura inițială nu conține nicio durată de viață asociată referințelor:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Aplicând prima regulă, compilatorul acordă fiecărui parametru propria durată de viață, etichetată în mod tradițional cu `'a`:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

Datorită prezenței unui unic parametru de intrare cu durată de viață, a doua regulă intervine pentru a atribui aceeași durată de viață și la ieșire, rezultând în următoarea semnătură:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Astfel, toate referințele din această semnătură de funcție sunt acum prevăzute cu durate de viață, dând voie compilatorului să progreseze în analiză fără ca programatorul să fie nevoit să adnoteze manual aceste durate.

Luăm ca exemplu funcția `longest`, care la început nu includea parametri de durată de viață, așa cum vedem în Listarea 10-20:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Aplicăm prima regulă și observăm că fiecare dintre cei doi parametri are asignată o durată de viață distinctă:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

În acest caz, nici a doua nici a treia regulă nu sunt aplicabile – a doua pentru că avem multiple durate de viață de intrare și a treia pentru că nu avem de-a face cu o metodă care implică `self`. Prin urmare, după examinarea celor trei reguli, încă nu se poate determina durata de viață a returnării funcției — motiv pentru care întâmpinăm o eroare la compilarea codului din Listarea 10-20: regulile de eliziune nu oferă suficiente informații compilatorului pentru a stabili duratele de viață ale referințelor din semnătură.

Deoarece a treia regulă este relevantă preponderent pentru metode, ne vom orienta în continuare spre analiza duratelor de viață în acest context particular, astfel încât să înțelegem de ce adnotarea manuală a duratelor de viață în semnăturile metodelor nu este, de obicei, necesară.

### Adnotarea duratelor de viață în definirea metodelor

Când definim metode pentru o structură ce include durate de viață, aplicăm aceeași sintaxă ca și pentru parametrii de tipuri generice, demonstrată în Listarea 10-11. Decizia de unde să declarăm și să utilizăm parametrii duratei de viață este influențată de faptul că aceștia sunt asociați cu câmpurile structurii sau cu parametrii metodelor și valorile returnate.

Numele pentru duratele de viață ale câmpurilor structurii sunt întotdeauna necesare după cuvântul `impl` și trebuie utilizate în continuarea numelui structurii, deoarece aceste durate de viață fac parte integrantă din tipul structurii.

În cadrul semnăturilor de metode din blocul `impl`, referințele pot fi conectate la durata de viață a referințelor din câmpurile structurii sau pot fi complet independente. În plus, regulile de eliziune a duratelor de viață percep frecvent în așa fel încât adnotările de durată de viață nu sunt necesare în semnăturile metodelor. Să evaluăm câteva exemple folosind structura `ImportantExcerpt`, pe care am definit-o în Listarea 10-24.

Mai întâi, examinăm o metodă denumită `level`, cu singurul parametru fiind o referință la `self`, care returnează o valoare de tip `i32` și nu este o referință la altceva:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

Este necesară declararea parametrului de durată de viață după `impl` și a utilizării acestuia după numele tipului, însă nu este imperativă adnotarea duratei de viață a referinței la `self` datorită primei reguli de eliziune.

Iată un exemplu în care intervine a treia regulă de eliziune a duratei de viață:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

Dat fiind că există două durate de viață pentru argumente, Rust aplică prima regulă de eliziune și atribuie fiecăruia dintre `&self` și `announcement` propria durată de viață. Deoarece `&self` este unul dintre parametri, tipul de retur primește durata de viață a referinței `&self`, astfel fiind stabilite toate duratele de viață.

### Durata de viață `'static`

Un tip special de durată de viață pe care trebuie să-l abordăm este `'static`, care semnalează că referința în cauză *poate* persista pentru întregul timp al execuției programului. Fiecare literal de tip string are durata de viață `'static`, pe care o putem nota în felul următor:

```rust
let s: &'static str = "Am o durată de viață statică.";
```

Textul acestui string este încapsulat direct în binarul programului, care este constant disponibil. Așadar, durata de viață a tuturor literalelor de string este `'static`.

Poate că vei întâlni sugestii de utilizare a duratei de viață `'static` atunci când apar mesaje de eroare. Însă, înainte de a atribui `'static` ca timp de durată de viață pentru o referință, e vital să te gândești dacă respectiva referință chiar necesită o durată de viață ce acoperă întreaga perioadă a programului, și dacă chiar vrei acest lucru. De regulă, un mesaj de eroare ce recomandă durata de viață `'static` este cauzat de încercarea de a genera o referință suspendată sau de o discrepanță între duratele de viață disponibile. În astfel de situații, este indicat să remediem aceste probleme, și nu să optăm pentru specificarea duratei de viață `'static`.

## Combinarea parametrilor generici de tip, delimitărilor de trăsături și a duratelor de viață

Să abordăm sintaxa necesară pentru a defini parametri generici de tip, delimitări de trăsături și durate de viață, toți concentrați într-o singura funcție!

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

Aceasta este funcția `longest` din Listarea 10-21, care va returna secțiunea de string mai lungă dintre două. Acum a fost adăugat un parametru suplimentar denumit `ann` de tipul generic `T`. Acesta poate fi orice tip ce implementează trăsătura `Display`, așa cum este indicat de clauza `where`. Parametrul suplimentar este afișat utilizând `{}`, de aici și necesitatea delimitării trăsăturii `Display`. Duratele de viață sunt considerate tot un tip de parametri generici, așadar declarațiile parametrului `a` și ale tipului generic `T` sunt listate împreună, în interiorul parantezelor unghiulare care succed numele funcției.

## Sumar

Am abordat numeroase subiecte în acest capitol! Ești acum înarmat cu cunoștințele necesare despre parametrii de tip generic, trăsăturile și delimitările de trăsătură, cât și despre parametrii generici de durată de viață, astfel încât să poți crea cod fără redundanțe, aplicabil în diverse contexte. Parametrii de tip generic îți permit să extinzi aplicabilitatea codului la mai multe tipuri. Folosirea trăsăturilor și a delimitărilor de trăsătură asigură că tipurile generice vor poseda comportamentul cerut de codul tău. Ți s-a prezentat modul de utilizare a adnotărilor de durată de viață pentru a preveni orice referințe suspendate în codul flexibil pe care îl produci. Toate aceste analize se desfășoară în etapa de compilare, fără a afecta performanța la rulare!

Este uimitor cât de multe mai sunt de învățat pe temele pe care le-am parcurs: Capitolul 17 se concentrează pe obiectele-trăsătură, oferindu-ți o nouă perspectivă asupra utilizării trăsăturilor. De asemenea, vei descoperi scenarii mai complexe ce implică adnotările de durată de viață, relevante îndeosebi în cazuri avansate; pentru aceste situații, recomandăm consultarea [Rust Reference][reference]. Dar înainte de aceasta, vei învăța cum să elaborezi testele în Rust, pentru a confirma că programul tău se comportă exact cum trebuie.

[references-and-borrowing]:
ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]:
ch04-03-slices.html#string-slices-as-parameters
[reference]: ../reference/index.html
