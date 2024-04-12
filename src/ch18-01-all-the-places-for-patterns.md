## Toate locurile unde pattern-urile pot fi utilizate

Pattern-urile pot fi întâlnite în mai multe locuri în Rust, fiind utilizate frecvent fără a ne da seama! În această secțiune, vom vedea toate locurile unde aceste pattern-uri sunt valide.

### Ramurile `match`

După cum am explicat în Capitolul 6, pattern-urile sunt utilizate în ramurile expresiilor `match`. Formal, expresiile `match` sunt definite prin cuvântul cheie `match`, o valoare cu care se face potrivirea, și unul sau mai multe ramuri de `match` care conțin un pattern și o expresie ce va fi executată dacă valoarea se potrivește cu pattern-ul respectivului ram, astfel:

```text
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

De exemplu, aceasta este expresia `match` din Listarea 6-5 care face potrivirea pe o valoare `Option<i32>` în variabila `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Pattern-urile din această expresie `match` sunt `None` și `Some(i)`, care se află în partea stângă a fiecărei săgeți.

O cerință pentru expresiile `match` este ca acestea să fie *exhaustive*, adică toate posibilitățile pentru valoarea subiect a expresiei `match` să fie acoperite. Una dintre metodele de a asigura că toate posibilitățile sunt cuprinse este de a include un pattern universal pentru ultimul ram: de exemplu, folosirea unui nume de variabilă care se potrivește cu orice valoare nu va da greș niciodată, cuprinzând astfel fiecare caz nemenționat anterior.

Pattern-ul specific `_` va potrivi orice, însă nu se va lega de vreo variabilă, așa că deseori este utilizat în ultimul ram de `match`. Pattern-ul `_` poate fi folositor atunci când dorim să ignorăm orice valoare care nu a fost specificată anterior, de exemplu. Vom analiza pattern-ul `_` în mai multe detalii în secțiunea [„Ignorarea valorilor într-un pattern”][ignoring-values-in-a-pattern]<!-- ignore -->, mai târziu în acest capitol.

### Expresii condiționale `if let`

În Capitolul 6, am discutat utilizarea expresiilor `if let` ca o formă mai concisă pentru a scrie echivalentul unei expresii `match` care se potrivește doar cu un singur caz. Opțional, `if let` poate include și un bloc `else`, care conține codul ce va fi executat dacă pattern-ul din `if let` nu se potrivește.

Listarea 18-1 demonstrează că este posibil să combinăm și să alternăm expresiile `if let`, `else if`, și `else if let`. Aceasta ne oferă o flexibilitate sporită în comparație cu o expresie `match`, unde putem compara o singură valoare cu pattern-urile. Mai mult, Rust nu solicită ca condițiile dintr-o serie de verificări `if let`, `else if`, `else if let` să fie interconectate.

Codul prezentat în Listarea 18-1 stabilește ce culoare să folosim pentru fond bazat pe o serie de condiții multiple. În acest exemplu, am definit variabile cu valori prestabilite, așa cum într-un program real acestea ar putea fi primite din intrările utilizatorilor.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-01/src/main.rs}}
```

<span class="caption">Listarea 18-1: Alternanța expresiilor `if let`, `else if`, `else if let`, și `else`</span>

Dacă utilizatorul indică o culoare preferată, aceasta este utilizată ca fond. Dacă nu este specificată nicio culoare preferată și astăzi e marți, culoarea de fond va fi verde. În caz contrar, dacă utilizatorul specifică vârsta sa ca string și putem să o convertim cu succes într-un număr, culoarea va fi ori mov, ori portocaliu, în funcție de valoarea numărului. Dacă niciuna dintre aceste condiții nu este îndeplinită, culoarea de fond va fi albastru.

Structura condițională descrisă ne permite gestionarea unor cerințe complexe. Având în vedere valorile prestabilite din acest exemplu, rezultatul va fi afișarea mesajului `Using purple as the background color`.

Reiese că expresiile `if let` pot introduce variabile umbrite în aceeași manieră ca brațele unui `match`: linia cu `if let Ok(age) = age` introduce o nouă variabilă umbrită `age`, care stochează valoarea din varianta `Ok`. De aceea, condiția `if age > 30` trebuie să se afle în blocul respectiv de cod: nu putem combina aceste două condiții în `if let Ok(age) = age && age > 30`, căci variabila umbrită `age` pe care dorim să o comparăm cu 30 nu devine validă decât în momentul inițierii noului domeniu de vizibilitate cu acolada.

Un dezavantaj al folosirii `if let` este că compilatorul nu verifică exhaustivitatea, spre deosebire de expresiile `match`, care sunt verificate. Dacă am omite blocul `else` final, ratând astfel tratarea unor cazuri, compilatorul nu ne-ar semnala posibila greșeală de logică.

### Bucle condiționale `while let`

Asemenea construcției `if let`, bucla condițională `while let` permite buclei `while` să fie executată cât timp un pattern continuă să fie potrivit. În Listarea 18-2, implementăm o buclă `while let` care folosește un vector ca stivă și afișează valorile din vector în ordinea inversă în care au fost introduse.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-02/src/main.rs:here}}
```

<span class="caption">Listarea 18-2: Utilizarea unei bucle `while let` pentru a afișa valori atâta timp cât `stack.pop()` returnează `Some`</span>

Acest exemplu afișează 3, 2 și apoi 1. Metoda `pop` înlătură ultimul element din vector și returnează `Some(value)`. Dacă vectorul este gol, `pop` returnează `None`. Bucala `while` continuă execuția codului din blocul său cât timp `pop` returnează `Some`. Când `pop` returnează `None`, bucla se oprește. Utilizăm `while let` pentru a elimina pe rând fiecare element din stiva noastră.

### Buclele `for`

Într-o buclă `for`, valoarea care urmează imediat după cuvântul cheie `for` este un pattern. De exemplu, în `for x in y`, `x` este pattern-ul. Listarea 18-3 ne arată cum folosim un pattern într-o buclă `for` pentru a destructura o tuplă ca parte a execuției buclei `for`.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-03/src/main.rs:here}}
```

<span class="caption">Listarea 18-3: Utilizarea unui pattern într-o buclă `for` pentru a destrucura o tuplă</span>

Codul din Listarea 18-3 va afișa următorul output:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-03/output.txt}}
```

Modificăm un iterator cu metoda `enumerate` pentru ca acesta să ofere o valoare și indicele acelei valori într-o tuplă. Prima valoare generată este tupla `(0, 'a')`. Când această valoare potrivește cu pattern-ul `(index, value)`, `index` va fi `0`, iar `value` va fi `'a'`, ceea ce rezultă în afișarea primei linii a output-ului.

### Declarațiile `let`

Până la acest capitol, am discutat explicit despre utilizarea pattern-urilor doar cu `match` și `if let`, dar de fapt, am aplicat pattern-uri și în alte contexte, inclusiv în declarațiile `let`. De exemplu, să luăm în considerare această atribuire simplă de variabilă cu `let`:

```rust
let x = 5;
```

De fiecare dată când ai utilizat o declarație `let` cum este aceasta, ai folosit pattern-uri, chiar dacă poate nu ai realizat acest lucru! Mai formal, o declarație `let` se prezintă în felul următor:

```text
let PATTERN = EXPRESSION;
```

În declarații precum `let x = 5;` cu un nume de variabilă în poziția `PATTERN`, numele variabilei este de fapt o formă extrem de simplificată de pattern. Rust corespunde expresia cu pattern-ul și atribuie orice nume identifică. Așadar, în exemplul `let x = 5;`, `x` este un pattern care înseamnă “asociază ce se potrivește aici variabilei `x`.” Întrucât `x` reprezintă întregul pattern, acesta de fapt înseamnă “asociază orice valoare variabilei `x`, indiferent de aceasta.”

Pentru a înțelege mai clar aspectul de potrivire de pattern într-o declarație `let`, să ne referim la Listarea 18-4, unde se utilizează un pattern cu `let` pentru a destructura o tuplă.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-04/src/main.rs:here}}
```


<span class="caption">Listarea 18-4: Utilizând un pattern pentru a destrucura o tuplă și a crea trei variabile simultan</span>

În acest caz, se face un match între o tuplă și un pattern. Rust compară valoarea `(1, 2, 3)` cu pattern-ul `(x, y, z)` și observă că valoarea se potrivește cu pattern-ul, astfel Rust leagă `1` la `x`, `2` la `y`, și `3` la `z`. Acest pattern de tuplă poate fi văzut ca o compoziție a trei pattern-uri individuale de variabila.

Dacă numărul elementelor din pattern nu se potrivește cu numărul elementelor din tuplă, tipul total nu va corespunde și vom întâlni o eroare de compilare. De exemplu, Listarea 18-5 prezintă o tentativă de a destructura o tuplă cu trei elemente în doar două variabile, ceea ce nu va funcționa.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-05/src/main.rs:here}}
```

<span class="caption">Listarea 18-5: Crearea incorectă a unui pattern în care variabilele nu corespund cu numărul de elemente din tuplă</span>

Încercarea de a compila acest cod rezultă în următoarea eroare de tip:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-05/output.txt}}
```

Pentru a rezolva eroarea, putem omite una sau mai multe valori ale tuplei folosind `_` sau `..`, cum vom vedea în secțiunea [„Ignorarea valorilor într-un pattern”][ignoring-values-in-a-pattern]<!-- ignore -->. Dacă problema este că avem prea multe variabile în pattern, soluția constă în a ajusta tipurile prin eliminarea variabilelor până când numărul de variabile este egal cu numărul elementelor din tuplă.

### Parametrii funcțiilor

Parametrii funcțiilor pot de asemenea să fie pattern-uri. Codul din Listarea 18-6, care declară o funcție denumită `foo` ce primește un parametru numit `x` de tip `i32`, ar trebui să ne fie acum cunoscut.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-06/src/main.rs:here}}
```

<span class="caption">Listarea 18-6: O semnătură de funcție utilizează pattern-uri în parametri</span>

Partea cu `x` reprezintă un pattern! Așa cum am procedat cu `let`, se poate potrivi o tuplă cu un pattern în argumentele unei funcții. Listarea 18-7 demonstrează despărțirea valorilor dintr-o tuplă pe când sunt transmise unei funcții.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-07/src/main.rs}}
```

<span class="caption">Listarea 18-7: O funcție cu parametri ce destructurează o tuplă</span>

Acest fragment de cod afișează `Current location: (3, 5)`. Valorile `&(3, 5)` se potrivesc cu pattern-ul `&(x, y)`, astfel `x` devine valoarea `3` iar `y`, valoarea `5`.

Putem folosi de asemenea și pattern-uri în listele de parametri ale închiderilor în mod similar cu listele de parametri ale funcțiilor, având în vedere că închiderile sunt asemănătoare cu funcțiile, așa cum am discutat în Capitolul 13.

Ajuns aici, am avut ocazia să vedem câteva metode de utilizare a pattern-urilor, însă acestea nu funcționează identic în toate locurile în care le putem folosi. În unele situații, pattern-urile trebuie să fie irefutabile, în timp ce în altele pot fi refutabile. Vom explora aceste două concepte în detaliu în continuare.

[ignoring-values-in-a-pattern]:
ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern
