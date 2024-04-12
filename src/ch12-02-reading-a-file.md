## Citirea fișierului

Acum vom adăuga funcționalitatea de citire a fișierului specificat în argumentul `file_path`. Mai întâi, avem nevoie de un exemplu de fișier pentru a testa: vom folosi un fișier cu o mică cantitate de text distribuit pe mai multe linii și cu anumite cuvinte repetate. Listarea 12-3 ne prezintă o poezie de Emily Dickinson care este foarte potrivită! Creează un fișier cu numele *poem.txt* la nivelul rădăcină al proiectului tău și include poezia “I’m Nobody! Who are you?”

<span class="filename">Numele fișierului: poem.txt</span>

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

<span class="caption">Listarea 12-3: O poezie de Emily Dickinson, un excelent caz de test</span>

Cu textul prezent, editează *src/main.rs* și adaugă codul pentru citirea fișierului, după cum e prezentat în Listarea 12-4.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

<span class="caption">Listarea 12-4: Citirea conținutului fișierului specificat de al doilea argument</span>

Inițial, introducem o parte relevantă din biblioteca standard cu instrucțiunea `use`: ne este necesar `std::fs` pentru gestionarea fișierelor.

În `main`, noua expresie `fs::read_to_string` utilizează `file_path`, deschide fișierul respectiv și returnează un `std::io::Result<String>` cu conținutul fișierului.

Ulterior, adăugăm încă o instrucțiune `println!` temporară pentru a afișa valoarea variabilei `contents` după ce fișierul a fost citit, pentru a verifica funcționalitatea codului până în acest moment.

Execută acest cod folosind orice șir de caractere ca prim argument al liniei de comandă (pentru că încă nu am implementat partea de căutare) și fișierul *poem.txt* ca al doilea argument:

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

Excelent! Codul a accesat și a afișat conținutul fișierului. Totuși, codul prezintă câteva slăbiciuni. În prezent, funcția `main` are responsabilități multiple: în general, funcțiile sunt mai clare și mai simplu de întreținut dacă fiecare este responsabilă doar pentru un singur concept. O altă problemă este gestionarea ineficientă a erorilor. Programul este mic acum, așa că aceste deficiențe nu reprezintă o problemă majoră, dar pe măsură ce programul se mărește, acestea vor fi mai dificil de rezolvat într-un mod clar. Începerea timpurie a refactoring-ului (reorganizarea codului) în timpul dezvoltării unui program este o practică bună, deoarece refacerea codului este mult mai simplă când porțiunile sunt mai mici. Asta și vom face în continuare.
