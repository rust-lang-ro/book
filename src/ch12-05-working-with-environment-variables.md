## Interacționând cu variabilele de mediu

Vom îmbunătăți `minigrep` prin adăugarea unei opțiuni noi: căutarea care nu face distincție între majuscule și minuscule, ce poate fi activată prin intermediul unei variabile de mediu. Deși această caracteristică ar putea fi oferită ca o opțiune de linie de comandă, ceea ce ar impune utilizatorilor să o introducă de fiecare dată, alegerea de a o configura ca variabilă de mediu le permite acestora să o seteze o singură dată, astfel toate căutările lor în cadrul sesiunii curente de terminal să fie insesibilă la majuscule.

### Scrierea unui test nereușit pentru funcția `search_case_insensitive`

Adăugăm mai întâi o nouă funcție `search_case_insensitive` care va fi apelată atunci când variabila de mediu deține o valoare. Continuăm să urmăm procesul TDD, așadar primul pas este, din nou, să scriem un test nereușit. Vom adăuga un nou test pentru funcția `search_case_insensitive` și vom redenumi testul vechi din `one_result` în `case_sensitive` pentru a face distincția clară între cele două teste, după cum se arată în Listarea 12-20.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

<span class="caption">Listarea 12-20: Adăugarea unui test nou nereușit pentru funcția de căutare insensibilă la majuscule pe care urmează să o implementăm</span>

Observați că am editat și conținutul testului anterior. Am introdus o linie nouă cu textul `"Duct tape."` care utilizează un D majuscul și care nu ar trebui să corespundă cu interogarea `"duct"` când efectuăm căutarea în mod sensibil la majuscule. Această modificare a testului anterior ajută la asigurarea faptului că nu stricăm din greșeală funcționalitatea de căutare sensibilă la majuscule pe care am realizat-o deja. Testul actual ar trebui să fie valid acum și ar trebui să rămână așa pe măsură ce dezvoltăm căutarea insensibilă la majuscule.

Testul nou pentru căutarea *insensibilă* la majuscule folosește interogarea `"rUsT"`. În funcția `search_case_insensitive` pe care o vom adăuga în curând, interogarea `"rUsT"` ar trebui să găsească linia ce conține `"Rust:"` cu un R majuscul și să identifice și linia `"Trust me."`, deși ambele au diferite utilizări de majuscule față de interogare. Aceasta este definiția testului nereușit, care va eșua în compilare deoarece nu am definit încă funcția `search_case_insensitive`. Nu vă abțineți de la adăugarea unei implementări schelet care să returneze mereu un vector gol, asemănător cu modul în care am procedat pentru funcția `search` în Listarea 12-16, pentru a observa compilarea nereușită a testului.

### Implementarea funcției `search_case_insensitive`

Funcția `search_case_insensitive`, ilustrată în Listarea 12-21, va fi foarte similară cu funcția `search`. Unica diferență constă în faptul că vom converti atât interogarea cât și fiecare linie la litere mici, astfel încât, indiferent de cazul literelor argumentelor de intrare, ele vor fi în același format când verificăm dacă linia conține interogarea.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

<span class="caption">Listarea 12-21: Definirea funcției `search_case_insensitive` pentru a converti interogarea și linia la litere mici înainte de a efectua comparația</span>

În primul rând, convertim string-ul `query` la litere mici și îl salvăm într-o variabilă nouă, umbrită, păstrând același nume. Apelarea funcției `to_lowercase` asupra interogării este esențială astfel încât, indiferent de forma interogării utilizatorului – fie `"rust"`, fie `"RUST"`, fie `"Rust"`, fie `"rUsT"` – noi vom considera interogarea ca fiind `"rust"`, tratând-o fără să ținem cont de cazul literelor. Deși `to_lowercase` poate procesa caractere Unicode de bază, acuratețea nu va fi de 100%. Dacă am dezvolta o aplicație reală, am dori să fim mai meticuloși în această privință, însă aici ne concentrăm pe variabilele de mediu și nu pe Unicode, așa că o lăsăm așa cum este.

Să reținem că acum `query` devine un `String` și nu o secțiune de string, deoarece aplicarea funcției `to_lowercase` generează date noi în loc să referințieze datele existente. Să luăm de exemplu interogarea `"rUsT"`: secțiunea de string originală nu include un `u` sau un `t` în litere mici la care noi să avem acces, așadar este necesar să creăm un nou `String` care să conțină `"rust"`. Când transmitem interogarea ca argument funcției `contains`, trebuie să adăugăm semnul ampersand (&) deoarece semnătura funcției `contains` necesită o secțiune de string.

Acum adăugăm un apel la `to_lowercase` pentru fiecare `line` pentru a transforma toate caracterele în litere mici. După ce am convertit `line` și `query` la litere mici, vom identifica potriviri indiferent de majusculele sau minusculele interogării.

Să verificăm dacă această implementare trece testele:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Excelent! Au trecut. Acum, să invocăm funcția nouă `search_case_insensitive` din cadrul funcției `run`. Pentru început, vom adăuga o opțiune de configurare în structura `Config` pentru a alege între căutarea sensibilă și căutarea insensibilă la majuscule. Adăugarea acestui câmp va cauza erori de compilare deoarece nu inițializăm acest câmp în niciun loc momentan:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:here}}
```

Am adăugat câmpul `ignore_case` care conține un Boolean. Acum trebuie ca funcția `run` să verifice valoarea câmpului `ignore_case` și să folosească această informație pentru a decide dacă va apela funcția `search` sau `search_case_insensitive`, după cum este indicat în Listarea 12-22. Această parte încă nu va compila.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:there}}
```

<span class="caption">Listarea 12-22: Apelarea fie `search`, fie `search_case_insensitive` în funcție de valoarea din `config.ignore_case`</span>

În final, avem nevoie să verificăm variabila de mediu. Funcțiile pentru interacțiunea cu variabilele de mediu se află în modulul `env` din biblioteca standard, așadar îl importăm în domeniu de vizibilitate la începutul *src/lib.rs*. Vom folosi apoi funcția `var` din modulul `env` pentru a determina dacă o valoare a fost stabilită pentru o variabilă de mediu numită `IGNORE_CASE`, așa cum se arată in Listarea 12-23.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/lib.rs:here}}
```

<span class="caption">Listarea 12-23: Căutarea pentru orice valoare într-o variabilă de mediu numită `IGNORE_CASE`</span>

Aici, definim o nouă variabilă denumită `ignore_case`. Pentru a-i aloca o valoare, apelăm funcția `env::var` pe care o furnizează modulul `env`, pasându-i ca argument numele variabilei de mediu `IGNORE_CASE`. Funcția `env::var` returnează un `Result` care în caz de succes devine varianta `Ok` și conține valoarea variabilei de mediu, dacă aceasta a fost setată. Dacă variabila de mediu nu este prezentă, funcția va returna varianta `Err`.

Folosim metoda `is_ok` de pe `Result` pentru a verifica existența variabilei de mediu, semn că programul ar trebui să efectueze o căutare insensibilă la majuscule. Dacă variabila `IGNORE_CASE` nu este specificată, `is_ok` va da ca rezultat fals și, în consecință, programul va realiza o căutare sensibilă la majuscule (case-sensitive). Nu ne concentram asupra *valorii* variabilei de mediu, ci doar asupra faptului dacă aceasta este definită sau nu, prin urmare ne bazăm pe metoda `is_ok` în loc să optăm pentru `unwrap`, `expect` sau alte metode asociate cu `Result`.

Valoarea din `ignore_case` este transmisă instanței `Config`, permițând astfel funcției `run` să acceseze această informație și să decidă dacă să apeleze `search_case_insensitive` sau `search`, conform implementării din Listarea 12-22.

Să încercăm! În primul rând, să rulăm programul fără să setăm variabila de mediu și cu interogarea `to`, care ar trebui să potrivească orice linie ce conține cuvântul „to” în litere mici:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Iată că funcționează și așa! Acum, să executăm programul cu variabila `IGNORE_CASE` setată la `1`, dar păstrând aceeași interogare `to`.

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Dacă utilizezi PowerShell, va trebui să setezi variabila de mediu și să execuți programul cu două comenzi separate:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Aceasta va face ca `IGNORE_CASE` să rămână activă pentru restul sesiunii de shell. Poate fi dezactivată folosind cmdlet-ul `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Ar trebui să obținem linii ce conțin „to” și care ar putea avea litere mari:

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Excelent, am primit și linii care conțin „To”! Programul nostru `minigrep` poate acum efectua căutări care nu țin cont de caz, controlate prin intermediul unei variabile de mediu. Acum cunoști modul în care se pot gestiona opțiunile setate fie prin argumente de linie de comandă, fie prin variabile de mediu.

Unele programe permit folosirea argumentelor *și* variabilelor de mediu pentru aceeași configurare. În asemenea situații, programele stabilesc care din cele două opțiuni are prioritate. Ca exercițiu suplimentar individual, încearcă să controlezi sensibilitatea la majuscule fie printr-un argument de linie de comandă, fie printr-o variabilă de mediu. Decide care dintre cele două - argumentul de linie de comandă sau variabila de mediu - ar trebui să prevaleze în cazul în care programul este executat cu unul setat pe sensibilitate la majuscule și celălalt pe ignorarea diferențelor.

Modulul `std::env` include multe alte funcționalități utile legate de variabilele de mediu: verifică documentația pentru a vedea ce alte posibilități îți oferă.
