## Control concis al executării cu `if let`

Sintaxa `if let` îți oferă posibilitatea de a combina `if` și `let` pentru a manipula, într-un mod mai concis, valorile care corespund unui anumit șablon, ignorând în același timp restul. Ia în considerare programul din Listarea 6-6, care creează o potrivire pentru o valoare `Option<u8>` în variabila `config_max`, dar care intenționează să execute codul doar dacă valoarea este variantă `Some`.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

<span class="caption">Listarea 6-6: Un `match` care este interesat doar de execuția
codului atunci când valoarea este `Some`</span>

Dacă valoarea este `Some`, afișăm valoarea din variantă `Some` prin asocierea acesteia cu variabila `max` în șablon. Nu dorim să facem nimic cu valoarea `None`. Pentru a respecta cerințele expresiei `match`, suntem obligați să adăugăm `_ => ()` după procesarea unei singure variante, ceea ce des reprezintă un cod suplimentar, inutil și iritant.

În loc să abordăm problema în modul complicat, putem simplifica procesul folosind `if let`. Următorul fragment de cod funcționează identic cu instrucțiunea `match` prezentată în Listarea 6-6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

Sintaxa `if let` preia un șablon și o expresie, separate de un semn de egalitate. Funcționează în mod similar cu `match`, unde expresia este atribuită instrucțiunii `match`, iar șablonul constituie prima sa ramură. În acest caz, șablonul este `Some(max)`, iar `max` se leagă de valoarea din interiorul `Some`. Apoi, putem utiliza `max` în corpul blocului `if let`, la fel cum am folosit `max` în ramura corespunzătoare a instrucțiunii `match`. Codul din blocul `if let` nu se execută dacă valoarea nu corespunde șablonului.

Folosind `if let`, vor rezulta mai puține caractere introduse, mai puțină indentare și mai puțin cod redundant. Totuși, pierdem verificarea completă pe care instrucțiunea `match` o asigură. Alegerea între `match` și `if let` depinde de natura situației cu care te confrunți și dacă economisirea spațiului este un compromis acceptabil pentru a nu realiza o verificare exhaustivă.

Cu alte cuvinte, poți percepe `if let` drept un zahăr sintactic pentru un `match` care rulează codul atunci când valoarea corespunde cu un anumit șablon, ignorând toate celelalte valori.

Avem posibilitatea de a include un `else` în structura `if let`. Blocul de cod alăturat lui `else` este identic cu cel care ar apărea în cazul `_` în expresia `match`, expresie care este echivalentă cu structura `if let` și `else`. Acum să-ți reamintim de definiția enumerării `Coin` din listarea 6-4, unde varianta `Quarter` deține, de asemenea, o valoare `UsState`. Dacă vom dori să numărăm toate monedele care nu sunt de tipul quarter, dar și să anunțăm starea acestora, putem realiza acest lucru cu ajutorul unei expresii `match`, astfel:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

Sau am putea folosi o expresie `if let` și `else`, în felul următor:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

Dacă te afli în situația în care logica din programul tău este prea complexă pentru a fi exprimată folosind un `match`, nu uita că `if let` este și el un instrument util în limbajul Rust.

## Sumar

Am tratat felul în care putem folosi enumerările pentru a crea tipuri personalizate care pot avea una dintre o mulțime de valori enumerate. Am evidențiat felul în care tipul `Option<T>` din biblioteca standard ne ajută a preveni erorile prin folosirea sistemului de tipuri. Atunci când valorile enumerărilor conțin date, putem utiliza `match` sau `if let` pentru a extrage și folosi acele valori, în funcție de numărul de cazuri pe care trebuie să le gestionăm.

Acum, programele tale Rust sunt capabile să exprime conceptele specifice domeniului tău folosind structuri și enumerări. Crearea de tipuri personalizate pentru utilizare în API-ul tău garantează securitatea tipurilor: compilatorul se va asigura că funcțiile tale primesc doar valori ale tipului așteptat de fiecare funcție.

Cu scopul de a oferi utilizatorilor tăi un API bine structurat, ușor de utilizat și care expune doar ceea ce aceștia realmente necesită, să ne focusăm acum pe modulele din Rust.

