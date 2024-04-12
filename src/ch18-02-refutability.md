## Refutabilitatea: când un pattern ar putea eșua la potrivire

Există două tipuri de pattern-uri: refutabile și irefutabile. Pattern-urile care se vor potrivi cu orice valoare posibilă sunt *irefutabile*. De exemplu, `x` din `let x = 5;` este irefutabil, deoarece `x` poate reprezenta orice valoare și astfel nu poate să nu se potrivească. Pattern-urile care ar putea să nu se potrivească pentru anumite valori sunt *refutabile*. Un astfel de exemplu este `Some(x)` din `if let Some(x) = a_value`, unde dacă `a_value` este `None` și nu `Some`, pattern-ul `Some(x)` va eșua să se potrivească.

Parametrii funcțiilor, instrucțiunile `let` și buclele `for` necesită pattern-uri irefutabile, deoarece programul nu are cum să procedeze în mod semnificativ atunci când valorile nu corespund. Expresiile `if let` și `while let` permit utilizarea ambelor tipuri de pattern-uri, însă compilatorul va avertiza împotriva folosirii pattern-urilor irefutabile, deoarece acestea sunt proiectate să gestioneze posibilitatea unui eșec.

De regulă, nu ar trebui să ne îngrijorăm despre diferența dintre pattern-urile refutabile și irefutabile; totuși, este important să înțelegem conceptul de refutabilitate pentru a putea răspunde corect atunci când întâmpinăm acești termeni în mesajele de eroare ale compilatorului. În astfel de situații, va trebui să ajustăm fie pattern-ul, fie contextul în care este folosit, în funcție de comportamentul dorit în cod.

Să examinăm un exemplu care ilustrează ce se întâmplă când încercăm să folosim un pattern refutabil în locul unuia irefutabil și invers. Listarea 18-8 ne arată o instrucțiune `let` unde am utilizat `Some(x)`, un pattern refutabil. După cum putem anticipa, codul acesta nu va fi compilat.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-08/src/main.rs:here}}

<span class="caption">Listarea 18-8: Tentativa de a folosi un pattern refutabil cu `let`</span>

Dacă `some_option_value` ar fi `None`, nu va corespunde cu pattern-ul `Some(x)`, indicând că pattern-ul este refutabil. Totuși, `let` necesită un pattern irefutabil pentru că, altfel, nu poate fi efectuată nici o operațiune validă cu valoarea `None`. La momentul compilării, Rust va indica eroarea de a încerca să folosim un pattern refutabil unde este cerut unul irefutabil:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-08/output.txt}}
```

Ne găsim în situația de a nu acoperi prin pattern-ul `Some(x)` toate valorile posibile, ceea ce conduce corect la o eroare de compilare în Rust.

Dacă avem un pattern refutabil unde se cere unul irefutabil, putem remedia situația modificând codul care îl folosește: în loc de `let`, putem opta pentru `if let`. Dacă pattern-ul nu se potrivește, codul va omite pur și simplu secțiunea din acolade, permițând continuarea executării în mod valid. Listarea 18-9 prezintă cum putem corecta codul din Listarea 18-8.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-09/src/main.rs:here}}
```

<span class="caption">Listarea 18-9: Utilizarea `if let` și a unui bloc cu pattern-uri refutabile în loc de `let`</span>

Acum codul are o cale de ieșire! Această variantă este complet validă, chiar dacă înseamnă că nu putem folosi un pattern irefutabil fără a avea parte de o eroare. În cazul în care `if let` primește un pattern care va coincide mereu, cum ar fi `x` din Listarea 18-10, compilatorul va semnala un avertisment.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-10/src/main.rs:here}}
```

<span class="caption">Listarea 18-10: Încercarea de folosire a unui pattern irrefutabil cu `if let`</span>

Rust sugerează că nu este logic să utilizăm `if let` cu un pattern care nu poate eșua:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-10/output.txt}}
```

Din acest motiv, în instrucțiunea `match`, trebuie să folosim pattern-uri refutabile, cu excepția ultimului caz, care ar trebui să acopere orice valori rămase cu un pattern irefutabil. Deși Rust permite utilizarea unui pattern irefutabil într-un `match` cu un singur caz, această sintaxă nu este foarte practică și ar putea fi simplificată prin folosirea unei instrucțiuni `let` mai directe.

Înțelegând unde și cum să folosim pattern-urile, cât și distincția dintre cele refutabile și irefutabile, să explorăm acum toată sintaxa disponibilă pentru crearea pattern-urilor.
