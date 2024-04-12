## Afișarea mesajelor de eroare pe eroare standard în loc de ieșire standard

În prezent, transmitem toate mesajele noastre către terminal folosind macro-ul `println!`. În majoritatea terminalelor există două tipuri de afișaj: *ieșirea standard* (`stdout`) pentru informații generale și *eroarea standard* (`stderr`) pentru mesajele de eroare. Acest lucru face posibilă alegerea de către utilizatori a redirecționării afișajului reușit al unui program spre un fișier, păstrând în același timp afișarea mesajelor de eroare pe ecran.

Macro-ul `println!` este capabil să afișeze doar pe ieșirea standard, așadar în scopul de a facilita afișarea pe eroarea standard, va trebui să apelăm la o altă metodă.

### Verificarea locului unde sunt scrise erorile

La început, să examinăm cum conținutul afișat de `minigrep` este redat în prezent pe ieșirea standard, și cum putem direcționa mesajele de eroare către eroarea standard în schimb. Facem acest lucru redirecționând fluxul de ieșire standard către un fișier și declanșăm intenționat o eroare. Fluxul de eroare standard nu va fi redirecționat, deci tot ceea ce este trimis către eroarea standard va apărea în continuare pe ecran.

Este așteptat ca programele de linie de comandă să expedieze mesajele de eroare către fluxul de eroare standard, permițându-ni să vedem aceste mesaje pe ecran chiar dacă ieșirea standard este trimisă către un fișier. În stadiul actual, programul nostru nu respectă această convenție: ne pregătim să constatăm că mesajele de eroare sunt salvate într-un fișier, nu afișate pe ecran!

Pentru a ilustra acest comportament, vom executa programul cu `>` urmat de numele fișierului, *output.txt*, către care dorim să redirecționăm ieșirea standard. Nu vom adăuga niciun argument pentru a induce o eroare:

```console
$ cargo run > output.txt
```

Sintaxa `>` instruiește shell-ul să trimită conținutul ieșirii standard în *output.txt* în loc să-l afișeze pe ecran. De vreme ce mesajul de eroare așteptat nu a fost vizibil pe ecran, presupunem că a fost redirecționat în fișier. Iată ce a ajuns în *output.txt*:

```text
Problem parsing arguments: not enough arguments
```

Așa este, mesajul de eroare se afișează pe ieșirea standard. Este preferabil ca astfel de mesaje să fie trimise către eroarea standard, astfel încât doar rezultatele corecte să poată fi salvate în fișier. Vom face schimbarea necesară.

### Afișarea erorilor pe eroarea standard

Vom utiliza codul din Listarea 12-24 pentru a schimba modul în care mesajele de eroare sunt afișate. Datorită refactoring-ului realizat anterior în acest capitol, toate porțiunile de cod ce afișează mesajele de eroare se găsesc într-o unică funcție, `main`. Macro-ul `eprintln!`, oferit de biblioteca standard, afișează către fluxul de eroare standard, așadar să modificăm cele două locuri unde apelam `println!` pentru a afișa erorile, utilizând în schimb `eprintln!`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

<span class="caption">Listarea 12-24: Scrierea mesajelor de eroare pe eroarea standard în locul ieșirii standard folosind `eprintln!`</span>

Acum să rulăm programul din nou în același mod, fără argumente și redirecționând ieșirea standard cu `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Observăm eroarea pe ecran și *output.txt* nu conține nimic, acesta fiind comportamentul așteptat de la programele de linie de comandă.

Să rulăm din nou programul cu argumente care nu provoacă o eroare, totuși redirecționăm ieșirea standard către un fișier, în felul următor:

```console
$ cargo run -- to poem.txt > output.txt
```

Nu vom avea parte de nicio afișare pe terminal, iar *output.txt* va conține rezultatele noastre:

<span class="filename">Numele fișierului: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Aceasta demonstrează că folosim acum ieșirea standard pentru afișajul succesului și eroarea standard pentru afișajul erorilor, așa cum este adecvat.

## Sumar

În acest capitol am revăzut unele dintre cele mai importante concepte pe care le-ai asimilat până în prezent și am discutat despre cum să executăm operațiuni I/O obișnuite în Rust. Prin utilizarea argumentelor de linie de comandă, fișierelor, variabilelor de mediu, și a macro-ului `eprintln!` pentru afișarea erorilor, ești acum echipat să creezi aplicații pentru linie de comandă. Îmbinând aceste cunoștințe cu conceptele prezentate în capitolele anterioare, codul tău va fi structurat corect, va stoca datele eficient în structurile de date potrivite, va gestiona erorile elegant și va beneficia de testare riguroasă.

În capitolul următor, ne vom aprofunda în explorarea unor funcționalități Rust inspirate din limbajele de programare funcționale: închiderile și iteratorii.
