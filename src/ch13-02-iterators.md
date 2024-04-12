## Procesarea unei serii de elemente cu ajutorul iteratorilor

Modelul iterator permite efectuarea unei anumite sarcini pe o secvență de elemente, unul câte unul. Un iterator este responsabil de logica parcurgerii fiecărui element și stabilirea momentului în care secvența este completă. Când folosești iteratori, nu este necesar să reimplementezi tu această logică.

În Rust, iteratorii sunt *indolenți* (lazy), adică nu produc niciun efect până nu apelezi metode care consumă iteratorul pentru utilizare. De exemplu, codul din Listarea 13-10 creează un iterator pentru elementele din vectorul `v1` prin apelarea metodei `iter`, definită pe `Vec<T>`. Acest cod în sine nu realizează nimic util.

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

<span class="caption">Listarea 13-10: Crearea unui iterator</span>

În variabila `v1_iter` este stocat iteratorul. După ce un iterator a fost creat, putem să-l utilizăm în diferite moduri. În Listarea 3-5 din Capitolul 3, am itinerat peste un array folosind o buclă `for` pentru a executa un cod pe fiecare element. În realitate, acest lucru a creat și consumat implicit un iterator, dar nu am detaliat cum funcționează acest proces până acum.

În exemplul din Listarea 13-11, crearea iteratorului este separată de utilizarea lui în bucla `for`. Când bucla `for` folosește iteratorul `v1_iter`, fiecare element din iterator participă la o iterație a buclei, imprimând astfel fiecare valoare.

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

<span class="caption">Listarea 13-11: Utilizarea unui iterator într-o buclă `for`</span>

În limbajele de programare care nu dispun de iteratori în bibliotecile lor standard, probabil ai scrie aceeași funcționalitate inițializând o variabilă la indexul 0, utilizând această variabilă pentru a accesa elementele din vector și incrementând valoarea variabilei într-o buclă până la atingerea numărului total de elemente din vector.

Iteratorii se ocupă de toată această logică pentru tine, minimizând codul repetitiv care ar putea fi și greșit. Aceștia oferă o flexibilitate sporită pentru a aplica aceeași logică la diferite tipuri de secvențe, nu doar la structuri de date indexabile, precum vectorii. Acum să explorăm cum realizează iteratorii acest lucru.

### Trăsătura `Iterator` și metoda `next`

Toți iteratorii implementează o trăsătură denumită `Iterator` definită în biblioteca standard. Definiția acestei trăsături este următoarea:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // metodele cu implementări predefinite au fost omise
}
```

Această definiție introduce o sintaxă nouă: `type Item` și `Self::Item`, care stabilește un *tip asociat* trăsăturii. Vom aborda tipurile asociate în detaliu în Capitolul 19. Deocamdată, e important să știm că, pentru a implementa trăsătura `Iterator`, trebuie definit și un tip `Item`, care este utilizat în tipul de retur al metodei `next`, adică tipul `Item` va fi tipul de date returnat de iterator.

Trăsătura `Iterator` necesită de la cei ce o implementează să definească o singură metodă: `next`, care returnează câte un element al iteratorului învelit în `Some`, iar când iterația s-a încheiat, returnează `None`.

Metoda `next` poate fi apelată direct pe iteratori; Listarea 13-12 ilustrează valorile returnate de apeluri multiple la `next` pe iteratorul creat din vector.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

<span class="caption">Listarea 13-12: Apelarea metodei `next` pe un iterator</span>

Trebuie să remarcăm că a fost necesar să facem `v1_iter` mutabil: invocarea metodei `next` pe un iterator modifică starea internă pe care acesta o utilizează pentru a urmări poziția curentă în secvență. Practic, acest cod *consumă* iteratorul. Fiecare chemare a lui `next` consumă un element din iterator. Când folosim o buclă `for`, nu este nevoie să facem `v1_iter` mutabil deoarece bucla preia posesiunea lui `v1_iter` și îl face mutabil în mod implicit.

Mai trebuie să observăm că valorile obținute din apelurile la `next` sunt referințe imutabile la elementele din vector. Metoda `iter` creează un iterator de referințe imutabile. Dacă vrem un iterator care să preia posesiunea `v1` și să returneze valori proprii, putem folosi metoda `into_iter` în loc de `iter`. În mod similar, pentru a itera peste referințe mutabile, putem utiliza `iter_mut` în locul lui `iter`.

### Metode care consumă iteratorul

Trăsătura `Iterator` include o serie de metode diferite cu implementări default furnizate de biblioteca standard; poți afla mai multe despre aceste metode consultând documentația API pentru trăsătura `Iterator`. Câteva dintre aceste metode folosesc `next` în definiția lor, de aceea este necesar să implementezi metoda `next` când realizezi implementarea trăsăturii `Iterator`.

Metodele care invocă `next` sunt denumite *adaptoare consumatoare*, deoarece utilizarea lor epuizează iteratorul. Un exemplu este metoda `sum`, care își asumă posesiunea iteratorului și parcurge elementele prin apeluri repetate ale `next`, consumând în acest fel iteratorul. În timpul iterării, adaugă fiecare element la un total acumulat și returnează acest total la finalizarea iterației. Listarea 13-13 prezintă un test care ilustrează folosirea metodei `sum`:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

<span class="caption">Listarea 13-13: Utilizarea metodei `sum` pentru a calcula suma totală a elementelor din iterator</span>

Nu ne este permis să utilizăm `v1_iter` după apelul la `sum`, deoarece `sum` preia posesiunea iteratorului pe care îl apelăm.

### Metode ce generează alți iteratori

*Adaptoarele de iterator* sunt metode definite pe trăsătura `Iterator` care nu consumă iteratorul. În loc de asta, ele generează alți iteratori modificând anumite aspecte ale iteratorului original.

Listarea 13-14 prezintă un exemplu de utilizare a metodei adaptor de iterator `map`, care primește o închidere ce este apelată pentru fiecare element în timp ce sunt iterați. Metoda `map` returnează un nou iterator ce produce elementele modificate. În acest caz, închiderea creează un nou iterator unde fiecare element din vector va fi mărit cu o unitate:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

<span class="caption">Listarea 13-14: Utilizarea adaptorului de iterator `map` pentru a genera un nou iterator</span>

Cu toate acestea, acest cod generează un avertisment:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Codul din Listarea 13-14 de fapt nu realizează nimic; închiderea pe care am specificat-o nu este invocată niciodată. Avertismentul ne reamintește motivul: adaptoarele de iterator sunt evaluate indolent și trebuie să consumăm iteratorul aici.

Pentru a corecta acest avertisment și a consuma iteratorul, ne vom folosi de metoda `collect`, aceeași pe care am utilizat-o în Capitolul 12 cu `env::args` în Listarea 12-1. Această metodă consumă iteratorul și colectează valorile obținute într-o structură de tip colecție.

În Listarea 13-15, noi colectăm rezultatele iterației peste noul iterator rezultat din apelul la `map` într-un vector. Acest vector va conține în final fiecare element din vectorul inițial incrementat cu 1.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

<span class="caption">Listarea 13-15: Utilizarea metodei `map` pentru a genera un nou iterator și apoi utilizarea metodei `collect` pentru a consuma acest iterator nou și a crea un vector</span>

Deoarece `map` primește o închidere, putem defini orice operație dorim să efectuăm asupra fiecărui element. Aceasta este o ilustrație excelentă a modului în care închiderile îți permit să personalizezi anumite comportamente în timp ce refolosești comportamentul de iterație pe care trăsătura `Iterator` îl furnizează.

Putem combina mai multe apeluri către adaptoarele de iteratori pentru a efectua acțiuni complexe într-un mod inteligibil. Totuși, fiindcă toți iteratorii sunt evaluați indolent, e necesar să folosim una dintre metodele adaptoare consumatoare pentru a obține rezultate din apelurile către adaptoare de iteratori.

### Folosirea închiderilor pentru captarea variabilelor din context

Adaptoarele de iteratori folosesc des închideri ca parametri. De obicei, închiderile pe care urmează să le specificăm pentru aceste adaptoare sunt cele care captează variabilele din contextul lor.

În exemplul nostru, vom apela la metoda `filter`, care acceptă o închidere. Aceasta primește fiecare element din iterator și returnează un `bool`. Dacă închiderea evaluează la `true`, elementul va fi inclus în seria de date generată de `filter`. Pe de altă parte, dacă închiderea evaluează la `false`, elementul nu va fi inclus.

În Listarea 13-16, folosim `filter` împreună cu o închidere care include variabila `shoe_size` din context, pentru a itera prin colecția de structuri `Shoe`, furnizând doar pantofii care au o anumită mărime.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

<span class="caption">Listarea 13-16: Utilizarea metodei `filter` cu o închidere care capturează variabila `shoe_size`</span>

Funcția `shoes_in_size` preia un vector de pantofi și o mărime a pantofului ca parametri. Ea returnează un vector care cuprinde exclusiv pantofi cu mărimea specificată.

În corpul funcției `shoes_in_size`, creăm un iterator prin apelarea `into_iter`, care preia vectorul. Următorul pas este aplicarea `filter` la acest iterator, transformându-l într-un nou iterator care va conține doar elementele pentru care închiderea evaluează la `true`.

Închiderea folosită captează parametrul `shoe_size` din context și îl compară cu mărimea fiecărui pantof, selectând doar pantofii care conform cu măsura specificată. Finalizând cu `collect`, agregăm valorile filtrate de iterator într-un vector care este apoi returnat de funcție.

Testul nostru confirmă că, după apelarea funcției `shoes_in_size`, rezultatul conține doar pantofii având mărimea egală cu cea specificată inițial.
