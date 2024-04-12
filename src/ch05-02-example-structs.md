## Un program exemplu ce utilizează structurile

Pentru a înțelege când și cum ne-ar fi util să utilizăm structurile, propunem să dezvoltăm împreună un program care calculează aria unui dreptunghi. Vom începe construind acest program cu ajutorul unor variabile simple și apoi ne vom concentra pe îmbunătățirea și refactorizarea lui prin utilizarea structurilor.

Să creăm un nou proiect binar cu Cargo, numit *rectangles*. Scopul acestui program va fi de a prelua lățimea și înălțimea unui dreptunghi, specificate în pixeli, și de a calcula aria acestuia. Listarea 5-8 ne arată un exemplu de program scurt care realizează exact acest lucru, modelul fiind implementat în fișierul *src/main.rs* al proiectului nostru.

<span class="filename">Numele fișierului:: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

<span class="caption">Listarea 5-8: Calculul ariei unui dreptunghi, definit prin variabile separate pentru lățime și înălțime</span>

Acum, execută acest program utilizând `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Deși codul calculează corect aria dreptunghiului, invocând funcția `area` cu fiecare dimensiune, acest lucru poate fi îmbunătățit pentru a crea un cod mai clar și mai lizibil.

Neclaritatea acestui cod devine evidentă atunci când ne uităm la semnătura funcției `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

Funcția `area` este destinată calculării ariei unui dreptunghi. Totuși, funcția pe care noi am redactat-o are doi parametri, și în niciun loc din programul nostru nu se menționează explicit faptul că acești parametri sunt interconectați. Ameliorarea lizibilității și gestionării codului s-ar putea realiza prin gruparea înălțimii și lățimii. Un astfel de procedeu a fost deja discutat în secțiunea [“Tipul Tuplă”][the-tuple-type]<!-- ignore -->, din Capitolul 3, prin utilizarea tuplelor.

### Refactorizarea prin utilizarea tuplelor

Lista 5-9 ilustrează o nouă variantă a programului nostru, în care am integrat utilizarea tuplelor.

<span class="filename">Numele fișierului:: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

<span class="caption">Listarea 5-9: Definirea lățimii și înălțimii dreptunghiului prin intermediul unei tuple</span>

Acest program, privit dintr-o anumită perspectivă, este mai eficient. Prin utilizarea tuplelor, adăugăm o structură și trimitem un singur argument. Totuși, acestă abordare este mai puțin clară: elementele unei tuple nu sunt denumite, astfel că trebuie să indexăm părțile tuplei, ceea ce face ca procesul de calcul să devină mai greu de înțeles.

Dacă am confunda lățimea cu înălțimea nu ar avea un impact semnificativ asupra calculării ariei, dar în cazul în care am dori să reprezentăm dreptunghiul pe un ecran, atunci acest lucru ar conta! Ar trebui să ne amintim că `width` (lățimea) este indexul 0 al tuplei, iar `height` (înălțimea) este indexul 1. Aceasta ar putea fi un lucru greu de înțeles și de reținut de către o altă persoană care ar urma să utilizeze codul nostru. Deoarece nu am transmis semnificația datelor în codul nostru, introducerea erorilor devine acum mult mai ușoară.

### Refactorizarea folosind structuri: Introducerea unui sens mai amplu

Folosim structurile pentru a adăuga mai mult sens datelor prin etichetarea acestora. Avem posibilitatea de a transforma tupla pe care o utilizăm într-o structură, acordând un nume atât pentru întreg, cât și pentru părțile acesteia. Acest lucru este ilustrat în Listarea 5-10.

<span class="filename">Numele fișierului:: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

<span class="caption">Listarea 5-10: Crearea structurii `Rectangle`</span>

În acest fragment de cod, am definit o structură numită `Rectangle`. În cadrul acoladelor, am precizat câmpurile acesteia: `width` (lăţime) şi `height` (înălţime), ambele de tipul `u32`. Apoi, în funcția `main`, am inițiat o instanță specifică `Rectangle` cu o lăţime de `30` şi o înălţime de `50`.

Acum, funcţia noastră `area` este definită având un singur parametru numit `rectangle`, de tip împrumut imutabil al unei instanţe `Rectangle`. După cum am menţionat anterior în Capitolul 4, dorim să împrumutăm structura mai degrabă decât să îi preluăm total posesiunea. Astfel, funcţia `main` îşi păstrează posesiunea şi poate continua să utilizeze `rect1`, motiv pentru care utilizăm `&` în semnătura funcţiei şi la apelarea acesteia.

Funcţia `area` accesează câmpurile `width` şi `height` ale instanţei `Rectangle`. Notăm că accesul la câmpurile unei instanţe împrumutate nu permută valorile acestor câmpuri, acesta fiind motivul pentru care în cod Rust deseori vedem structuri anume împrumutate. Semnătura actuală a funcţiei `area` exprimă în mod precis intenţia noastră: de a calcula aria lui `Rectangle` utilizând câmpurile sale `width` și `height`. Acest lucru subliniază relaţia dintre lăţime şi înălţime şi le conferă nume descriptive, în locul folosirii valorilor de index ale tuplei `0` şi `1`, implicând un câştig semnificativ în ceea ce priveşte claritatea.

### Îmbunătățirea funcționalității prin folosirea de trăsături derivate

Ne-ar fi de folos să putem afișa o instanță a `Rectangle` în timp ce depanăm programul, vizualizând valorile tuturor câmpurilor sale. În Listarea 5-11, încercăm să utilizăm macro-ul [`println!`][println], așa cum am făcut în capitolele anterioare. Totuși, aceasta nu va funcționa.

<span class="filename">Numele fișierului:: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

<span class="caption">Listare 5-11: Tentativa de a afișa o instanță de `Rectangle`</span>

Atunci când rulăm procesul de compilare pentru acest cod, obținem o eroare ce conține următorul mesaj principal:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

Macroul `println!` are capacitatea de a realiza diferite formate de afișare. În mod implicit, acoladele indică faptul că `println!` ar trebui să utilizeze un tip de formatare numit `Display`, care reprezintă o ieșire destinată direct consumului de către utilizatorul final. Tipurile primitive pe care le-am întâlnit până acum implementează `Display` în mod automat, deoarece există o singură modalitate în care ai dori să arăți un `1` sau orice alt tip primitiv unui utilizator. Însă, în cazul structurilor, modul în care `println!` ar trebuie să formateze ieșirea este mai puțin evident, existând numeroase variante de afișare: vrei să se utilizeze virgule sau nu? Dorești ca acoladele să fie afișate? Ar trebui toate câmpurile să fie vizibile? Din cauza acestor incertitudini, Rust nu încearcă să presupună intențiile noastre, astfel că structurile nu au o implementare predefinită a `Display` pentru utilizare cu `println!` și substituentul `{}`.

Dacă vom urmări în continuare mesajele de eroare, vom descoperi o notă foarte utilă:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Să încercăm acest lucru! Acum, apelul macro `println!` ar urma să arate astfel: `println!("rect1 este {:?}", rect1);`. Prin adăugarea specificatorului `:?` în interiorul acoladelor, îi indicăm macro-ului `println!` că dorim să utilizăm un format de afișare numit `Debug`. Trăsătura `Debug` ne oferă capacitatea de a afișa structura noastră într-un mod care este eficient pentru dezvoltatori, permițându-ne să vizualizăm valoarea acesteia în timp ce lucrăm la depanarea codului.

Încearcă să compilezi codul cu această modificare. Drăcie! Încă o eroare:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Dar, din nou, compilatorul ne oferă o sugestie utilă:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust dispune de funcționalitatea de a afișa informații pentru depanare, dar pentru a face această funcționalitate disponibilă structurii noastre, trebuie să optăm în mod explicit. Putem face asta prin adăugarea atributului extern `#[derive(Debug)]` imediat înainte de definiția structurii, așa cum este ilustrat în Listarea 5-12.

<span class="filename">Numele fișierului:: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

<span class="caption">Listarea 5-12: Incorporarea atributului pentru a deriva trăsătura `Debug` și afișarea instanței `Rectangle` prin intermediul formatării de depanare</span>

La rularea programului, acum nu vom întâlni nici o eroare, iar rezultatul arată astfel:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Excelent! Rezultatul nu este cel mai estetic, dar afișează valorile tuturor câmpurilor pentru instanța dată, ceea ce cu siguranță ne este de folos în timpul depanării. Când lucrăm cu structuri mai complexe, este util să obținem un rezultat mai ușor de analizat; în acele cazuri, putem aplica `{:#?}` în loc de `{:?}` in interiorul string-ului `println!`. De exemplu, folosirea stilului `{:#?}` va produce următorul rezultat:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Un alt mod de a afișa o valoare utilizând formatul `Debug` este prin intermediul macro-ului [`dbg!`][dbg]<!-- ignore -->. Acesta preia în posesiune o anumită expresie (în contradicție cu `println!`, care folosește o referință), afișează fișierul și linia de cod unde macro-ul `dbg!` este apelat în programul tău, împreună cu rezultatul acelei expresii, după care restituie posesiunea valorii.

> Notă: Apelarea macro-ului `dbg!` afișează informațiile în fluxul standard de erori (`stderr`), spre deosebire de `println!`, care afișează în fluxul standard de ieșire (`stdout`). Vom vorbi mai mult despre `stderr` și `stdout` în secțiunea [„Scrierea mesajelor de eroare la fluxul standard de erori în loc de fluxul standard de ieșire” din Capitolul 12][err]<!-- ignore -->.

Iată un exemplu în care ne interesează valoarea atribuită câmpului `width`, precum și valoarea întregii structuri numită `rect1`:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Putem înconjura expresia `30 * scale` cu `dbg!` și, din moment ce `dbg!` ne redă posesiunea valorii expresiei, câmpul `width` va primi aceeași valoare ca și în cazul în care nu am fi apelat `dbg!`. Nu dorim ca `dbg!` să preia posesiunea asupra `rect1`, astfel încât utilizăm o referință la `rect1` în următorul apel. Iată cum se prezintă rezultatul acestui exemplu:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Putem observa că primul fragment de output provine de la *src/main.rs*, linia 10, unde depanăm expresia `30 * scale`. Valoarea rezultată este `60`, având în vedere că formatarea `Debug` pentru numerele întregi este realizată prin afișarea exclusivă a valorilor acestora. Apelul `dbg!` de pe linia 14 din *src/main.rs* afișează valoarea `&rect1`, care reprezintă structura `Rectangle`. Acest output utilizează o formatare frumoasă, `Debug`, specifice tipului `Rectangle`. Macro-ul `dbg!` poate fi foarte util atunci când încerci să înțelegi ce face codul tău!

Pe lângă trăsătură `Debug`, Rust ne oferă numeroase trăsături pe care le putem folosi cu atributul `derive`. Acestea pot adăuga comportamente utile tipurilor noastre. Trăsăturile date și comportamentele lor se găsesc în [Anexa C][app-c]<!-- 
ignore -->. Vom discuta în Capitolul 10 modul de implementare a acestor trăsături cu funcționalitate particularizată, dar și modul de creare a propriilor trăsături. De asemenea, există și alte atribute în afară de `derive`; pentru mai multe informații, vezi [secțiunea "Atribute"][attributes].

Funcția noastră `area` este foarte specifică. Aceasta calculează doar ariile dreptunghiurilor. Ar fi benefic să legăm mai strâns acest comportament de structura noastră `Rectangle`, având în vedere faptul că nu funcționează cu niciun alt tip. Să vedem cum putem continua să îmbunătățim acest cod, transformând funcția `area` într-o *metodă* `area` definită pe tipul nostru `Rectangle`.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
