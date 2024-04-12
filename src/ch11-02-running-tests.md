## Controlul modului în care testele sunt executate

Similar cu `cargo run`, care compilează codul și ulterior rulează binarul rezultat, 
`cargo test` compilează codul în mod de test și execută binarul de testare creat. Comportamentul standard al binarului generat de `cargo test` este să execute toate testele simultan și să rețină output-ul produs pe durata execuției testelor, evitând astfel afișarea output-ului și ajutând la o lectură facilă a rezultatelor testelor. Pot fi utilizate opțiuni de linie de comandă pentru a modifica acest comportament standard.

Anumite opțiuni de linie de comandă sunt pentru `cargo test`, iar altele pentru binarul de test rezultat. Pentru a le diferenția, argumentele destinate `cargo test` sunt plasate înaintea separatorului `--`, urmate de argumentele pentru binarul de test. Executând `cargo test --help` se afișează opțiunile disponibile pentru `cargo test`, iar cu `cargo test -- --help` vei obține informații despre opțiunile ce pot fi aplicate după separator.

### Executarea testelor în paralel sau secvențial

Când execuți mai multe teste, implicit acestea se desfășoară în paralel folosind thread-uri, ceea ce duce la finalizarea lor mai rapidă și la obținerea rapidă a feedback-ului. Testele rulând simultan, este esențial să te asiguri că nu sunt interdependente sau că nu depind de o stare comună, incluzând medii partajate, precum directoriul de lucru curent sau variabilele de mediu.

Considerând cazul în care fiecare test execută cod care creează pe disk un fișier numit *test-output.txt* și îi scrie date, iar apoi verifică dacă fișierul conține o anumită valoare, diferită pentru fiecare test, simultaneitatea poate cauza probleme. Testele pot să rescrie reciproc fișierul în timp ce unul scrie și altul citește, conducând la eșecul celui de-al doilea test nu din cauza unui cod defectuos, ci din pricina interferenței între teste pe durata execuției în paralel. Soluțiile includ fiecare test să scrie într-un fișier unic sau execuția secvențială a testelor.

Pentru a evita executarea în paralel sau pentru a controla mai fin numărul de thread-uri, utilizează opțiunea `--test-threads` cu numărul de thread-uri dorit pentru binarul de test. Un exemplu ar fi:

```console
$ cargo test -- --test-threads=1
```

Setând numărul de thread-uri de test la `1`, ii transmitem programului să renunțe la paralelism. Execuția testelor pe un singur thread va lua mai mult timp decât în mod paralel, dar va preveni interferența între teste dacă acestea partajează stări comune.

### Afișarea ieșirilor funcțiilor

Implicit, când un test este efectuat cu succes în Rust, orice ieșire produs de acesta este capturat și ascuns. Să spunem că un test folosește `println!` însă el trece; în acest caz, ieșirea lui `println!` nu va fi vizibil în terminal, ci doar mesajul ce indică succesul testului. Dacă un test eșuează, tot ceea ce a fost scris în ieșirea consolă standard va fi afișat împreună cu detalii despre eșecul testului.

De exemplu, Listarea 11-10 ne prezintă o funcție care afișează valoarea parametrului său și returnează numărul 10, un test care trece și un altul care eșuează.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

<span class="caption">Listarea 11-10: Testele unei funcții ce utilizează `println!`</span>

Când executăm aceste teste cu comanda `cargo test`, vom obține următorul afișaj:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Notăm că în acest afișaj nu găsim expresia `I got the value 4`, ceea ce ar fi trebuit să fie afișat de testul care trece. Ieșirea acestuia a fost capturat. Pe de altă parte, afișajul testului eșuat, `I got the value 8`, figurează în sumarul afișajului de test, arătând și motivul eșecului.

Dacă dorim să vedem ieșirile testelor reușite, putem folosi flag-ul `--show-output` pentru a instrui Rust să afișeze rezultatele și pentru acele teste.

```console
$ cargo test -- --show-output
```

Rulând din nou testele din Listarea 11-10 cu flag-ul `--show-output`, observăm următorul afișaj:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Executarea unui subset de teste în funcție de nume

În unele cazuri, executarea întregii suite de teste poate lua timp. Dacă ești concentrat pe o secțiune anume a codului, ar putea fi util să rulezi doar testele asociate cu acea parte. Poți specifica care teste să fie executate oferind argumentul `cargo test` cu numele sau numele testelor dorite.

Ca exemplu pentru rularea unui subset de teste, vom defini întâi trei teste pentru funcția `add_two`, așa cum e ilustrat în Listarea 11-11, și vom selecta care dintre acestea să fie rulate.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

<span class="caption">Listarea 11-11: Trei teste cu nume diferite</span>

Dacă le executăm pe toate fără a specifica argumente, așa cum am văzut mai înainte, toate testele se vor executa simultan:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Executarea Testelor Individuale

Putem da numele oricărei funcții de test către `cargo test` pentru a rula exclusiv acel test:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Numai testul cu numele `one_hundred` a fost executat; celelalte două teste nu au corespuns acelui nume. Afișajul ne anunță că au existat alte teste care nu au rulat prin indicativul `2 filtered out` de la final.

Nu este posibil să specificăm numele mai multor teste în acest fel; doar prima valoare furnizată lui `cargo test` va fi utilizată. Există, însă, o metodă de a executa mai multe teste simultan.

#### Filtrarea testelor spre executare

Dacă specificăm o parte din numele unui test, orice test a cărui nume conține acea secvență va fi executat. Spre exemplu, având în vedere că numele a două dintre testele noastre includ `add`, putem executa aceste două teste folosind `cargo test add`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

Această comandă a executat toate testele ce includ `add` în numele lor și a exclus testul denumit `one_hundred`. De asemenea, este important de știut că modulul în care se află un test devine parte integrantă a numelui său, deci putem executa toate testele dintr-un modul filtrând după numele acestuia.

### Excluderea unor teste pînă la cerere explicită

Uneori, anumite teste pot fi extrem de consumatoare de timp atunci când sunt executate, motiv pentru care s-ar putea dori să le excluzi pe parcursul majorității rulărilor de `cargo test`. În loc să specifici nume după nume toate testele pe care ai de gând să le execuți, poți alege să marchezi teste consumatoare de timp cu atributul `ignore` pentru a le exclude, după cum urmează:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs}}
```

În urma adnotării `#[test]` adăugăm `#[ignore]` pentru testul pe care dorim să îl omitem. Acum, când executăm testările, `it_works` se execută, în timp ce `expensive_test` nu:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

Funcția `expensive_test` este listată ca `ignored`. Dacă dorim să executăm exclusiv testele ignorate, putem folosi comanda `cargo test -- --ignored`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Controlând care teste sunt executate, poți asigura că rezultatele tale `cargo test` se obțin rapid. Când ai stabilit că este momentul oportun să verifici rezultatele testelor `ignored` și ai timp disponibil pentru a aștepta aceste rezultate, poți opta să rulezi `cargo test -- --ignored`. Dacă intenționezi să execuți toate testele, indiferent de statutul acestora,
comanda este `cargo test -- --include-ignored`.
