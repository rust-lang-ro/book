## Anexa A: Cuvinte cheie

Următoarea listă conține cuvinte cheie care sunt rezervate pentru uzul curent sau viitor
de către limbajul Rust. Ca atare, acestea nu pot fi utilizate ca identificatori (cu excepția
ca identificatori "raw", așa cum vom discuta în secțiunea "[Identificatori Raw][raw-identifiers]<!-- ignore -->"). Identificatorii sunt nume
de funcții, variabile, parametri, câmpuri struct, module, crate-uri, constante,
macro-uri, valori statice, atribute, tipuri, traits, sau lifetimes.

[raw-identifiers]: #raw-identifiers

### Cuvinte cheie actualmente în uz

Următoarea este o listă de cuvinte cheie actualmente în uz, cu funcționalitatea lor 
prezentată.

* `as` - efectuează tip casting primitiv, disambiguează trait-ul specific care conține
  un item sau redenumeste items în instrucțiunea `use`
* `async` - întoarce un `Future` în loc de a bloca firul curent de execuție
* `await` - suspendă execuția până când rezultatul unui `Future` este gata
* `break` - iese imediat dintr-o buclă
* `const` - definește constante sau pointeri raw constanți
* `continue` - continuă la următoarea iterație de buclă
* `crate` - în calea unui modul, se referă la rădăcina crate-ului
* `dyn` - expediază dinamic către un obiect trait
* `else` - fallback pentru construcții de control `if` și `if let`
* `enum` - definește o enumerație
* `extern` - leagă o funcție sau o variabilă externă
* `false` - literal Boolean fals
* `fn` - definește o funcție sau tipul unui pointer de funcție
* `for` - trece prin elementele unui iterator, implementează un trait sau specifică un
  durată de viață de rang superior
* `if` - alege în funcție de rezultatul unei expresii condiționale
* `impl` - implementează funcționalitatea inherentă sau a unei trăsături
* `in` - face parte din sintaxa buclei `for`
* `let` - leagă o variabilă
* `loop` - buclă necondiționată
* `match` - potrivește o valoare cu modele
* `mod` - definește un modul
* `move` - face ca o închidere să preia posesiunea tuturor capturilor sale
* `mut` - denotă mutabilitate în referințe, pointeri raw, sau legături de pattern
* `pub` - denotă vizibilitate publică în câmpuri struct, blocuri `impl` sau module
* `ref` - leagă prin referință
* `return` - întoarce din funcție
* `Self` - un alias pentru tipul pe care îl definim sau implementăm
* `self` - subiectul metodei sau modulul curent
* `static` - variabilă globală sau timp de viață pe întreaga durată de execuție a programului
* `struct` - definește o structură
* `super` - modulul părinte al modulului curent
* `trait` - definește o trăsătură
* `true` - literal Boolean adevărat
* `type` - definește un alias de tip sau un tip asociat
* `union` - definește o [uniune][union]<!-- ignore -->; este un cuvânt cheie doar când este folosit într-o declarație de uniune
* `unsafe` - denotă cod, funcții, traits, sau implementări nesigure
* `use` - aduce simboluri în domeniul de vizibilitate
* `where` - denotă clauze care constrâng un tip
* `while` - buclă condiționată bazată pe rezultatul unei expresii

[union]: ../reference/items/unions.html

### Cuvinte cheie rezervate pentru utilizare viitoare

Următoarele cuvinte cheie nu au încă nicio funcționalitate, dar sunt rezervate de
Rust pentru o potențială utilizare în viitor.

* `abstract`
* `become`
* `box`
* `do`
* `final`
* `macro`
* `override`
* `priv`
* `try`
* `typeof`
* `unsized`
* `virtual`
* `yield`

### Identificatori bruți

*Identificatorii bruți* sunt sintaxa care îți permite să folosești cuvinte cheie acolo unde în mod normal nu ar fi permise. Folosești un identificator brut prin prefixarea unui cuvânt cheie cu `r#`.

De exemplu, `match` este un cuvânt cheie. Dacă încerci să compilezi următoarea funcție care folosește `match` ca nume al său:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

vei primi această eroare:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

Eroarea arată că nu poți folosi cuvântul cheie `match` ca identificator al funcției. Pentru a folosi `match` ca nume de funcție, trebuie să folosești sintaxa pentru identificatori bruți, în felul următor:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

Acest cod se va compila fără nicio eroare. Observă prefixul `r#` în numele funcției la definirea acesteia, cât și atunci când funcția este apelată în `main`.

Identificatorii bruți îți permit să folosești orice cuvânt alegi ca identificator, chiar dacă acel cuvânt se întâmplă să fie un cuvânt cheie rezervat. Acest lucru ne oferă mai multă libertate în alegerea numelor de identificatori, precum și ne permite să ne integrăm cu programe scrise într-un limbaj unde aceste cuvinte nu sunt cuvinte cheie. În plus, identificatorii bruți îți permit să folosești librării scrise într-o ediție Rust diferită față de crate-ul tău. De exemplu, `try` nu este un cuvânt cheie în ediția 2015, dar este în ediția 2018. Dacă depinzi de o librărie care este scrisă folosind ediția 2015 și are o funcție `try`, va trebui să folosești sintaxa pentru identificatori bruți, `r#try` în acest caz, pentru a apela acea funcție din codul tău din ediția 2018. Vezi [Anexa E][appendix-e]<!-- ignore --> pentru mai multe informații despre ediții.

[appendix-e]: appendix-05-editions.html