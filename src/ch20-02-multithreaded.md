## Transformarea serverului nostru single-threaded într-unul multithreaded

În starea actuală, serverul nostru procesează fiecare cerere succesiv, ceea ce înseamnă că nu va gestiona o altă conexiune până când nu termină de procesat prima. Dacă serverul se confruntă cu un număr mare de cereri, această metodă de execuție secvențială devine tot mai puțin eficientă. Mai mult, dacă serverul întâmpină o cerere ce necesită timp îndelungat pentru procesare, cererile ulterioare trebuie să aștepte finalizarea acesteia, chiar dacă ar putea fi procesate rapid. Acesta este un aspect pe care trebuie să-l îmbunătățim, dar înainte de asta, să analizăm problema în detaliu.

### Simularea unei solicitări lente în implementarea actuală a serverului

Să examinăm cum o solicitare care se procesează lent poate influența alte solicitări transmise serverului nostru actual. Listarea 20-10 ilustrează gestionarea unei solicitări către adresa */sleep* cu un răspuns lent simulat, care va cauza serverului să execute un somn de 5 secunde înainte de a răspunde.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-10/src/main.rs:here}}
```

<span class="caption">Listarea 20-10: Simularea unei solicitări lente prin dormit timp de 5 secunde</span>

Am trecut de la utilizarea `if` la `match`, deoarece acum avem de-a face cu trei scenarii diferite. Este necesar să realizăm potriviri explicite asupra unei secțiuni din `request_line` pentru a se potrivi cu valorile literale string; `match` nu realizează referențierea și dereferențierea automată, cum o face metoda de egalitate.

Primul caz este la fel ca blocul `if` din Listarea 20-9. Al doilea caz corespunde unei solicitări către */sleep*. Când se primește aceasta, serverul va aștepta 5 secunde înainte de a reda pagina HTML de succes. Cazul al treilea este identic cu blocul `else` din Listarea 20-9.

Putem vedea natura simplistică a serverului nostru: biblioteci reale ar trata recunoașterea multiplelor solicitări într-un mod mult mai eficient și concis!

Pentru a iniția serverul, folosește `cargo run`. Apoi, deschide două ferestre de browser: una pentru *http://127.0.0.1:7878/* și cealaltă pentru *http://127.0.0.1:7878/sleep*. Dacă accesezi URI-ul */* de mai multe ori, la fel ca anterior, va răspunde rapid. Dar dacă accesezi */sleep* și apoi încarci */*, observi că */* va aștepta finalizarea celor 5 secunde de somn a funcției `sleep` înainte să se încarce.

Există mai multe tehnici pe care le-ar putea folosi pentru a evita întârzierea solicitărilor din cauza unei alte solicitări lente; vom alege să implementăm un pool de fire de execuție.

### Lărgirea capacității de bandă cu un pool de fire de execuție

Un *thread pool* este un ansamblu de fire de execuție inițializate și în așteptare de sarcini. Când programul detectează o nouă sarcină, alocă un fir din pool pentru gestionarea acesteia, care o va prelucra. Celelalte fire rămân disponibile pentru orice alte sarcini care apar în timp ce primul fir este ocupat. După finalizarea sarcinii, firul revine în pool-ul de fire inactive, pregătit pentru o nouă provocare. Astfel, thread pool-ul facilitează procesarea concurentă a conexiunilor, sporind capacitatea de bandă a serverului.

Vom limita numărul de fire de execuție din pool pentru a ne proteja împotriva atacurilor de tipul Denial of Service (DoS); altfel, creând un fir nou pentru fiecare cerere, un atacator care trimite milioane de cereri ar putea să epuizeze resursele serverului, blocând procesarea acestora.

Deci, în loc de a avea un număr nelimitat de fire, vom menține un număr fix de fire în așteptare în pool. Sosirea unei cereri conduce la direcționarea acesteia spre pool pentru prelucrare, unde există o coadă de cereri în așteptarea gestiunii. Fiecare fir din pool preia o cerere din coadă, o gestionează și apoi solicită o altă cerere. Prin această abordare, putem procesa concurent până la `N` cereri, unde `N` este numărul de fire. Aceasta crește numărul de cereri de lungă durată pe care le putem prelucra înainte de a se forma o coadă de așteptare.

Metoda dată este una dintre multele strategii de îmbunătățire a capacității unui server web. Alte opțiuni demne de luat în considerare sunt modelul *fork/join*, modelul *single-threaded async I/O*, sau modelul *multi-threaded async I/O*. Dacă domeniul te pasionează, e bine de știut că Rust, ca limbaj de nivel scăzut, face posibilă explorarea și implementarea acestor alternative.

Înainte să trecem la construirea unui thread pool, să examinăm cum ar trebui să arate interacțiunea cu acesta. În etapa de proiectare a codului, construirea în prealabil a interfeței cu utilizatorul poate servi ca un bun ghid pentru design. Concepe API-ul astfel încât să se potrivească cu modul în care vrei să-l utilizezi; apoi dezvoltă funcționalitatea în acest cadru, în loc să creezi funcționalitățile și pe urmă să te ocupi de API.

Similar cu dezvoltarea ghidată de teste din proiectul Capitolului 12, vom folosi aici dezvoltarea ghidată de compilator. Vom scrie cod ce cheamă funcțiile dorite și vom corecta erorile indicate de compilator pentru a avansa spre o implementare funcțională. Dar înainte de asta, să explorăm o tehnică pe care nu o vom adopta în calitate de punct de plecare.

<!-- Old headings. Do not remove or links may break. --> <a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Crearea unui fir de execuție pentru fiecare cerere

Să explorăm cum ar arăta codul nostru dacă ar genera un nou fir de execuție pentru fiecare conexiune. După cum am explicat anterior, aceasta nu este soluția finală datorită complicațiilor care ar apărea prin crearea unui număr nelimitat de fire de execuție, însă este un punct de start pentru a realiza mai întâi un server funcțional multithreaded. Apoi, vom adăuga un thread pool ca o îmbunătățire, iar compararea celor două abordări va fi mai simplă. Listarea 20-11 ilustrează modificările necesare în `main` pentru a crea un nou fir de execuție care să proceseze fiecare stream în cadrul buclei `for`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-11/src/main.rs:here}}
```

<span class="caption">Listarea 20-11: Crearea unui nou fir de execuție pentru fiecare stream</span>

După cum ai învățat în Capitolul 16, `thread::spawn` va crea un nou fir de execuție și va executa codul din încheiere în acest nou fir. Dacă rulezi acest cod și accesezi
*/sleep* în browser-ul tău, apoi */* în alte două tab-uri, vei vedea că cererile către */* nu trebuie să aștepte finalizarea cererii către */sleep*. Cu toate acestea, așa cum am menționat, această metodă poate în cele din urmă să suprasolicite sistemul prin crearea continuă de noi fire de execuție fără niciun fel de limită.

<!-- Old headings. Do not remove or links may break. --> <a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Crearea unui număr finit de fire de execuție

Scopul nostru este ca pool-ul de fire să fie intuitiv și să funcționeze în mod similar cu firele de execuție individuale, astfel încât să nu fie necesare ajustări majore în codul sursă care interacționează cu API-ul creat de noi. Listarea 20-12 ilustrează interfața ideală pentru structura `ThreadPool` pe care dorim să o adoptăm în locul folosirii `thread::spawn`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/listing-20-12/src/main.rs:here}}
```

<span class="caption">Listarea 20-12: Interfața vizată pentru `ThreadPool`</span>

Utilizăm `ThreadPool::new` pentru a pune la punct un pool nou de fire de execuție cu un număr de fire pe care îl putem configura, aici exemplificând cu patru. Mai departe, în bucla `for`, `pool.execute` propune o interfață similară cu cea a lui `thread::spawn`, primind o închidere care urmează a fi executată de pool individual pentru fiecare flux. Este necesar să definim `pool.execute` astfel încât să preia închiderea și să o repartizeze către un fir din pool pentru execuție. Deocamdată codul nu este compilabil, însă o vom încerca pentru ca compilatorul să ne poată îndruma spre soluționarea problemelor.

<!-- Old headings. Do not remove or links may break. --> <a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Construirea unui `ThreadPool` prin dezvoltare ghidată de compilator

Aplică modificările din Listarea 20-12 în *src/main.rs*, după care să utilizăm erorile de compilator de la `cargo check` ca ghid în procesul nostru de dezvoltare. Iată prima eroare care ne apare:

```console
{{#include ../listings/ch20-web-server/listing-20-12/output.txt}}
```

Minunat! Această eroare ne indică faptul că avem nevoie de un tip sau modul denumit `ThreadPool`, deci e momentul să construim unul. Implementarea `ThreadPool` va fi realizată independent de tipul de muncă pe care serverul nostru web îl execută. Astfel, să convertim crate-ul `hello` dintr-un crate tip binar într-unul de tip bibliotecă, pentru a include implementarea `ThreadPool`. Odată ce am trecut la un crate de tip bibliotecă, am putea folosi biblioteca de pool de fire de execuție și pentru alte munci la care am avea nevoie de un pool de fire de execuție, nu doar pentru prelucrarea cererilor web.

Creează un fișier *src/lib.rs* care să conțină următoarea definiție, care este cel mai simplu exemplu al structurii `ThreadPool` pe care îl putem avea în acest moment:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

Acum, editează fișierul *main.rs* pentru a aduce `ThreadPool` în domeniul de vizibilitate din crate-ul de tip bibliotecă adăugând următorul cod la partea de sus a *src/main.rs*:

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

Acest cod nu va funcționa încă, dar să facem încă o verificare pentru a primi următoarea eroare ce trebuie abordată:

```console
{{#include ../listings/ch20-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

Această eroare ne arată că trebuie să creăm în continuare o funcție asociată numită `new` pentru `ThreadPool`. De asemenea, înțelegem că funcția `new` trebuie să aibă un parametru ce acceptă `4` ca argument și care să returneze o instanță de `ThreadPool`. Să implementăm cea mai simplă versiune a funcției `new`, care să corespundă acestor cerințe:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

Am ales tipul `usize` pentru parametrul `size`, deoarece este clar că un număr negativ de fire de execuție nu ar avea sens. În plus, știm că acest număr 4 va fi folosit pentru a reprezenta cantitatea de elemente într-o colecție de fire de execuție, acesta fiind rolul tipului `usize`, după cum am discutat în secțiunea [“Tipurile de întregi”][integer-types]<!-- ignore --> din Capitolul 3.

Să examinăm din nou codul:

```console
{{#include ../listings/ch20-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

Eroarea apare acum deoarece `ThreadPool` nu are o metodă `execute`. Reamintim din secțiunea [„Crearea unui număr finit de fire de execuție”](#creating-a-finite-number-of-threads)<!-- ignore --> că am stabilit ca pool-ul nostru de fire de execuție să aibă o interfață asemănătoare cu `thread::spawn`. În plus, intenționăm să implementăm funcția `execute` într-o manieră care să preia o închidere furnizată și să o aloce unui fir inactiv din pool pentru executare.

Definirea metodei `execute` pe `ThreadPool` se va face astfel încât aceasta să primească o închidere ca parametru. Reflectând asupra secțiunii [„Permutarea valorilor capturate din închideri și trăsături `Fn`”][fn-traits]<!-- ignore --> din Capitolul 13, ținem minte că putem accepta închideri ca parametri utilizând trei trăsături distincte: `Fn`, `FnMut` și `FnOnce`. Acum trebuie să hotărâm care dintre acestea este potrivită în contextul nostru. Având în vedere că vom urma o abordare similară cu cea a implementării `thread::spawn` din librăria standard, putem examina restricțiile aplicate de semnătura lui `thread::spawn` asupra parametrului său. Documentația ne evidențiază următoarea formă:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Aici, parametrul de tip `F` este cel la care ne vom concentra; parametrul de tip `T` se raportează la valoarea de retur, ce nu constituie un punct de interes în acest context. Observăm că `spawn` recurge la trăsătura `FnOnce` ca delimitare pe `F`. Probabil că aceasta este alegerea corectă pentru noi, întrucât argumentul primit în `execute` va fi transferat către `spawn`. Convicția că `FnOnce` este trăsătura adecvată se întărește de faptul că un fir de execuție destinat procesării unei cereri va rula închiderea respectivă doar o singură dată, conform implicării termenului `Once` în `FnOnce`.

Parametrul de tip `F` are acum și delimitarea de trăsătură `Send` și delimitarea de durata de viață `'static`, esențiale în cazul nostru: `Send` este necesar pentru a transfera închiderea de la un fir de execuție la altul și `'static` este necesar deoarece nu putem determina durata execuției firului. În continuare să implementăm o metodă `execute` pe `ThreadPool` care să primească un parametru generic de tip `F` cu aceste constrângeri:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

Continuăm să utilizăm `()` după `FnOnce` pentru că acest `FnOnce` definește o închidere fără parametri și care returnează tipul unit `()`. Ca și în cazul definițiilor funcțiilor, putem omite tipul de retur din semnătură, dar chiar și în absența parametrilor, avem totuși nevoie de paranteze.

Revenind, aceasta este implementarea cea mai simplificată a metodei `execute`: nu realizează nimic, însă obiectivul nostru este numai de a verifica compilarea codului. Să verificăm din nou:

```console
{{#include ../listings/ch20-web-server/no-listing-03-define-execute/output.txt}}
```

Compilarea reușește! Totuși, vei vedea că dacă rulezi `cargo run` și inițiezi o cerere în browser, vei întâmpina erorile din browser pe care le-am observat la începutul capitolului. Biblioteca noastră încă nu execută închiderea transmisă metodei `execute`!

> Notă: O expresie pe care o poți auzi în legătură cu limbaje de programare cu
> compilatoare stricte, cum sunt Haskell și Rust, este „dacă codul compilează,
> înseamnă că funcționează.” Totuși, acest adagiu nu este valabil universal.
> Proiectul nostru compilează, însă nu realizează efectiv nimic! Dacă am
> dezvolta un proiect real și complet, acum ar fi momentul oportun pentru a
> compune teste unitare pentru a confirma că codul nu doar că compilează, dar
> și că se comportă așa cum anticipăm.

#### Validarea numărului de fire de execuție în `new`

Încă nu am folosit parametrii metodei `new` și `execute`. Să realizăm acum implementările acestora conform comportamentului dorit. Începând cu `new`, mai devreme am optat pentru un tip de date fără semn pentru parametrul `size`, deoarece ideea unui pool cu un număr negativ de fire de execuție e lipsită de sens. Totuși, un pool cu zero fire este de asemenea ilogic, chiar dacă zero este o valoare perfect validă pentru `usize`. Vom include un cod care să asigure că `size` este mai mare ca zero înainte să returnăm o instanță `ThreadPool`, forțând programul să genereze panică dacă primește zero folosind macrocomanda `assert!`, conform prezentării din Listarea 20-13.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-13/src/lib.rs:here}}
```

<span class="caption">Listarea 20-13: Implementarea lui `ThreadPool::new` astfel încât să genereze panică dacă `size` este zero</span>

Am integrat de asemenea comentarii doc pentru documentarea `ThreadPool`. Observi că am respectat practicile recomandate pentru documentație, incluzând o secțiune care subliniază situațiile în care funcția noastră poate intra în panică, aspect abordat în Capitolul 14. Încearcă să executați comanda `cargo doc --open` și să selectezi structura `ThreadPool` pentru a vedea cum arată documentația generată pentru `new`!

În loc să folosim macrocomanda `assert!`, am putea transforma `new` în `build` și să returnăm un `Result` așa cum am procedat cu `Config::build` în proiectul legat de intrare/ieșire (I/O) din Listarea 12-9. Cu toate acestea, am decis că în acest caz încercarea de a crea un pool de fire de execuție fără fire este o eroare irecuperabilă. Dacă ești în căutarea unei provocări, încearcă să scrii o metodă denumită `build` cu următoarea semnătură pentru a o compara cu metoda `new`:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Crearea spațiului pentru stocarea firelor de execuție

Acum, că avem o metodă de a verifica dacă avem un număr valid de fire de execuție pentru a le stoca în pool, putem crea aceste fire și să le stocăm în structura `ThreadPool` înainte de a o returna. Dar cum „stocăm” un fir de execuție? Să aruncăm din nou o privire la semnătura `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Funcția `spawn` returnează un `JoinHandle<T>`, unde `T` este tipul returnat de închidere. Să încercăm să folosim și `JoinHandle` și să vedem ce se întâmplă. În cazul nostru, închiderile pe care le pasăm pool-ului de fire sunt responsabile pentru gestionarea conexiunii și nu returnează nimic, astfel `T` va fi de tipul unit `()`.

Codul din Listarea 20-14 va compila, dar nu va crea niciun fir de execuție până în acest punct. Am modificat definiția `ThreadPool` pentru a include un vector de `thread::JoinHandle<()>`, am inițializat vectorul cu o capacitate de `size`, am configurat o buclă `for` pentru a executa un cod ce va crea firele și am returnat o instanță `ThreadPool` care le include.

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/listing-20-14/src/lib.rs:here}}
```

<span class="caption">Listarea 20-14: Crearea unui vector pentru `ThreadPool` pentru stocarea firelor de execuție</span>

Am importat `std::thread` în domeniul de vizibilitate al crate-ului de bibliotecă, deoarece utilizăm `thread::JoinHandle` ca tip al elementelor din vectorul `ThreadPool`.

Odată ce se confirmă primirea unei dimensiuni valide, `ThreadPool`-ul nostru generează un nou vector capabil să conțină `size` elemente. Funcția `with_capacity` îndeplinește aceeași sarcină ca şi `Vec::new`, dar cu un avantaj considerabil: ea alocă spațiul în vector în prealabil. Având în vedere că știm că trebuie să stocăm `size` de elemente în vector, această alocare inițială este mai eficientă decât utilizarea `Vec::new`, ce își ajustează mărimea pe măsură ce se adaugă elemente.

Când rulezi din nou `cargo check`, procesul ar trebui să fie realizat cu succes.

#### Structura `Worker` responsabilă de transmiterea codului din `ThreadPool` către un fir de execuție

În listarea 20-14, în secțiunea buclei `for`, am deschis subiectul creării firelor de execuție. Vom detalia acum modalitatea efectivă de creare a acestora. Biblioteca standard ne oferă funcționalitatea `thread::spawn` pentru a genera fire de execuție, prin care `thread::spawn` anticipează recepționarea unui fragment de cod ce trebuie executat imediat după pornirea firului. Cu toate acestea, cazul nostru este de a inițializa firele de execuție pentru ca acestea să *rămână în așteptare*, în vederea primirii codului pe care îl vom furniza la un moment ulterior. Abordarea standard a firelor de execuție nu include o astfel de capacitate, ceea ce ne obligă să implementăm manual acest comportament.

Vom obține acest comportament prin integrarea unei noi structuri de date între `ThreadPool` și firele de execuție, destinată gestionării acestei funcționalități noi. Această structură va purta denumirea de *Worker* (lucrător), termen des întâlnit în schemele de pooling. `Worker`-ul preia codul așteptând execuția și îl procesează folosind firul de execuție alocat lui. Gândește-te la bucătarii dintr-un restaurant: aceștia așteaptă comenzi de la clienți și, la sosirea acestora, sunt responsabili pentru prelucrarea și finalizarea lor.

În `ThreadPool`, în locul unui vector conținând instanțe de `JoinHandle<()>`, vom include mai degrabă instanțe ale structurii `Worker`. Fiecare `Worker` va deține o singură instanță `JoinHandle<()>`. Vom dezvolta o metodă pe structura `Worker` ce va accepta o închidere cu cod spre a fi executat, pe care o va expedia firului de execuție activ pentru procesare. În plus, vom atribui fiecărei instanțe de `Worker` un `id`, astfel încât să putem distinge cu ușurință între diferiții lucrători ai pool-ului, în contexte de logging sau depanare.

Iată procedura nouă care are loc atunci când inițiem un `ThreadPool`. Vom implementa codul care expediază închiderea la firul de execuție odată ce avem `Worker` configurat în această manieră:

1. Definește o structură `Worker` care posedă un `id` și un `JoinHandle<()>`.
2. Modifică `ThreadPool` pentru a păstra un vector de instanțe `Worker`.
3. Creează o funcție `Worker::new` care acceptă un număr `id` și înapoiază o instanță `Worker` care conține `id`-ul și un fir de execuție generat cu o închidere goală.
4. În `ThreadPool::new`, utilizează contorul din bucla `for` pentru a produce un `id`, creează un nou `Worker` cu acest `id` și îl stochează în vector.

Dacă ești dispus de o provocare, încearcă să realizezi aceste modificări de unul singur înainte de a consulta codul din Listarea 20-15.

Ești gata? Aici este Listarea 20-15 care prezintă una dintre modalitățile de a efectua modificările discutate.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-15/src/lib.rs:here}}
```

<span class="caption">Listarea 20-15: Modificarea `ThreadPool` pentru a include instanțe `Worker` în loc de a păstra direct fire de execuție</span>

Am modificat numele câmpului din `ThreadPool` de la `threads` la `workers`, deoarece acum include instanțe `Worker` în loc de instanțe `JoinHandle<()>`. Utilizăm numărătorul din bucla `for` ca argument pentru `Worker::new` și stocăm fiecare `Worker` nou în vectorul numit `workers`.

Codul exterior (precum serverul nostru din *src/main.rs*) nu trebuie să fie conștient de detalii de implementare legate de utilizarea structurii `Worker` în `ThreadPool`, de aceea structura `Worker` și funcția sa `new` sunt marcate ca private. Funcția `Worker::new` folosește `id`-ul primit și reține o instanță `JoinHandle<()>` care este creată generând un fir de execuție nou cu o închidere goală.

> Notă: Dacă sistemul de operare nu reușește să creeze un fir de execuție din
> cauza lipsei resurselor de sistem, `thread::spawn` va genera panică. Asta va
> provoca generația de panică a întregului server, chiar dacă crearea altor fire
> de execuție ar putea avea succes. Din motive de simplitate, acest comportament
> este accpetabil, însă într-o implementare ThreadPool destinată producției, ar
> fi de preferat să utilizăm [`std::thread::Builder`][builder]<!-- ignore --> și
> metoda sa [`spawn`][builder-spawn]<!-- ignore -->, care returnează `Result`,
> în schimb.

Codul se va compila și va reține numărul de instanțe `Worker` pe care le-am definit ca argument pentru `ThreadPool::new`. Totuși, *încă* nu gestionăm închiderea pe care o primim în `execute`. Să vedem cum putem proceda în continuare.

#### Trimiterea cererilor către fire de execuție utilizând canale

Problema pe care o abordăm acum este că închiderile oferite lui `thread::spawn` în acest moment sunt nefuncționale. Obținem închiderea dorită pentru execuție în metoda `execute`, dar trebuie să furnizăm lui `thread::spawn` o închidere pentru a o executa atunci când constituim fiecare `Worker` la inițierea `ThreadPool`.

Ne dorim ca structurile `Worker` să recupereze codul ce trebuie rulat dintr-o coadă menținută în `ThreadPool` și să îl transmită propriului fir pentru execuție.

Canalele care au fost analizate în Capitolul 16, o cale simplă de comunicare între două fire de execuție, se potrivesc excelent pentru necesitatea noastră. Vom utiliza un canal ca și coadă a sarcinilor, iar `execute` va trimite sarcinile de la `ThreadPool` spre instanțele de `Worker`, care la rândul lor vor trimite sarcina către firul de execuție. Iată strategia:

1. `ThreadPool` va iniția un canal și va menține transmițătorul.
2. Fiecare `Worker` va deține receptorul.
3. Vom crea o structură de tip `Job` care va conține închiderile pe care intenționăm să le expediem prin canal.
4. Metoda `execute` va livra sarcina preconizată pentru execuție printr-un transmițător.
5. În cadrul firului său de execuție, `Worker` va parcurge receptorul implementând închiderile oricărui sarcini recepționate.

Începem prin a constitui un canal în `ThreadPool::new` și reținem transmițătorul în instanța `ThreadPool`, așa cum este ilustrat în Listarea 20-16. Deocamdată, structura `Job` nu înmagazinează nimic, dar va constitui tipul elementelor ce vor fi transmise prin canal.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-16/src/lib.rs:here}}
```

<span class="caption">Listarea 20-16: Modificarea `ThreadPool` pentru stocarea transmițătorului unui canal care expediază instanțe de sarcini</span>

În `ThreadPool::new`, concepem noul nostru canal și alocăm pool-ului păstrarea transmițătorului, care va asigura o compilare de succes.

Să transmitem un receptor al canalului fiecărui lucrător în momentul în care pool-ul de fire de execuție inițiază canalul. Ne propunem să folosim receptorul în firul pe care lucrătorii îl vor genera, așadar vom referi `receiver` în închidere. Codul prezentat în Listarea 20-17, însă, nu va compila încă.

```rust,ignore,does_not_compile
...
```

<span class="caption">Listarea 20-17: Transmiterea receptorului lucrătorilor</span>

Am implementat câteva schimbări mici și simple: pasăm receptorul către `Worker::new`, după care îl utilizăm în cadrul închiderii.

La verificarea acestui cod, apare următoarea eroare:

```console
{{#include ../listings/ch20-web-server/listing-20-17/output.txt}}
```

Se încearcă transmiterea `receiver` către multiple instanțe `Worker`. Aceasta nu va avea succes, deoarece, așa cum ne amintim din Capitolul 16, implementarea canalului în Rust este configurată pentru mai mulți *producători* și un singur *consumator*. Astfel, nu putem duplica simplu partea ce consumă din canal pentru a remedia codul. Nu intenționăm nici să expediem același mesaj la diverși consumatori; intenționăm să avem o listă unică de mesaje procesate de mai mulți lucrători, fiecare mesaj fiind procesat individual.

Mai mult, preluarea unei sarcini din coada canalului presupune modificarea lui `receiver`, motiv pentru care firele de execuție necesită o cale sigură de a împărți și de a modifica `receiver`. Altminteri, am putea întâlni situații de concurență, așa cum am discutat în Capitolul 16.

Adusu-ți aminte de conceptul de pointeri inteligenți siguri de fir de execuție abordat în Capitolul 16: pentru a distribui posesiunea între mai multe fire și pentru a permite modificări ale valorii, trebuie să utilizăm `Arc<Mutex<T>>`. `Arc` le va permite lucrătorilor să posede receptorul împreună, iar `Mutex` va garanta că un singur lucrător va prelua o sarcină de la receptor la un moment dat. Listarea 20-18 prezintă ajustările necesare.

```rust,noplayground
...
```

<span class="caption">Listarea 20-18: Partajarea receptorului între lucrători, utilizând `Arc` și `Mutex`</span>

În funcția `ThreadPool::new`, așezăm receptorul într-un `Arc` și un `Mutex`. Pentru fiecare lucrător nou, replicăm `Arc` pentru a spori contorul de referințe, permițând lucrătorilor să dețină receptorul comun.

Odată cu aceste modificări, codul se compilează cu succes! Suntem pe calea cea bună!

#### Implementarea metodei `execute`

Acum să ne apucăm de implementarea metodei `execute` pentru `ThreadPool`. Totodată, transformăm `Job` dintr-o structură într-un alias de tip pentru un obiect-trăsătură care înglobează tipul închiderii pe care `execute` o primește. După cum am discutat în secțiunea [„Crearea sinonimelor de tip cu aliasuri de tip”][creating-type-synonyms-with-type-aliases]<!-- ignore --> din Capitolul 19, aliasurile de tip ne sunt de ajutor pentru a abrevia tipuri lungi și a le folosi mai ușor. Să privim Listarea 20-19.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-19/src/lib.rs:here}}
```

<span class="caption">Listarea 20-19: Crearea unui alias de tip `Job` sub forma unui `Box` care încapsulează fiecare închidere și ulterior trimiterea sarcinii printr-un canal</span>

După ce generăm o instanță nouă de `Job` utilizând închiderea primită în `execute`, expediem această sarcină către capătul transmițător al canalului. Se apelează `unwrap` pe `send` în cazul în care transmisia dă greș. Acest lucru s-ar putea întâmpla dacă, spre exemplu, am înceta execuția tuturor firelor noastre, caz în care capătul receptor nu mai primește mesaje noi. Momentan, firele noastre de execuție nu pot fi oprite: acestea vor rula atât timp cât pool-ul este activ. Folosim `unwrap` deoarece suntem siguri că situația de eșec nu se va produce, deși compilatorul nu poate avea această certitudine.

Totuși, ne mai rămâne un pas de efectuat! În cadrul firului de execuție, închiderea trimisă către `thread::spawn` se limitează în continuare la utilizarea capătului receptor al canalului. Trebuie ca această închidere să intre într-o buclă care să solicite în mod repetat de la receptorul canalului o nouă sarcină și să o execute odată ce a fost primită. Implementăm schimbarea indicată în Listarea 20-20 în `Worker::new`.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-20/src/lib.rs:here}}
```

<span class="caption">Listarea 20-20: Recepția și executarea sarcinilor în firul de execuție</span>

Aici, inițial facem un apel la `lock` pe `receiver` pentru a obține mutex-ul, urmând să apelăm `unwrap` pentru a provoca panică în caz de erori. Procesul de a dobândi un lock poate eșua dacă mutex-ul se află într-o stare *otrăvită*, o situație ce poate apărea dacă un alt fir de execuție a intrat în panică în timp ce avea lock-ul, în loc să-l elibereze. În această circumstanță, utilizarea `unwrap` pentru ca acest fir să intre în panică este acțiunea corectă. Ai libertatea de a înlocui acest `unwrap` cu un `expect` care să conțină un mesaj de eroare care îți este familiar.

Dacă reușim să obținem lock-ul pe mutex, apelăm `recv` pentru a recepționa un `Job` din canal. Un alt `unwrap` ne ajută să depășim orice alte erori aici, care ar putea surveni dacă firul care deține sender-ul s-a terminat, într-o manieră similară cu modul în care metoda `send` returnează `Err` dacă receiver-ul este oprit.

Apelul la `recv` blochează execuția, deci dacă nu există încă o sarcină, firul curent va aștepta până când una devine disponibil. `Mutex<T>` asigură că numai un fir `Worker` încearcă la un moment dat să solicite o sarcină. 

Acum, pool-ul nostru de firuri este operațional! Încearcă să-l rulezi cu `cargo run` și trimite câteva solicitări.

<!-- manual-regeneration
cd listings/ch20-web-server/listing-20-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field is never read: `workers`
 --> src/lib.rs:7:5
  |
7 |     workers: Vec<Worker>,
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: field is never read: `id`
  --> src/lib.rs:48:5
   |
48 |     id: usize,
   |     ^^^^^^^^^

warning: field is never read: `thread`
  --> src/lib.rs:49:5
   |
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

warning: `hello` (lib) generated 3 warnings
    Finished dev [unoptimized + debuginfo] target(s) in 1.40s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Success! Acum avem un pool de fire de execuție care procesează conexiunile în mod asincron. Suntem limitați la crearea a maximum patru thread-uri, prin urmare sistemul nostru nu va fi copleșit dacă serverul va primi un număr mare de cereri. Dacă trimitem o cerere către
*/sleep*, serverul va fi capabil să gestioneze alte cereri utilizând un alt thread pentru executarea lor.

> Notă: dacă deschizi */sleep* în multiple ferestre de browser în același timp,
> este posibil ca paginile să fie încărcate una după alta, în interval de 5
> secunde fiecare. Unele browsere web procesează multiple instanțe ale
> aceleiași cereri în mod secvențial din cauza strategiilor de caching. Această
> limitare nu este un efect al serverului nostru web.

După ce am explorat bucla `while let` în Capitolul 18, ai putea fi curios de ce nu am implementat codul firului `Worker` așa cum este prezentat în Listarea 20-21.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/listing-20-21/src/lib.rs:here}}
```

<span class="caption">Listarea 20-21: O versiune alternativă de implementare a `Worker::new` utilizând `while let`</span>

Codul se compilează și funcționează, dar nuduce la comportamentul dorit al thread-urilor: o cerere lentă va încetini procesarea altor cereri. Explicația este destul de subtilă: structura `Mutex` nu oferă o metodă `unlock` accesibilă publicului deoarece dreptul asupra blocării se stabilește prin durata de viață a `MutexGuard<T>` în rezultatul `LockResult<MutexGuard<T>>` returnat de metoda `lock`. Acest fapt permite verificatorului de împrumut să impună reguli la compilare care asigură că o resursă protejată de un `Mutex` nu poate fi accesată decât dacă deținem blocarea. Totodată, această implementare poate conduce la menținerea blocării pentru un timp mai îndelungat decât este necesar dacă nu suntem atenți la durata de viață a `MutexGuard<T>`.

În Listarea 20-20, codul care conține `let job = receiver.lock().unwrap().recv().unwrap();` se execută corect datorită faptului că, prin utilizarea `let`, valorile temporare implicate în expresia de la partea dreaptă a semnului egal sunt imediat eliberate odată ce instrucțiunea `let` se finalizează. În contrast, `while let` (la fel ca `if let` și `match`) nu eliberează valorile temporare decât după terminarea blocului de cod corespunzător. Conform Listării 20-21, lock-ul este păstrat activ pe întreaga durată a executării lui `job()`. Acesta împiedică ceilalți lucrători să preia joburi noi.

[creating-type-synonyms-with-type-aliases]: ch19-04-advanced-types.html#creating-type-synonyms-with-type-aliases [integer-types]: ch03-02-data-types.html#integer-types [fn-traits]: ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits [builder]: ../std/thread/struct.Builder.html [builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
