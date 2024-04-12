## Acceptarea argumentelor liniei de comandă

Să creăm un nou proiect folosind, ca întotdeauna, `cargo new`. Vom numi proiectul nostru `minigrep` pentru a-l diferenția de instrumentul `grep` pe care s-ar putea să îl ai deja pe sistemul tău.

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

Prima sarcină este să configurăm `minigrep` să accepte cei doi parametri de linie de comandă: calea fișierului și un string pentru căutare. Astfel, dorim să putem executa programul nostru cu `cargo run`, urmat de două liniuțe pentru a indica faptul că argumentele ulterioare sunt destinate programului nostru și nu pentru `cargo`, un string de căutat, și o cale către un fișier în care să efectuăm căutarea, ca în exemplul următor:

```console
$ cargo run -- searchstring example-filename.txt
```

Deocamdată, programul generat de `cargo new` nu este capabil să proceseze argumentele pe care i le transmitem. Există biblioteci disponibile pe [crates.io](https://crates.io/) care ne-ar putea facilita scrierea unui program capabil să accepte argumente din linia de comandă, dar deoarece suntem în proces de învățare a acestui concept, să încercăm să implementăm această funcționalitate pe cont propriu.

### Citirea valorilor argumentelor

Pentru ca `minigrep` să poată citi valorile argumentelor de pe linia de comandă, e necesar să folosim funcția `std::env::args` din biblioteca standard a Rust. Aceasta returnează un iterator cu argumentele de linia de comandă primite de `minigrep`. Detalii complete despre iteratori vor fi disponibile în [Capitolul 13][ch13]<!-- ignore -->. În prezent, trebuie să reținem două aspecte despre iteratori: aceștia generează o serie de valori și putem folosi metoda `collect` pentru a transforma iteratorul într-o colecție, cum ar fi un array, care include toate elementele produse de acesta.

Codul din Listarea 12-1 îi va permite programului `minigrep` să citească argumentele de pe linia de comandă care îi sunt transmise și să le transfere într-un array.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

<span class="caption">Listarea 12-1: Colectarea argumentelor transmise pe linia de comandă într-un array și afișarea acestora</span>

Înainte de toate, importăm modulul `std::env` în contextul nostru cu ajutorul unei instrucțiuni `use`, pentru ca să putem utiliza funcția `args` din acesta. Se remarcă faptul că funcția `std::env::args` se află în cadrul a două module. Așa cum s-a discutat în [Capitolul 7][ch7-idiomatic-use]<!-- ignore -->, atunci când funcția dorită este situată în mai multe module, optăm să importăm modulul părinte în locul unei funcții anume. Prin această abordare, putem accesa alte funcții din `std::env` cu mai multă ușurință. De asemenea, evităm ambiguitatea care ar apărea prin adăugarea `use std::env::args` și apoi apelarea funcției simplu cu `args`, ceea ce ar putea fi confundat cu o funcție definită local.

> ### Funcția `args` și Unicode invalid
>
> Este important de reținut că `std::env::args` va genera panică dacă orice
> argument conține Unicode invalid. Dacă este necesar ca programul tău să
> accepte argumente care includ Unicode invalid, atunci ar trebui utilizată
> funcția `std::env::args_os`. Această funcție oferă un iterator ce generează
> valori de tip `OsString` în loc de `String`. Am optat pentru utilizarea lui
> `std::env::args` aici datorită simplității, deoarece valorile de tip
> `OsString` variază în funcție de platformă și sunt mai complicate de
> manevrat decât valorile de tip `String`.

La prima linie a funcției `main`, invocăm `env::args`, apoi utilizăm imediat `collect` pentru a converti iteratorul într-un vector care conține toate valorile generate de iterator. Putem apela funcția `collect` pentru a crea diferite tipuri de colecții, motiv pentru care specificăm în mod explicit tipul pentru `args`, indicând că dorim un vector de string-uri. Chiar dacă în Rust rar este necesar să adnotăm tipurile, când vine vorba de `collect`, adesea este nevoie de acest lucru deoarece aici Rust nu îți poate infera tipul de colecție dorit.

În final, afișăm vectorul folosind macro-ul de debug. Să încercăm să executăm codul mai întâi fără argumente, apoi cu două argumente:

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Observăm că prima valoare din vector este `"target/debug/minigrep"`, numele executabilului nostru. Aceasta este în acord cu comportamentul listei de argumente din C, care permite programelor să utilizeze numele sub care au fost invocate în cursul execuției. Oferirea accesului la numele programului poate fi utilă dacă dorim să îl afișăm în mesaje sau să modificăm comportamentul programului în funcție de aliasul de la linia de comandă folosit pentru invocare. Totuși, pentru nevoile acestui capitol, vom ignora acest aspect și vom salva doar cele două argumente necesare.

### Salvând valorile argumentelor în variabile

Programul nostru este acum capabil să acceseze valorile specificate ca argumente ale liniei de comandă. Trebuie să salvăm acum valorile celor doi argumente în variabile, pentru a putea utiliza aceste valori pe parcursul restului programului. Această etapă este realizată în Listarea 12-2.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

<span class="caption">Listarea 12-2: Creăm variabile pentru memorarea argumentului de căutare și argumentului cu calea fișierului</span>

Așa cum am observat când am afișat vectorul, numele programului este stocat în prima valoare a vectorului la `args[0]`, deci ne apucăm să lucram cu argumentele începând de la indexul `1`. Primul argument pe care `minigrep` îl solicită este string-ul pe care dorim să îl căutăm, așa că atribuim o referință către primul argument variabilei `query`. Pentru al doilea argument, care reprezintă calea fișierului, atribuim o referință către al doilea argument variabilei `file_path`.

Printăm temporar valorile acestor variabile pentru a ne asigura că programul funcționează conform intențiilor noastre. Să executăm acest program din nou, de data aceasta cu argumentele `test` și `sample.txt`:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Minunat, programul funcționează corect! Valorile argumentelor necesare sunt acum salvate în variabilele potrivite. Ulterior, vom adăuga gestionarea erorilor pentru a face față situațiilor eronate, cum ar fi cazul în care utilizatorul nu furnizează niciun argument; deocamdată, vom trece peste astfel de situații și ne vom concentra asupra implementării funcționalității de citire a fișierelor.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
