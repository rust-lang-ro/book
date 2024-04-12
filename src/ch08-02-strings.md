## Manipularea textelor codate UTF-8 utilizând string-uri

Am introdus noțiunile despre string-uri în Capitolul 4, dar acum vom aprofunda subiectul. Utilizatorii noi de Rust au adesea dificultăți cu string-urile din cauza unei combinații a trei aspecte: abordarea Rust de a evidenția posibilele greșeli, complexitatea neașteptată a string-urilor ca structură de date și particularitățile codării UTF-8. Toate acestea pot crea provocări, mai ales pentru cei veniți din alte limbaje de programare.

Vorbim despre string-uri în cadrul temei colecțiilor pentru că, în esență, un string este o colecție de octeți (bytes), completată de metode care facilitează lucrul cu textul. În această secțiune, ne vom concentra pe operațiunile comune oricărui tip de colecție, care se aplică și pentru `String`: cum le creăm, le actualizăm și le citim. Vom discuta și despre caracteristicile care diferențiază `String` de alte tipuri de colecții, concentrându-ne în special pe complexitatea indexării într-un `String`, dată de modul diferit în care oamenii și calculatoarele procesează informația dintr-un `String`.

### Ce este un string?

Să clarificăm ce înțelegem noi prin "string". În limbajul Rust, există un singur tip fundamental de string, anume secțiunea de string - `str`, care de obicei apare sub forma împrumutată `&str`. În Capitolul 4, am abordat conceptul de *secțiuni de string*, care reprezintă referințe la date de tip string codificate în UTF-8 și stocate în altă parte. Spread exemplu, literalele de string sunt incluse în fișierul binar al programului, motiv pentru care sunt considerate secțiuni de string.

Pe de altă parte, avem tipul `String`, oferit de biblioteca standard a limbajului Rust și nu parte integrantă a nucleului limbajului. Acesta este un tip de string extensibil, mutabil, cu proprietate asupra datelor și codificat în UTF-8. Când vorbim despre "string-uri" în Rust, ne referim atât la tipul `String`, cât și la secțiunile de string `&str`, nu exclusiv la unul dintre ele. Deși accentul acestei secțiuni este pe `String`, ambele forme sunt fundamentale în biblioteca standard și, atât `String`, cât și secțiunile de string respectă codificarea UTF-8.

### Crearea unui string nou

Operațiunile pe care le facem cu `Vec<T>` pot fi aplicate și pe `String`, deoarece `String` este efectiv o încapsulare peste un vector de octeți, având anumite garanții suplimentare, restricții și funcționalități. Să luăm drept exemplu funcția `new`, care ne permite să creăm o nouă instanță de `String`, ilustrată în Listarea 8-11.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

<span class="caption">Listarea 8-11: Crearea unui `String` gol</span>

Această linie de cod creează un `String` nou și gol, pe nume `s`, în care putem încărca date ulterior. Adesea avem date inițiale pe care dorim să le folosim pentru a popula string-ul. Pentru acest caz folosim metoda `to_string`, disponibilă pentru orice tip ce implementează trăsătura `Display`, așa cum fac literalele string. Listarea 8-12 prezintă două exemple.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

<span class="caption">Listarea 8-12: Crearea unui `String` dintr-un literal string folosind metoda `to_string`</span>

Codul de mai sus generează un string ce conține textul `initial contents`.

Putem folosi, de asemenea, funcția `String::from` pentru a crea un `String` pornind de la un literal string. Codul din Listarea 8-13 este similar celui din Listarea 8-12, care a utilizat `to_string`.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

<span class="caption">Listarea 8-13: Utilizarea funcției `String::from` pentru a obține un `String` dintr-un literal string</span>

Având în vedere numărul mare de aplicații ale string-urilor, există multe API-uri generice pentru lucrul cu acestea, oferindu-ne o varietate mare de opțiuni. Unele dintre acestea par redundante, dar fiecare își are rostul său! În acest caz, `String::from` și `to_string` îndeplinesc aceeași funcție, astfel alegerea dintre cele două este bazată mai mult pe preferințe de stil și claritate.

Rețineți că string-urile sunt codate în UTF-8, astfel orice date codate adecvat pot fi inclusă în acestea, așa cum este demonstrat în Listarea 8-14.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

<span class="caption">Listarea 8-14: Salvarea mesajelor de salut în diferite limbi în string-uri</span>

Toate acestea reprezintă valori `String` corecte.

### Actualizarea unui string

Un `String` poate să se extindă și să-și schimbe conținutul, similar cu `Vec<T>`, atunci când adaugi mai multe date. De asemenea, e simplu să concatenezi valori `String` folosind operatorul `+` sau macro-ul `format!`.

#### Adăugând la un string cu `push_str` și `push`

Un `String` poate fi mărit adăugând o secțiune de string cu ajutorul metodei `push_str`, așa cum vedem în Listarea 8-15.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

<span class="caption">Listarea 8-15: Adăugarea unei secțiuni de string la un `String` cu `push_str`</span>

După aceste operațiuni, `s` va conține `foobar`. Metoda `push_str` primește o secțiune de string pentru că nu dorește, de obicei, să preia controlul acestuia. De exemplu, în exemplul din Listarea 8-16, vrem ca `s2` să fie utilizabil și după ce l-am concatenat cu `s1`.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

<span class="caption">Listarea 8-16: Utilizarea unei secțiuni de string după ce a fost adăugată la un `String`</span>

Dacă metoda `push_str` ar fi solicitat posesiunea asupra lui `s2`, nu am fi putut afișa valoarea acestuia la sfârșit. Însă, codul funcționează cum ne-am așteptat!

Metoda `push` acceptă un caracter și îl adaugă la `String`. În Listarea 8-17, adăugăm litera "l" la un `String` folosind `push`.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

<span class="caption">Listarea 8-17: Adăugarea unui caracter la un `String` cu `push`</span>

Rezultatul va fi că `s` va conține `lol`.

#### Concatenarea cu operatorul `+` sau macro-ul `format!`

Adesea, ai nevoie să unifici două string-uri existente. Poți face asta folosind operatorul `+`, cum este arătat în Listarea 8-18.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

<span class="caption">Listarea 8-18: Utilizarea operatorului `+` pentru a concatena două valori de tip `String` într-una nouă</span>

`String`-ul `s3` va conține textul `Hello, world!`. După această operație, `s1` nu mai este valid, iar motivul pentru care am utilizat o referință la `s2` este legat de semnătura metodei `add`, apelată prin operatorul `+`. Iată cum arată semnătura:

```rust,ignore
fn add(self, s: &str) -> String {
```

Deși în biblioteca standard `add` este definit cu generice, noi am folosit tipuri specifice pentru a putea explica. Acest lucru se întâmplă automat când invoci metoda cu valori de tip `String`. Vom detalia genericele în Capitolul 10. Semnătura ne oferă indicii pentru a demistifica operatorul `+`.

Mai întâi, observăm că `s2` este precedat de `&`, ceea ce indică faptul că adăugăm o referință la al doilea string. Aceasta este necesar, deoarece funcția `add` acceptă doar o referință `&str` pentru concatenare, nu două valori de tip `String`. Dar de ce funcționează codul din Listarea 8-18, având în vedere că `&s2` este de fapt de tip `&String` şi nu `&str`?

Explicația este că compilatorul poate converti `&String` la `&str` prin intermediul unei coerciții de dereferențiere. Când apelăm metoda `add`, `&s2` este convertit la `&s2[..]`. Aflăm mai multe despre aceasta în Capitolul 15. Deoarece `add` nu preia posesiunea parametrului `s`, `s2` rămâne un `String` valid după operație.

În al doilea rând, `add` preia posesiunea lui `self`, care nu este marcat cu un `&`. Acest lucru înseamnă că `s1` va fi consumat de apelul `add` și nu va mai putea fi folosit în continuare. Prin urmare, instrucțiunea `let s3 = s1 + &s2;` poate părea că doar copiază string-urile pentru a crea unul nou, dar, de fapt, preia `s1`, adaugă la el conținutul copiat din `s2`, și returnează noua valoare. Astfel, în loc să efectueze multe copieri inutile, operația este de fapt mai eficientă.

Pentru concatenarea mai multor string-uri, folosirea operatorului `+` poate deveni incomodă:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

Aici, `s` va fi `tic-tac-toe`. Însă, cu atâtea `+` și `"`, codul devine încărcat și greu de urmărit. Pentru cazuri mai complexe, putem apela la macro-ul `format!`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

Acest fragment de cod atribuie tot `tic-tac-toe` pentru `s`. Macro-ul `format!`, similar cu `println!`, nu afișează rezultatul pe ecran, ci returnează un `String` cu conținutul formatat. Varianta cu `format!` este clar mai lizibilă și nu preia posesiunea niciunui parametru, lucru de preferat în anumite contexte.

### Accesarea elementelor unui string prin index

Accesul la caracterele unui string folosind indexul lor este o practică obișnuită în multe limbaje de programare. Cu toate acestea, în Rust, încercarea de a accesa elemente dintr-un `String` prin indexare va genera o eroare. Iată codul incorect prezentat în Listarea 8-19.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

<span class="caption">Listarea 8-19: Tentativa de utilizare a indexării cu un String</span>

Executarea acestui cod va produce următoarea eroare:

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

Mesajul de eroare explică problema: string-urile din Rust nu permit folosirea indexării. De ce se întâmplă asta? Pentru a înțelege motivul, trebuie să discutăm despre modul în care Rust gestionează string-urile în memorie.

#### Structura internă a unui string

Un `String` este practic un `Vec<u8>`. Să analizăm câteva exemple de string-uri corect codificate în UTF-8 din Listarea 8-14. Începem cu exemplul acesta:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

Aici, `len` va fi 4, ceea ce înseamnă că vectorul care păstrează string-ul "Hola" are o lungime de 4 bytes. Fiecare literă ocupă 1 byte în codificarea UTF-8. Totuși, următoarea linie te poate surprinde. (Observă că acest string începe cu litera mare chirilică Ze, nu numărul 3.)

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

La întrebarea despre lungimea string-ului, ai putea răspunde că este de 12. În realitate, Rust îți spune că este 24: acesta este numărul de bytes necesari pentru a codifica "Здравствуйте" în UTF-8. Acest lucru se datorează faptului că fiecare valoare scalară Unicode în acel string ocupă 2 bytes. Așadar, un indice al octeților string-ului nu corespunde întotdeauna unei valori scalare Unicode valide. Ca exemplu, iată un cod Rust incorect:

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

Știm că `answer` nu va fi `З`, prima literă. În codificarea UTF-8, primul byte pentru `З` este `208`, iar al doilea este `151`. Prin urmare, ai putea crede că `answer` ar trebui să fie `208`, dar `208` nu reprezintă un caracter valid de sine stătător. Acest rezultat nu este probabil ceea ce un utilizator ar aștepta când solicită prima literă a string-ului; în orice caz, Rust nu are la dispoziție alte date la indicele de byte zero. În general, utilizatorii nu doresc valoarea octetului în sine, chiar dacă string-ul constă doar din litere latine: dacă `&"hello"[0]` ar fi un cod valid care returnează valoarea octetului, acesta ar returna `104`, nu `h`.

Concluzia este că pentru a evita generarea unei valori neașteptate și introducerea de bug-uri ce ar putea rămâne nedetectate pentru o perioadă, Rust alege să nu compileze acest cod deloc, prevenind astfel confuziile încă din fazele incipiente ale dezvoltării software.

#### Octeți, valori scalare și grupuri de grafeme

Un alt aspect legat de UTF-8 este faptul că există trei moduri în care Rust consideră string-urile: ca octeți, valori scalare, și grupuri de grafeme (cel mai apropiat termen pentru ceea ce numim "litere").

De exemplu, cuvântul hindi "नमस्ते" scris în scriptul Devanagari este stocat ca un vector de valori `u8` în felul următor:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Acestea sunt 18 octeți și reprezintă modul în care calculatoarele stochează aceste date. Privindu-le ca valori scalare Unicode, care sunt reprezentate de tipul `char` în Rust, acești octeți arată astfel:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Avem aici șase valori `char`, dar a patra și a șasea nu reprezintă litere: ele sunt diacritice fără sens de sine stătător. În final, dacă le privim ca grupuri de grafeme, obținem cele patru "litere" care formează cuvântul hindi:

```text
["न", "म", "स्", "ते"]
```

Rust oferă diferite metode de interpretare a datelor brute ale unui string pentru ca fiecare program să își poată alege abordarea necesară, indiferent de limba umană în care sunt datele.

Un motiv suplimentar pentru care Rust nu permite indexarea unui `String` pentru a extrage un caracter este că se așteaptă ca operațiile de indexare să se efectueze într-un timp constant (O(1)). Dar acest lucru nu poate fi garantat cu `String`, deoarece Rust ar trebui să parcurgă conținutul de la început până la index pentru a identifica numărul de caractere valide.

### Secționarea string-urilor

A accesa un string folosind indici este adesea nerecomandat deoarece nu este clar ce tip de valoare ar trebui să returneze operația de indexare: un octet, un caracter, un cluster de grafeme sau o secțiune de string. Din acest motiv, Rust necesită specificații mai exacte atunci când vrei să folosești indici pentru a obține secțiuni de string.

În loc să te bazezi pe `[]` cu un singur număr, este posibil să utilizezi `[]` cu un diapazon pentru a crea o secțiune ce include anumiți octeți:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

În exemplul de mai sus, `s` este un `&str` care cuprinde primii 4 octeți din string. Anterior, am menționat că fiecare caracter este reprezentat de 2 octeți, deci `s` va conține `Зд`.

Dacă am încerca să extragem doar o parte a octeților unui caracter, cum ar fi cu `&hello[0..1]`, Rust va genera o eroare la rulare asemănătoare celei întâmpinate când se accesează un index invalid într-un array:

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

Utilizează cu atenție diapazoanele când creezi secțiuni de string, deoarece acest lucru poate duce la erori grave în timpul execuției programului.

### Metode de iterație pentru string-uri

Când lucrezi cu părți din string-uri, e important să specifici clar dacă dorești să accesezi caractere sau octeți. Dacă ai nevoie de valori scalare Unicode individuale, folosește metoda `chars`. Dacă aplici `chars` pe textul “Зд”, vei obține două valori de tip `char`, pe care le poți parcurge prin iterare pentru a accesa fiecare caracter:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

Acest exemplu de cod va afișa caracterele:

```text
З
д
```

Pe de altă parte, metoda `bytes` îți returnează octeții individuali ai textului, ceea ce poate fi util în anumite scenarii:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

Acest cod va afișa cei patru octeți care formează textul dat:

```text
208
151
208
180
```

Este important să ții minte că o valoare scalară validă în Unicode poate fi alcătuită din mai mulți octeți.

Extragerea clusterelor de grafeme din string-uri, cum ar fi cele din scrierea Devanagari, este o operație complexă și, din acest motiv, nu este inclusă în biblioteca standard. Pentru această nevoie, poți găsi crate-uri specializate pe [crates.io](https://crates.io/)<!-- ignore -->.

### Simplitatea înșelătoare a string-urilor

Pentru a rezuma, lucrurile cu string-urile sunt complexe. Fiecare limbaj de programare alege diferit cum să îţi prezinte această complexitate. Rust optează pentru gestionarea corectă a datelor `String` ca opțiune standard în orice program Rust. Acest lucru înseamnă că trebuie să acorzi o atenție sporită procesării datelor UTF-8 din start. Această abordare scoate la iveală complexitatea string-urilor mai mult decât în alte limbaje, însă astfel, eviți întâmpinarea erorilor legate de caractere non-ASCII în etapele avansate de dezvoltare a proiectelor tale.

Partea încurajatoare este că biblioteca standard vine la pachet cu multe funcționalități, bazate pe `String` și `&str`, care sunt gândite să ajute la navigarea cu succes prin aceste complexități. Nu rata documentația, unde vei găsi metode valoroase cum ar fi `contains`, pentru căutarea într-un string, sau `replace`, pentru înlocuirea secțiunilor dintr-un string cu alte string-uri.

Acum, să ne îndreptăm atenția spre ceva mai simplu: hash map-urile!
