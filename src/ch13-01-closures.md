<!-- Old heading. Do not remove or links may break. -->
<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>

## Închideri: funcții anonime care captează contextul

Închiderile din Rust sunt funcții anonime pe care le poți stoca într-o variabilă sau le poți folosi ca argumente în alte funcții. Poți defini o închidere într-un anumit punct și mai târziu să o apelezi într-un alt context pentru evaluare. Diferența majoră față de funcții este că închiderile pot prelua valori din domeniul de vizibilitatea (numite și context) în care au fost create. Vom arăta cum aceste posibilități ale închiderilor sporesc reutilizabilitatea codului și permit adaptarea acestuia la necesități specifice.

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>

### Capturarea contextului cu închideri

Să examinăm cum putem utiliza închiderile pentru a captura valori din mediul înconjurător în care sunt definite, pentru a le folosi ulterior. Iată scenariul: Ocazional, compania noastră de tricouri oferă ca promoție un tricou exclusiv, în ediție limitată, uneia dintre persoanele abonate la lista noastră de distribuție de emailuri. Membrii listei de distribuție pot selecta, dacă doresc, culoarea preferată în profilul lor. Dacă persoana aleasă pentru un tricou gratuit a specificat o culoare preferată, atunci va primi un tricou în acea culoare. Dacă persoana nu a indicat o culoare preferată, aceasta va primi culoarea pentru care compania are cel mai mare stoc.

Pentru realizarea acestui lucru, există numeroase abordări. În exemplul nostru, vom utiliza un enum denumit `ShirtColor` care include variantele `Red` și `Blue` (pentru a limita numărul de culori disponibile, simplificând astfel exemplul). Inventarul companiei este reprezentat printr-o structură numită `Inventory`, ce dispune de un câmp denumit `shirts` care conține un `Vec<ShirtColor>`, reprezentarea culorilor tricourilor curent în stoc. Metoda `giveaway`, definită în `Inventory`, preia preferința de culoare a tricoului opțională a câștigătorului promoției și returnează culoarea tricoului pe care îl va primi. Această structură este ilustrată în Listarea 13-1:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

<span class="caption">Listarea 13-1: Exemplul promoției companiei de tricouri</span>

Magazinul `store` definit în funcția `main` are în stoc două tricouri de culoare albastră și unul de culoare roșie, destinate distribuției în cadrul acestei promoții limitate. Invocăm metoda `giveaway` pentru un utilizator care preferă tricouri de culoare roșie și pentru altul care nu are o preferință de culoare anume.

Încă o dată, codul prezentat putea fi implementat în diverse moduri, dar pentru a ne concentra pe închideri, ne-am limitat la conceptele deja cunoscute, cu singura excepție reprezentată de corpul metodei `giveaway`, ce folosește o închidere. Metoda `giveaway`, primește preferința utilizatorului ca parametru de tip `Option<ShirtColor>` și apoi invocă metoda `unwrap_or_else` pe `user_preference`. Metoda [`unwrap_or_else` aplicată pe `Option<T>`][unwrap-or-else]<!-- ignore --> e definită de biblioteca standard Rust. Aceasta acceptă un singur argument: o închidere care nu necesită argumente și care returnează o valoare `T` (tipul stocat în varianta `Some` a lui `Option<T>`, în acest caz `ShirtColor`). Dacă `Option<T>` este `Some`, `unwrap_or_else` returnează valoarea conținută acolo. Dacă `Option<T>` este `None`, `unwrap_or_else` apelează închiderea și returnează rezultatul acesteia.

Pentru `unwrap_or_else` definim expresia `|| self.most_stocked()` ca argument. Aceasta este o închidere fără argumente proprii, evidentă prin absența parametrilor între barele verticale. Corpul închiderii apelează metoda `self.most_stocked()`. Închiderea este definită în acel punct și va fi evaluată de `unwrap_or_else` la momentul necesar.

Executând acest cod primim următorul afișaj:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Un detaliu demn de remarcat este că am pasat către metoda `unwrap_or_else` o închidere care invocă `self.most_stocked()` pe instanța curentă de `Inventory`. Biblioteca standard nu este forțată să cunoască tipurile `Inventory` sau `ShirtColor` pe care le-am definit, nici logica specifică dorită de noi în acest caz. Închiderea reține o referință imutabilă la instanța `self` de `Inventory` și o transmite, împreună cu codul definit, metodei `unwrap_or_else`. În contrast, funcțiile clasice nu pot captura contextul înconjurător în aceeași manieră.

### Inferența de tip și adnotarea pentru închideri

Sunt diferite distincții între funcții și închideri. De obicei, închiderile nu necesită să adnotăm tipurile parametrilor sau tipul valorii de retur, spre deosebire de funcțiile marcate cu `fn`. Adnotările de tip sunt imperios necesare la funcții pentru că fac parte integrantă dintr-o interfață explicit expusă utilizatorilor. Este indispensabil să definim această interfață în mod precis pentru a ne asigura că toți utilizatorii avem un acord unanim privind tipurile de valori pe care o funcție le utilizează și le întoarce. Pe de altă parte, închiderile nu sunt exploatate într-o interfață expusă într-un mod similar: sunt păstrate în variabile și utilizate anonim, fără a fi expuse utilizatorilor bibliotecii noastre.

Închiderile sunt frecvent concise și pertinente doar într-un context restrâns, nu în scenarii arbitrare. În cadrul acestui context delimitat, compilatorul poate infera tipurile parametrilor și tipul valorii returnate, similar cu capacitatea sa de a deduce tipurile pentru majoritatea variabilelor (există cazuri excepționale unde compilatorul necesită și adnotări de tip pentru închideri).

Similar cu variabilele, avem opțiunea de a adnota tipurile dacă dorim să creștem nivelul de explicitate și claritate, cu prețul unui stil mai verbos decât este absolut necesar. Procesul de adnotare a tipurilor pentru o închidere se reflectă în definiția prezentată în Listarea 13-2. În exemplul nostru, definim o închidere și o salvăm într-o variabilă, în loc să o definim imediat în punctul în care o utilizăm ca argument, așa cum am făcut în Listarea 13-1.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

<span class="caption">Listarea 13-2: Adăugarea adnotațiilor de tip opționale pentru parametri și tipul de valoare de retur în închidere</span>

Cu adnotații de tip adăugate, sintaxa închiderilor devine mai asemănătoare cu cea a funcțiilor. Definim o funcție care adaugă 1 la parametrul ei și o închidere cu același comportament pentru comparație. Am inclus și câteva spații pentru alinierea părților corespunzătoare. Aici observăm că sintaxa închiderii este asemănătoare cu cea a funcției, diferența fiind utilizarea simbolului bară și faptul că multă sintaxă este opțională;

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

Prima linie prezintă definiția unei funcții, iar a doua linie o definiție a unei închideri cu adnotații complete. În a treia linie, scoatem adnotațiile de tip din definiția închiderii. În a patra, eliminăm acoladele, ce devin opționale când corpul închiderii constă doar într-o singură expresie. Toate acestea sunt definiții valide care produc același comportament la apelare. Liniile `add_one_v3` și `add_one_v4` necesită ca închiderea să fie evaluată la compilare, întrucât tipurile sunt inferate din contextul utilizării. Situația e similară cu `let v = Vec::new();`, unde sunt necesare adnotații de tip sau valori pentru determinarea tipului de către Rust.

Pentru definițiile închiderilor, compilatorul va infera un singur tip concret pentru fiecare dintre parametrii și pentru valoarea de retur. De exemplu, Listarea 13-3 arată definiția unei închideri concise care se limitează la a returna valoarea primită ca parametru. Această închidere este simplă și e prezentată doar ca exemplu. Remarcăm că nu avem adăugate adnotații de tip în definiție. Din acest motiv, putem invoca închiderea cu orice tip, așa cum am făcut aici cu `String` prima dată. Dacă încercăm să apelăm `example_closure` cu un număr întreg, întâlnim o eroare.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

<span class="caption">Listarea 13-3: Tentativa de apelare a unei închideri cu tipuri inferate utilizând două tipuri diferite</span>

Compilatorul ne semnalează următoarea eroare:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

Când apelăm `example_closure` prima dată cu o valoare de tip `String`, compilatorul inferă că `x` și valoarea de retur a închiderii sunt de tip `String`. Aceste tipuri sunt apoi fixate pentru `example_closure`, generând o eroare de tip atunci când încercăm să utilizăm un tip diferit cu aceeași închidere.

### Capturarea referințelor sau transferul posesiunii

Închiderile pot captura valori din mediul înconjurător în trei moduri, care se mapează direct pe cele trei metode prin care o funcție poate primi un parametru: împrumutând imutabil, împrumutând mutabil și transferând posesiunea. În funcție de cum corpul funcției folosește valorile capturate, închiderea va decide care metodă să folosească.

În Listarea 13-4, definim o închidere care capturează o referință imutabilă către vectorul numit `list`, deoarece are nevoie doar de o referință imutabilă pentru a afișa valoarea:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

<span class="caption">Listarea 13-4: Definirea și apelarea unei închideri care capturează o referință imutabilă</span>

Acest exemplu arată de asemenea că o variabilă poate fi legată de o definiție de închidere, și mai târziu putem apela închiderea utilizând numele variabilei și paranteze, ca și când numele variabilei ar fi numele unei funcții.

Pentru că putem avea mai multe referințe imutabile la `list` în același timp, `list` rămâne accesibil din codul de înainte de definirea închiderii, după definirea închiderii, dar înainte ca închiderea să fie apelată, și după ce închiderea este apelată. Acest cod se compilează, rulează și produce următorul afișaj:

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

În continuare, în Listarea 13-5, modificăm corpul închiderii astfel încât să adauge un element în vectorul `list`. Acum, închiderea capturează o referință mutabilă:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

<span class="caption">Listarea 13-5: Definirea și apelarea unei închideri care capturează o referință mutabilă</span>

Acest cod se compilează, rulează și produce următorul afișaj:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

Remarcăm că nu mai există un `println!` între momentul definirii și cel al apelării închiderii `borrows_mutably`: când se definește `borrows_mutably`, aceasta captează o referință mutabilă la `list`. Nu mai utilizăm închiderea după ce este apelată, deci împrumutul mutabil se sfârșește. Între momentul definirii închiderii și apelul acesteia, un împrumut imutabil pentru a afișa valoarea nu este permis, deoarece în prezența unui împrumut mutabil nu sunt permise alte împrumuturi. Încercați să adăugați un `println!` în acel loc pentru a vedea ce mesaj de eroare apare!

Dacă intenționați să forțați închiderea să preia posesiunea valorilor pe care le folosește din context, chiar dacă în mod strict corpul închiderii nu ar avea nevoie de posesiune, puteți folosi cuvântul cheie `move` înainte de lista de parametri.

Această tehnică este utilă mai ales când transmitem o închidere către un nou fir de execuție (thread) pentru a transfera datele astfel încât să fie posesiunea noului fir. Discuțiile despre firele de execuție și motivele pentru care ai vrea să le folosești le vom aprofunda în Capitolul 16, când abordăm concurența. Până atunci, să analizăm pe scurt crearea unui nou fir de execuție folosind o închidere care necesită utilizarea cuvântului cheie `move`. Listarea 13-6 prezintă Listarea 13-4 modificată pentru a afișa vectorul într-un nou fir de execuție în loc de firul principal:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

<span class="caption">Listarea 13-6: Utilizarea `move` pentru a forța închiderea să preia posesiunea lui `list` în contextul firului de execuție nou creat</span>

Inițializăm un nou fir de execuție, pasându-i o închidere ce urmează a fi executată ca argument. Corpul închiderii afișează lista. În Listarea 13-4, închiderea a capturat doar `list` printr-o referință imutabilă, pentru că asta era tot accesul necesar pentru a tipări `list`. În exemplul prezent, deși corpul închiderii încă necesită doar o referință imutabilă, este necesar să specificăm că `list` trebuie să fie permutat în închidere, prin plasarea cuvântului cheie `move` la începutul definiției închiderii. Este posibil ca noul fir de execuție să se finalizeze înainte de a se încheia restul firului principal, sau invers. Dacă firul principal ar păstra posesiunea lui `list` și s-ar termina înaintea firului nou și ar distruge `list`, referința imutabilă din cadrul noului fir ar deveni invalidă. Așadar, compilatorul impune ca `list` să fie permutat în închiderea destinată noului fir de execuție, pentru a asigura validitatea referinței. Încearcă să elimini cuvântul cheie `move` sau să utilizezi `list` în firul principal după definirea închiderii, pentru a descoperi ce erori de compilare apar!

<!-- Old headings. Do not remove or links may break. -->
<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>

### Permutarea valorilor capturate din închideri și trăsăturile `Fn`

După ce o închidere a capturat fie o referință, fie a luat posesiunea unei valori din mediul unde este definită (influențând astfel ce anume este permutat *în* închidere), codul din corpul închiderii stabilește ce se va întâmpla cu referințele sau valorile atunci când închiderea este evaluată mai târziu (influențând astfel ce anume este permutat *din* închidere). Corpul închiderii poate realiza oricare dintre acțiunile următoare: să permute o valoare capturată din închidere, să modifice valoarea capturată, să nu permute și să nu modifice valoarea, sau să nu captureze deloc din mediul înconjurător inițial.

Modul cum o închidere capturează și administrează valorile din mediu dictează ce trăsături `Fn` va implementa, iar aceste trăsături sunt modul prin care funcțiile și structurile pot specifica ce tipuri de închideri sunt compatibile. Închiderile vor implementa automat una, două sau chiar toate cele trei trăsături `Fn`, în mod aditiv, depinzând de cum corpul închiderii gestionează valorile:

1. `FnOnce` se aplică pentru închiderile care pot fi folosite o singură dată. Toate închiderile implementează minim această trăsătură, din moment ce orice închidere poate fi folosită. O închidere care permută valorile capturate din corpul propriu va implementa exclusiv `FnOnce` și nicio altă trăsătură `Fn`, fiindcă poate fi utilizată doar o singură dată.
2. `FnMut` se aplică pentru închiderile care nu permută valorile capturate din corpul lor, dar care pot modifica aceste valori capturate. Aceste închideri pot fi folosite de mai multe ori.
3. `Fn` se aplică pentru închiderile care nu permută valorile capturate din corpul lor și nu modifică aceste valori capturate, precum și pentru închiderile care nu capturează nimic din propriul mediu. Astfel de închideri pot fi apelate de multiple ori fără a modifica mediul lor, lucru esențial în situații precum apelul multiplu simultan al unei închideri.

Să examinăm definiția metodei `unwrap_or_else` pentru `Option<T>` prezentată în Listarea 13-1:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Reamintim că `T` este tipul generic destinat valorii din varianta `Some` a unui `Option`. Același tip `T` este, de asemenea, tipul de retur al funcției `unwrap_or_else`: atunci când este apelată pe un `Option<String>`, spre exemplu, va returna un `String`.

Continuând, observăm că funcția `unwrap_or_else` introduce un tip generic suplimentar `F`. Acest tip `F` corespunde parametrului denumit `f`, care este închiderea utilizată atunci când se apelează `unwrap_or_else`.

Limita impusă tipului generic `F` este `FnOnce() -> T`, adică `F` trebuie să poată fi invocat o singură dată, fără argumente și să returneze un `T`. Prin utilizarea `FnOnce` în delimitarea de trăsătură, se specifică faptul că `unwrap_or_else` va apela `f` cel mult o singură dată. Analizând corpul funcției `unwrap_or_else`, rezultă că în cazul unei variante `Some`, funcția `f` nu este apelată. În contrast, dacă varianta este `None`, `f` va fi invocată o dată. Fiindcă toate închiderile implementează `FnOnce`, `unwrap_or_else` poate accepta cel mai larg spectru de închideri, fiind astfel extrem de flexibilă.

> Notă: Funcțiile pot implementa, de asemenea, toate cele trei trăsături `Fn`. Dacă demersul pe care doriți să-l executați nu necesită capturarea unei valori din mediu, puteți opta pentru numele unei funcții, în loc de o închidere, ori de câte ori este nevoie de ceva care implementează una din trăsăturile `Fn`. De exemplu, pentru o valoare `Option<Vec<T>>`, am putea folosi `unwrap_or_else(Vec::new)` pentru a obține un vector nou și gol dacă valoarea respectivă este `None`.

Acum să analizăm metoda `sort_by_key` a bibliotecii standard, care este definită pe secțiuni (slice), pentru a înțelege cum diferă aceasta de `unwrap_or_else` și motivul pentru care `sort_by_key` recurge la `FnMut` în loc de `FnOnce` ca delimitare de trăsătură. Închiderea primește un argument sub formă de referință la elementul actual din secțiunea în curs de evaluare și returnează o valoare de tip `K`, care poate fi ordonată. Această funcție este practică atunci când intenționăm să sortăm o secțiune după un anumit atribut al elementelor sale. În Listarea 13-7, dispunem de o listă de instanțe `Rectangle` și utilizăm `sort_by_key` pentru a le sorta după atributul `width`, de la cel mai mic la cel mai mare:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

<span class="caption">Listarea 13-7: Utilizarea metodei `sort_by_key` pentru sortarea dreptunghiurilor în funcție de lățime</span>

Codul afișează următoarea ieșire:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

`sort_by_key` este definită să accepte închideri de tip `FnMut` deoarece invocă închiderea de mai multe ori - câte o dată pentru fiecare element din secțiune. Închiderea `|r| r.width` nu captează, nu modifică și nu permută nimic din contextul său, îndeplinind astfel criteriile delimitării de trăsătură.

Pe de altă parte, în Listarea 13-8 este prezentat un exemplu de închidere care implementează doar trăsătura `FnOnce`, deoarece permută o valoare din context. Compilatorul nu ne permite să utilizăm acest tip de închidere cu metoda `sort_by_key`:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

<span class="caption">Listarea 13-8: Tentativa de utilizare a unei închideri `FnOnce` cu `sort_by_key`</span>

Aceasta este o încercare forțată și complicată (care nu funcționează) de a număra de câte ori `sort_by_key` este invocată în timpul sortării `list`. Codul încearcă să realizeze această contorizare prin adăugarea `value`—un `String` din contextul închiderii—în vectorul `sort_operations`. Închiderea preia `value` și apoi îl permută în afara închiderii transferând posesiunea lui `value` către vectorul `sort_operations`. Această închidere poate fi folosită doar o dată; o nouă încercare de a o apela nu ar funcționa pentru că `value` nu ar mai fi prezent în context pentru a fi adăugat din nou în `sort_operations`! Prin urmare, închiderea implementează exclusiv `FnOnce`. În momentul la care încercăm să compilăm acest cod, primim următoarea eroare cum că `value` nu poate fi permutat din închidere pentru că închiderea trebuie să implementeze `FnMut`:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

Eroarea indică spre linia din corpul închiderii unde `value` este permutat din context. Pentru a corecta acest lucru, trebuie să modificăm corpul închiderii astfel încât să nu mai permutăm valori din context. O metodă mai simplă și directă de a contoriza câte apeluri ale funcției `sort_by_key` se efectuează, este să păstrăm un contor în context și să-l incrementăm în corpul închiderii. Închiderea din Listarea 13-9 funcționează cu `sort_by_key` pentru că se limitează la capturarea unei referințe mutabile la contorul `num_sort_operations`, permițând astfel apelări multiple:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

<span class="caption">Listarea 13-9: Folosirea unei închideri `FnMut` împreună cu `sort_by_key` este acceptată</span>

Trăiturile `Fn` sunt cruciale atunci când definim sau folosim funcții sau tipuri care exploatează închideri. În următoarea secțiune, ne vom axa pe iteratori. Diverse metode ale iteratorilor solicită argumente sub formă de închideri, deci nu uita aceste detalii legate de închideri în timp ce mergem mai departe!
[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
