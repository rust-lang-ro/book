<!-- Old heading. Do not remove or links may break. -->
<a id="the-match-control-flow-operator"></a>
## Structura de control `match`

Rust dispune de o structură de control extrem de puternică numită `match`, care permite compararea unei valori cu o serie de șabloane și executarea codului în funcție de șablonul care se potrivește. Șabloanele pot fi realizate din valori literale, nume de variabile, wildcard-uri și multe altele. Capitolul 18 acoperă toate aspectele referitoare la șabloane și funcționalitatea acestora. Puterea lui `match` vine din expresivitatea șabloanelor și faptul că compilatorul verifică dacă toate cazurile posibile sunt gestionate.

O expresie `match` poate fi înțeleasă ca o mașină de sortat monede: monedele alunecă pe o pistă cu găuri de diferite mărimi iar fiecare monedă cade prin prima gaură în care se încadrează. Similar, valorile parcurg fiecare șablon într-un `match`, iar la primul șablon unde valoarea se "potrivește", ea este redirecționată în blocul de cod asociat pentru a fi utilizat în timpul execuției.

Luând exemplul monedelor, putem crea o funcție care primește o monedă necunoscută din SUA și, asemenea mașinii de numărat, determină tipul monedei și returnează valoarea acesteia în cenți, așa cum se arată în Listarea 6-3.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

<span class="caption">Listarea 6-3: O enumerare și o expresie `match` ce include variantele enum ca șabloane</span>

Să analizăm `match` în funcția `value_in_cents`. Începând, scriem cuvântul-cheie `match` urmat de o expresie, în acest caz, valoarea `coin`. Aceasta poate părea similară cu o expresie condițională folosită împreună cu `if`, dar există o diferență semnificativă: dacă în cazul `if` condiția trebuie să evalueze o valoare Booleană, aici, poate fi orice tip. `coin` în acest exemplu este enum-ul `Coin` pe care l-am definit în prima linie.

Apoi, avem ramurile `match`. Fiecare ramură conține două părți: un șablon și un cod. Prima ramură are un șablon care corespunde valorii `Coin::Penny` și apoi operatorul `=>` care separă șablonul de codul care trebuie executat. În acest caz, codul este doar valoarea `1`. Fiecare ramură este separată de cea următoare printr-o virgulă.

Când expresia `match` se execută, valoarea rezultată se compară cu șablonul fiecărei ramuri, în ordine. Dacă un șablon corespunde valorii, codul asociat cu acel șablon este executat. Dacă șablonul nu se potrivește valorii, execuția continuă la următoarea ramură, similar cu mașina de sortat monede. Putem avea câte ramuri dorim: în Listarea 6-3, `match`-ul nostru are patru ramuri.

Codul asociat fiecărei ramuri este o expresie, iar valoarea rezultată din expresia ramurii care se potrivește este returnată pentru întreaga expresie `match`.

Nu folosim, de regulă, acolade dacă codul ramurii `match` este scurt, așa cum se vede în Listarea 6-3, unde fiecare ramură returnează doar o valoare. Dacă dorești să execuți mai multe linii de cod într-o ramură `match`, trebuie să folosești acolade, iar virgula care urmează ramurii devine atunci opțională. De exemplu, în codul de mai jos textul "Penny norocos!" este afișat de fiecare dată când metoda este apelată cu `Coin::Penny`, totuși, ultima valoare a blocului, `1`, este returnată:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### Șabloane ce se leagă de valori

O altă particularitate valoroasă a segmentelor `match` este că ele pot încorpora elementele valorilor care corespund cu șablonul definit. Exact în acest mod putem desprinde valori din variantele unei enumerări.

Pentru exemplificare, modificăm una dintre variantele enumerării noastre, astfel încât să cuprindă date. În perioada 1999 - 2008, Statele Unite ale Americii au emis monede "quarter" personalizate, cu design-uri distincte pentru fiecare dintre cele 50 de state. Niciun alt tip de monedă nu a primit acest tratament special, de aceea doar "quarter"-urile au această extra valoare. Putem îngloba această informație în enumerarea noastră prin modificarea variantei `Quarter`, astfel încât să includă o valoare `UsState` în interiorul ei, așa cum am făcut în Listarea 6-4.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

<span class="caption">Listare 6-4: O enumerare `Coin` în care și varianta `Quarter` deține o valoare `UsState`</span>

Să presupunem că un prieten se străduiește să colecționeze monedele "quarter" pentru toate cele 50 de state. În timp ce noi sortăm restul de monede după tip, vom menționa și numele statului asociat cu fiecare "quarter", astfel încât, dacă este una pe care prietenul nostru nu o deține, să o poată adăuga în colecția sa.

În expresia `match` pentru acest segment de cod, adăugăm o variabilă numită `state` la șablon, care se potrivește cu valorile variantei `Coin::Quarter`. Când un `Coin::Quarter` se potrivește, variabila `state` se va lega de valoarea statului respectivului "quarter". Apoi putem utiliza `state` în codul specfic acelui segment din `match`, astfel:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

Dacă am apela `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` ar fi `Coin::Quarter(UsState::Alaska)`. Când comparam valoarea aceasta cu fiecare segment al `match`-ului, niciunul nu se potrivește până la segmentul `Coin::Quarter(state)`. În acest punct, legătura pentru `state` va fi valoarea `UsState::Alaska`. Putem folosi apoi acea legătură în expresia `println!`, extrăgând astfel valoarea internă a statului din varianta "Quarter" din enumerarea `Coin`.

### Corelarea cu `Option<T>`

În secțiunea precedentă, am aspirat să extragem valoarea internă `T` din variantă `Some`, folosind structura `Option<T>`. În ciuda schimbării obiectului de la enum `Coin` la `Option<T>`, funcționarea expresiei `match` rămâne neschimbată.

Consideră că avem nevoie de o funcție care acceptă ca parametru o structură `Option<i32>`. Rolul ei este de a adaugă 1 la valoarea conținută, dacă aceasta există. În caz negativ, funcția nu ar trebui să execute nicio operație și să returneze `None`.

Având la dispoziție expresia `match`, implementarea funcției devine extrem de simplă și intuitivă, asemenea exemplelor prezentate în Lista 6-5.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

<span class="caption">Listarea 6-5: O funcție care utilizează o expresie `match` pe variabila `Option<i32>`</span>

Să examinăm cu mai multă atenție prima execuție a `plus_one`. Atunci când invocăm `plus_one(five)`, variabila `x` din interiorul funcției `plus_one` va primi valoarea `Some(5)`. Aceasta este apoi comparată cu fiecare ramură din instrucțiunea `match`:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Valoarea `Some(5)` nu corespunde modelului `None`, așa că trecem mai departe la următoarea ramură:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

Se potrivește `Some(5)` cu `Some(i)`? Chiar se potrivește! Avem aceeași variantă. Simbolul `i` se leagă apoi de valoarea stocată în `Some`, astfel că `i` devine `5`. Codul din această ramură `match` este executat, deci adăugăm 1 la valoarea lui `i` și producem o nouă valoare `Some` care încapsulează rezultatul nostru, `6`.

Analizăm acum a doua apelare a funcției `plus_one` din Listarea 6-5, unde `x` este `None`. Pătrundem în interiorul `match` pentru a compara cu prima ramură:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Într-adevăr, se potrivește! Nu există o valoare la care să adăugăm, așadar programul încetează și returnează valoarea `None` aflată în partea dreaptă a `=>`. De vreme ce prima ramură s-a potrivit, nicio altă ramură nu va mai fi evaluată.

Folosirea împreună a `match` și a enumerărilor este extrem de utilă în numeroase situații. Astfel de constructe îți vor fi familiare în lucrul tău cu Rust: `match` asociat unui enum, creare de legături cu o variabilă către datele interne și apoi execuția codului în funcție de aceasta. Deși poate părea complex la început, pe măsură ce te obișnuiești cu acesta, vei începe să îți dorești ca acesta să fie disponibil în toate limbajele de programare. Nu întâmplător este unul dintre cele mai îndrăgite caracteristici ale limbajului de către comunitatea de utilizatori.

### Corelările de tip match sunt exhaustive

Mai există un aspect important legat de sintaxa `match`: șabloanele asociate fiecărei ramuri din `match` trebuie să acopere toate scenariile posibile. Să luăm în considerare o versiune modificată a funcției noastre `plus_one`, care conține o eroare și, prin urmare, nu va reuși să compileze:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

Am omis gestionarea cazului `None`, deci acest cod implicit va conține o eroare. Din fericire, aceasta este o eroare pe care Rust o poate identifica. Dacă încercăm să compilăm acest cod, vom întâmpina următoarea eroare:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust e conștient că nu am cuprins fiecare scenariu posibil și indică inclusiv șablonul pe care l-am omis! Corelările de șabloane din Rust sunt *exhaustive*: trebuie să epuizăm fiecare posibilitate pentru ca codul să fie considerat valid. În special în cazul `Option<T>`, atunci când Rust ne împiedică să trecem cu vederea cazul `None`, ne protejează de a face presupuneri incorecte - că avem o valoare, când de fapt ne-am putea confrunta cu o valoare null. Astfel, se evită genul de eroare costisitoare pe care l-am discutat mai devreme.

### Șabloane universale și substituentul `_`

Prin intermediul enumerărilor, putem declanșa acțiuni specifice pentru unele valori speciale, dar pentru restul valorilor vom adopta o acțiune prestabilită. Imaginați-vă că implementăm un joc în care, dacă la o aruncare de zar iese 3, jucătorul nu se mișcă, ci primește o pălărie nouă și elegantă. Dacă zarul arată 7, jucătorul își pierde pălăria elegantă. Pentru orice altă valoare, jucătorul avansează pe tabla de joc cu numărul de spații egal cu valoarea zarului. Iată o implementare cu `match` a aceastei logici, unde rezultatul aruncării cu zarul este hardcoded, în locul unei valori aleatorii, iar restul logicii este reprezentat de funcții fără corpuri deoarece actuala implementare nu intră în sfera acestui exemplu:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```
 
Pentru primele două ramuri, șabloanele sunt valoriile exacte `3` și `7`. Pentru ultima ramură care acoperă toate celelalte valori posibile, șablonul este variabila pe care am ales-o cu numele `other`. Codul ce rulează pentru ramura `other` utilizează această variabilă, trimițând-o către funcția `move_player`.

Acest cod este valid și se va compila, chiar dacă nu am enumerat explicit toate valorile pe care tipul `u8` le poate avea, deoarece ultimul șablon este conceput să acopere toate aceste situații. Acest șablon universal satisface cerința ca `match` să fie exhaustiv. Este important de reținut că acest șablon universal trebuie plasat ultimul, pentru că șabloanele sunt evaluate în ordinea în care au fost scrise. Dacă am adăuga acest șablon universal mai devreme, celelalte ramuri nu ar mai fi accesate, de aceea Rust ne va da o alertă dacă încercăm să adăugăm alte ramuri după șablonul universal! 

Rust pune la dispoziție un șablon numit `_`, pe care îl putem folosi atunci când dorim să implementăm un mecanism universal, însă nu suntem interesați să utilizăm valoarea pe care o captează acest mecanism. Acest șablon special se potrivește cu orice valoare și nu realizează o legătură cu acea valoare, ceea ce indică faptul că nu avem de gând să o utilizăm. Rust ne scutește de avertismentul privind o variabilă neutilizată în acest context.

Analizăm acum un scenariu în care schimbăm regulile jocului: dacă rulezi alt rezultat decât un 3 sau un 7, va trebui să rulezi din nou zarul. Ne putem dispensa de utilizarea valorii universale, deci putem modifica codul pentru a utiliza `_` în locul variabilei denumite `other`: 

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

Acest exemplu îndeplinește și cerința de exhaustivitate, întrucât ignorăm în mod explicit toate celelalte valori în ultimul braț al structurii de control; nu am omis nimic.

Urcăm miza și schimbăm regulile jocului o dată în plus: nimic nu se va întâmpla în tura ta dacă rolezi un rezultat care nu este nici 3, nici 7. Putem exprima acest lucru prin utilizarea valorii unit (tipul de tuplă vidă despre care am vorbit în secțiunea [“Tipul tuplă”][tuples]<!-- ignore -->), value care să fie asociată brațului de cod `_`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

Aici, clarificăm explicit că nu intenționăm să folosim orice altă valoare care nu corespunde unui șablon dintr-un braț anterior, și că nu dorim să se execute vreun cod în acest caz.

Vom aprofunda modul în care funcționează șabloanele și mecanismul de potrivire a acestora în [Capitolul 18][ch18-00-patterns]<!-- ignore -->. Deocamdată, ne vom concentra pe sintaxa `if let`, utilă în situațiile în care expresia `match` pare a fi prea stufoasă.

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch18-00-patterns]: ch18-00-patterns.html
