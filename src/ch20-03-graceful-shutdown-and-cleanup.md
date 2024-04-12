## Închiderea ordonată și curățarea

Codul din Listarea 20-20 gestionează cererile asincron folosind un pool de fire, conform intenției noastre. Apar câteva atenționări referitoare la câmpurile `workers`, `id` și `thread` pe care nu le utilizăm în mod direct, ceea ce ne aduce aminte că nu facem nicio curățare. Utilizând metoda mai puțin rafinată <span class="keystroke">ctrl-c</span> pentru a opri firul principal, toate celelalte fire sunt întrerupte brusc de asemenea, chiar și atunci când sunt pe punctul de a servi o cerere.

În pasul următor, vom implementa trăsătura `Drop` pentru a invoca `join` pe fiecare din fire din pool, astfel încât să își finalizeze cererile în curs de procesare înainte de a se închide. După aceea, vom crea un mecanism prin care firele de execuție să fie informate că trebuie să se oprească din a accepta cereri noi și să se închidă. Pentru a vedea acest cod în practică, vom ajusta serverul nostru să accepte doar două cereri înainte de a-și închide în mod ordonat pool-ul de fire.

### Implementarea trăsăturii `Drop` pe `ThreadPool`

Să începem prin implementarea trăsăturii `Drop` pentru pool-ul nostru de fire de execuție. Când acesta este eliminat, este esențial ca toate firele să fie reunite pentru a asigura finalizarea muncii lor. Listarea 20-22 ilustrează o primă tentativă de implementare pentru `Drop`; Codul prezentat încă nu va funcționa cum trebuie.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/listing-20-22/src/lib.rs:here}}
```

<span class="caption">Listarea 20-22: Procedura de unire a fiecărui fir odată cu ieșirea pool-ului de fire din domeniul de vizibilitate</span>

În primul rând, iterăm prin fiecare `worker` din pool-ul de thread-uri. Utilizăm `&mut` deoarece `self` este o referință mutabilă și avem nevoie să modificăm `worker`. Pentru fiecare `worker`, vom afișa un mesaj care anunță că acel `worker` specific este pe cale de a se opri, după care apelăm metoda `join` pe firul respectivului `worker`. Dacă apelul la `join` eșuează, utilizăm `unwrap` pentru a forța Rust să genereze panică și să realizeze o închidere forțată.

Acesta este mesajul de eroare pe care îl obținem la compilarea codului:

```console
{{#include ../listings/ch20-web-server/listing-20-22/output.txt}}
```

Eroarea indică faptul că nu putem apela `join` deoarece avem doar un împrumut mutabil pentru fiecare `worker` și `join` își asumă posesiunea argumentului pe care îl primește. Pentru a depăși această problemă, este necesar să mutăm firul din interiorul instanței `Worker` ce deține `thread`, permițând astfel `join` să consume firul. Acest lucru a fost implementat în Listarea 17-15: dacă `Worker` are un `Option<thread::JoinHandle<()>>`, putem folosi metoda `take` pe `Option` pentru a extrage valoarea din varianta `Some` și a lăsa locul ocupat de `None`. Pe scurt, un `Worker` activ va avea o variantă `Some` în `thread`, iar când dorim să curățăm un `Worker`, schimbăm `Some` cu `None`, făcând astfel ca `Worker` să nu mai aibă un fir activ.

Prin urmare, știm că dorim să actualizăm definiția lui `Worker` în felul următor:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/no-listing-04-update-worker-definition/src/lib.rs:here}}
```

Să ne sprijinim pe compilator pentru a identifica și alte segmente care necesită schimbări. Examinând acest cod, ne confruntăm cu două erori:

```console
{{#include ../listings/ch20-web-server/no-listing-04-update-worker-definition/output.txt}}
```

Să ne ocupăm de a doua eroare, care ne îndreaptă atenția către codul de la sfârșitul lui `Worker::new`; trebuie să plasăm valoarea `thread` în `Some` atunci când construim un nou `Worker`. Efectuați următoarele ajustări pentru a remedia această eroare:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/no-listing-05-fix-worker-new/src/lib.rs:here}}
```

Prima eroare se regăsește în implementarea noastră `Drop`. Anterior, am indicat intenția de a folosi `take` pe valoarea `Option` pentru a muta `thread` din `worker`. Modificările următoare sunt necesare pentru a realiza aceasta:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/no-listing-06-fix-threadpool-drop/src/lib.rs:here}}
```

Așa cum am explicat în Capitolul 17, metoda `take` pe `Option` scoate varianta `Some` și lasă `None` în locul ei. Utilizăm `if let` pentru a destructura `Some` și a accesa firul; după care invocăm `join` pe acel `thread`. Dacă valoarea lui `thread` a unui `worker` este deja `None`, asta înseamnă că firul respectiv a fost deja curățat și atunci nu se întâmplă nimic în acea situație.

### Semnalizarea firelor pentru a înceta acceptarea sarcinilor

După toate modificările efectuate, codul nostru se compilează fără avertismente. Totuși, partea negativă este că acest cod nu funcționează încă așa cum ne-am propus. Elementul cheie este logica din închiderea executată de firele de execuție ale instanțelor `Worker`: acum când apelăm `join`, acest lucru nu va opri firele deoarece acestea execută o buclă `loop` interminabilă în căutarea de sarcini. Dacă încercăm să distrugem `ThreadPool` cu implementarea curentă a metodei `drop`, firul principal va aștepta la nesfârșit ca primul fir să finalizeze procesarea.

Pentru a corecta această problemă, trebuie să modificăm implementarea metodei `drop` în `ThreadPool` și apoi să ajustăm bucla din `Worker`.

În primul rând, vom schimba implementarea `drop` pentru `ThreadPool` pentru a scăpa explicit de `sender` înainte de a aștepta finalizarea firelor. Listarea 20-23 ne arată schimbările aduse `ThreadPool` pentru a renunța explicit la `sender`. Utilizăm aceeași tehnică `Option` și `take` cum am făcut în cazul firului, pentru a putea muta `sender` în afara obiectului `ThreadPool`:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/listing-20-23/src/lib.rs:here}}
```

<span class="caption">Listarea 20-23: Ștergerea explicită a `sender` înainte de a face `join` la firele din instanțele `worker`</span>

Renunțarea la `sender` conduce la închiderea canalului, ceea ce semnalează faptul că nu se vor mai trimite alte mesaje. Atunci când se întâmplă asta, toate apelurile la `recv` efectuate de instanțele `worker` în bucla infinită vor rezultat într-o eroare. În Listarea 20-24, modificăm bucla `Worker` astfel încât să iasă elegant din buclă în acest caz, ceea ce înseamnă că firele de execuție se vor încheia odată ce implementarea `drop` pentru `ThreadPool` inițiază `join` pe acestea.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-24/src/lib.rs:here}}
```

<span class="caption">Listarea 20-24: Ieșirea explicită din buclă când `recv` returnează o eroare</span>

Pentru a ilustra acest cod în practică, să modificăm funcția `main` pentru a accepta doar două cereri înainte de a încheia activitatea serverului într-un mod ordonat, așa cum arată Listarea 20-25.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/listing-20-25/src/main.rs:here}}
```

<span class="caption">Listarea 20-25: Încheierea activității serverului după procesarea a două cereri, ieșind din buclă</span>

În practică, nu ar fi ideal ca un server web să se oprească după procesarea a doar două cereri. Acest exemplu are scopul de a arăta că procedura de închidere și curățare funcționează corespunzător.

Metoda `take` este definită în trăsătura `Iterator` și limitează iterarea la primele două elemente, cel mult. `ThreadPool` va ieși din domeniul de aplicare la sfârșitul funcției `main`, ceea ce va declanșa executarea implementării `drop`.

Pornește serverul cu comanda `cargo run` și efectuează trei cereri. La cea de-a treia cerere, ar trebui să se întâmpine o eroare și în terminal ar trebui să ai o ieșire similară cu următoarea:

<!-- manual-regeneration
cd listings/ch20-web-server/listing-20-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

S-ar putea să vezi o ordonare diferită a instanțelor de `worker` și a mesajelor afișate. Vedem cum funcționează acest cod prin intermediul mesajelor: instanțele 0 și 3 au procesat primele două cereri. Serverul a încetat de a mai accepta conexiuni după al doilea client, și implementarea `Drop` a `ThreadPool` a început să se activeze chiar înainte ca `worker`-ul 3 să își înceapă sarcina. Eliminând `sender`-ul, se deconectează toți `worker`-ii și li se transmite să se oprească. Fiecare `worker` afișează un mesaj în momentul deconectării, după care pool-ul de fire invocă `join` pentru a aștepta finalizarea fiecărui fir de execuție de `worker`.

Putem observa un detaliu interesant în această execuție: după ce `ThreadPool` a eliminat `sender`-ul, înainte ca vreun `worker` să întâmpine o eroare, am încercat să facem join pe `worker`-ul 0. Instanța 0 nu primise încă vreo eroare de la `recv`, deci firul principal a fost blocat așteptând ca `worker`-ul 0 să-și termine sarcina. Între timp, `worker`-ul 3 a primit o sarcină și apoi toate firele de execuție au întâmpinat o eroare. Odată ce `worker`-ul 0 și-a încheiat treaba, firul principal a așteptat finalizarea celorlalte fire de execuție ale `worker`-ilor. Toți ieșiră deja din buclele lor și s-au oprit.

Felicitări! Am completat proiectul nostru; acum avem la dispoziție un server web de bază care utilizează un pool de fire de execuție pentru a răspunde asincron. Suntem capabili să realizăm o închidere ordonată a serverului, care finalizează toate firele din pool.

Iată codul complet pentru referință:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/no-listing-07-final-code/src/main.rs}}
```

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-07-final-code/src/lib.rs}}
```

Putem adăuga și mai multe îmbunătățiri acestui proiect! Dacă dorești să continui dezvoltarea lui, iată câteva sugestii:

* Completează documentația pentru `ThreadPool` și metodele sale publice.
* Creează teste pentru a verifica funcționalitățile bibliotecii.
* Înlocuiește apelurile la `unwrap` cu mecanisme de gestionare a eroarelor mai solide.
* Folosește `ThreadPool` pentru a executa o sarcină diferită de procesarea cererilor web.
* Explorează un crate pentru thread pool de pe [crates.io](https://crates.io/) și construiește un server web similar folosind acel crate. Compară apoi API-ul și nivelul de robustețe cu thread pool-ul pe care l-am creat noi.

## Sumar

Bravo! Ai parcurs cartea până la capăt! Îți mulțumim că ai acceptat să ni te alături în acest periplu al limbajului Rust. Acum ești pregătit să demarezi propriile proiecte Rust și să contribui la proiectele altora. Reține că avem o comunitate disponibilă de Rustaceani care te așteaptă cu brațele deschise să te ajute în fața oricăror provocări pe care le poți întâlni în drumul tău cu Rust.
