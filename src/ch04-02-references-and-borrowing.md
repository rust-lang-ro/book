## Referințe și procesul de împrumutare

Dificultatea pe care o întâmpinăm cu codul pentru tuplă, din Listarea 4-5, este că trebuie să returnăm `String`-ul către funcția apelantă pentru a putea utiliza în continuare `String`-ul după apelul funcției `calculate_length`. Aceasta se datorează faptului că `String`-ul a fost permutat în interiorul funcției `calculate_length`. O soluție alternativă ar fi să furnizăm o referință la valoarea `String`-ului. O *referință* funcționează similar unui pointer - ea reprezintă o adresă ce permite accesul la datele stocate la acea adresă; aceste date fiind deținute de o altă variabilă. Însă, spre deosebire de un pointer, o referință este garantată să indice o valoare validă a unui anumit tip, pe toată durata vieții acestei referințe.

Iată cum definim și utilizăm funcția `calculate_length` care are ca parametru o referință la un obiect, în loc să preia posesiunea valorii:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

În primul rând, observăm că tot codul referitor la tuplă, din declarația variabilei și valoarea de return a funcției, a dispărut. În al doilea rând, remarcăm faptul că transmitem `&s1` către funcția `calculate_length` și că, în definiția sa, preluăm `&String` și nu `String`. Aceste semne de ampersand reprezintă *referințe* și permit să te referi la o anumită valoare fără a-i prelua posesiunea. Conceptul este reprezentat în Figura 4-5.

<img alt="Trei tabele: tabelul destinat lui s conține doar un pointer către tabelul
asociat lui s1. Tabelul pentru s1 include datele din stivă pentru s1 și direcționează
spre datele de tip string stocate pe heap." src="img/trpl04-05.svg" class="center" />

<span class="caption">Figura 4-5: Diagrama ilustrând `&String s` care direcționează spre `String s1`</span>

> Notă: Procesul invers referențierii prin utilizarea `&` este
> *dereferențierea*, realizată cu ajutorul operatorului de dereferențiere,
> `*`. Vom explora câteva situații de utilizare a operatorului de
> dereferențiere în Capitolul 8 și vom discuta în detaliu despre
> dereferențiere în Capitolul 15.

Să examinăm mai în detaliu apelul funcției de aici:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

Sintaxa `&s1` ne permite să creăm o referință care *indică* valoarea `s1` însă fără a o deține. Dat fiind faptul că nu o deține, valoarea la care face referire nu va fi abandonată (dropped) când referința nu mai este folosită.

În mod similar, semnătura funcției folosește `&` pentru a indica faptul că tipul parametrului `s` este o referință. Să adăugăm câteva adnotări care să clarifice aceste aspecte:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

Domeniul în care este validă variabila `s` coincide cu cel al oricărui parametru de funcție. Cu toate acestea, valoarea către care se referă nu este eliminată când nu mai folosim `s`. Motivul este simplu: `s` nu are posesiunea acestei valori. Dacă funcțiile noastre folosesc referințe ca parametri, în locul valorilor propriu-zise, nu trebuie să returnăm valorile pentru a restitui posesiunea, deoarece, de fapt, nu am avut niciodată posesiunea acestora.

Acest act de a crea o referință îl numim *împrumutare*. În mod similar cu viața de zi cu zi, dacă o persoană deține ceva, tu poți să îi împrumuți acel lucru. Odată ce ai terminat cu acel lucru, trebuie să îl restitui. Întrucât nu îți aparține.

Așadar, ce se întâmplă dacă încercăm să modificăm ceva ce doar împrumutăm? Încearcă codul din Listarea 4-6. Dar te previn: nu o să iasă exact cum te-ai așteptat!

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

<span class="caption">Listarea 4-6: Tentativa de a modifica o valoare împrumutată</span>

Aceasta este eroarea întâlnită:

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

Exact așa cum sunt imutabile variabilele, în mod implicit, la fel sunt și referințele. Nu ne este permisă modificarea unei valori la care deținem doar o referință.

### Referințe mutabile

Putem corecta codul din Listarea 4-6 pentru a ne oferi posibilitatea de a modifica o valoare împrumutată, prin câteva ajustări minore care implică utilizarea unei *referințe mutabile*:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

În primul rând, modificăm `s` pentru a fi `mut`. Apoi, creăm o referință mutabilă cu `&mut s` în momentul în care invocăm funcția `change`, și actualizăm semnătura funcției pentru a accepta o referință mutabilă cu `some_string: &mut String`. Astfel, devine foarte clar că funcția `change` va modifica valoarea pe care o împrumută.

Referințele mutabile vin însă cu o limitare importantă: dacă deții o referință mutabilă la o valoare, nu poți deține alte referințe către aceeași valoare. Acest cod, care încearcă să creeze două referințe mutabile la `s`, va da eroare:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

Și eroarea:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

Această eroare ne informează că respectivul cod nu este valid, întrucât nu se poate împrumuta `s` în mod mutabil de mai multe ori concomitent. Primul împrumut mutabil se realizează în `r1` și trebuie să continue până când este folosit în instrucțiunea `println!`. Cu toate acestea, între momentul creării acestei referințe mutabile și momentul utilizării sale, am încercat să generăm o altă referință mutabilă în `r2`, care împrumută aceleași date ca și `r1`.

Restricția care interzice mai multe referințe mutabile simultane la aceleași date ne permite să modificăm datele, dar într-un mod extrem de controlat. Aceasta este o provocare des întâlnită pentru cei nou-veniți în lumea Rust, deoarece majoritatea limbajelor de programare permit modificarea datelor oricând. Avantajul acestei restricții este faptul că Rust poate preveni apariția curselor de date în timpul compilării. O *cursă de date* este similară cu o condiție de cursă și se produce atunci când au loc următoarele trei evenimente:

* Două sau mai multe pointere accesează simultan aceleași date.
* Cel puțin un pointer este utilizat pentru a scrie date.
* Nu există niciun mecanism în vigoare care să sincronizeze accesul la date.

Cursele de date creează un comportament nedefinit și pot fi dificile de diagnosticat și remediat atunci când încerci să le detectezi în timpul execuției; Rust preîntâmpină acest gen de problemă prin faptul că refuză să compileze codul în care apar curse de date!

Ca de obicei, avem posibilitatea de a utiliza acolade pentru a iniția un nou domeniu de vizibilitate, astfel facilitând existența mai multor referințe mutabile; singura condiție este aceea de a nu fi simultane:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust impune o regulă similară pentru combinarea referințelor mutabile și imutabile. Acest cod generează o eroare:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

Eroarea:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

*De asemenea*, nu ne este permis să avem o referință mutabilă în același timp cu o referință imutabilă către aceeași valoare.

Cei ce folosesc o referință imutabilă nu se așteaptă ca valoarea să se schimbe brusc, fără un avertisment! Totuși, sunt permise multiple referințe imutabile, deoarece nicio persoană ce doar citește datele nu are abilitatea de a interfera cu lectura altcuiva a acelor date.

Trebuie să reții că domeniul de vizibilitate al unei referințe începe de unde este aceasta introdusă și continuă până la ultima utilizare a respectivei referințe. De pildă, acest cod se va compila deoarece ultima folosire a referințelor imutabile, și anume `println!`, are loc înainte de a fi introdusă referința mutabilă:

```rust,edition2021
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

Domeniile de vizibilitate pentru referințele imutabile `r1` și `r2` se încheie imediat după ce au fost utilizate pentru ultima dată în `println!`. Asta se întâmplă înainte ca referința mutabilă `r3` să fie creată. Aceste domenii nu se intersectează, deci nu interferă unul cu celălalt, făcând acest cod valid. Compilatorul Rust este în stare să determine că referința nu mai este folosită înainte de sfârșitul domeniului de vizibilitate.

Știm că erorile legate de împrumutări pot fi frustrante uneori. Totuși, trebuie să înțelegem că acesta este un mecanism prin care compilatorul Rust ne avertizează asupra unui potențial bug în stadiul de compilare, nu în timpul rulării. În plus, ne indică exact locul unde apare problema. Aceasta îți va economisi timp deoarece nu va trebui să cauți motivul pentru care datele tale nu corespund cu ceea ce te așteptai.

### Referințe fără destinație

În limbajele de programare ce utilizează pointeri, este foarte ușor să generăm, chiar și involuntar, un *pointer în aer* (dangling) - un pointer ce își pierde legătura cu o parte de memorie care poate fi atribuită altei operațiuni - prin simpla eliberare a unei părți din memoria alocate, în timp ce pointerul către acea memorie rămâne încă activ. În contrast, Rust, prin intermediul compilatorului, garantează că niciodată nu vor exista referințe fără destinație: dacă avem o referință la niște date, compilatorul se va asigura că acele date nu vor fi scoase din domeniul de vizibilitate înainte ca referința spre ele să o facă.

Acum, să încercăm să generăm o referință fără destinație, doar pentru a observa cum Rust previne acest lucru printr-o eroare la etapa de compilare:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

Vom primi eroarea:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

Acest mesaj de eroare face referire la un concept pe care încă nu l-am abordat: durate de viață. Vom aprofunda această temă în Capitolul 10. Cu toate acestea, chiar dacă nu ținem cont de porțiunile ce se referă la durate de viață, mesajul (tradus) ne oferă o pistă esențială pentru a intelege de ce acest fragment de cod nu funcționează:

```text
tipul de retur al acestei funcții conține o valoare împrumutată, dar nu există nicio valoare pentru a fi împrumutată
```

Să examinăm mai atent ce se petrece la fiecare pas în codul nostru `dangle`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

Dat fiind că `s` este generat în interiorul `dangle`, la finalizarea rulării codului din `dangle`, `s` va fi dezalocat. Cu toate acestea, noi am încercat să returnăm o referință către `s`. Aceasta presupune că referința noastră ar fi îndreptată către un `String` ce devine invalid. Situația nu este deloc favorabilă! Rust nu ne va permite să desfășurăm o astfel de acțiune.

Soluția în acest caz constă în returnarea directă a `String`:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

Aceasta funcționează fără nici o problemă. Posesiunea este permutată și nimic nu este dezalocat.

### Regulamentul referințelor

Să reamintim principiile pe care le-am discutat despre referințe:

* În orice clipă, ai voie să deții *ori* o singură referință mutabilă *sau* o cantitate nelimitată de referințe imutabile.
* Referințele trebuie să fie valide în permanență.

Următoarea etapă va fi examinarea unui alt tip de referință: slice-urile.
