## Construirea unui server web cu un singur fir de execuție

Vom începe prin a pune în funcțiune un server web care rulează pe un singur fir de execuție (sau *thread*). Înainte de a ne apuca de treabă, să aruncăm o privire sumară asupra protocoalelor implicate în construcția serverelor web. Detaliile profunde ale acestor protocoale depășesc scopul acestei cărți, dar o înțelegere elementară ne va furniza cunoștințele necesare.

Principalele două protocoale folosite în serverele web sunt *Hypertext Transfer Protocol* *(HTTP)* și *Transmission Control Protocol* *(TCP)*. Acestea sunt protocoale de tip *request-response*, prin care un *client* inițiază solicitări iar un *server* le recepționează și furnizează un răspuns corespunzător clientului. Conținutul specific al cererilor și răspunsurilor este stipulat de către aceste protocoale.

TCP este protocolul de fundament, ce explică mecanismele de transfer al informațiilor între servere, fără a detalia natura acestor informații. Pe de altă parte, HTTP se suprapune peste TCP, stabilind conținutul specific al solicitărilor și răspunsurilor. Deși este posibilă utilizarea HTTP în conjuncție cu alte protocoale, în majoritatea covârșitoare a situațiilor, HTTP își expediază datele prin TCP. Vom lucra direct cu octeții nealterați ai cererilor și răspunsurilor oferite de TCP și HTTP.

### Ascultarea unei conexiuni TCP

Serverul nostru web trebuie să fie capabil să asculte o conexiune TCP, iar aceasta este prima parte la care ne vom concentra. Biblioteca standard ne oferă un modul `std::net` prin care putem realiza acest lucru. Să creăm un nou proiect în modul obișnuit:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Introdu acum codul din Listarea 20-1 în fișierul *src/main.rs* ca punct de pornire. Acest cod va asculta la adresa locală `127.0.0.1:7878` pentru fluxurile TCP care sosesc. Când un astfel de flux e detectat, programul va afișa mesajul `Connection established!`.

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-01/src/main.rs}}...
```

<span class="caption">Listarea 20-1: Ascultarea fluxurilor TCP și afișarea unui mesaj la primirea unui flux</span>

Utilizând `TcpListener`, putem asculta conexiuni TCP la adresa `127.0.0.1:7878`. În această adresă, partea dinaintea celor două puncte `:` este o adresă IP ce reprezintă computerul tău (fiind același IP pe toate computerele, el nu reprezintă un IP specific autorilor cărții), iar `7878` este portul utilizat. Am ales acest port din două motive: în mod normal HTTP nu operează pe acest port, așadar este improbabil ca serverul nostru să între în conflict cu alte servere web care ar putea fi active pe computerul tău, și 7878 reprezintă cuvântul *rust* pe tastatura unui telefon.

Funcția `bind`, în acest context, funcționează similar cu funcția `new` în sensul că va returna o nouă instanță `TcpListener`. Se numește `bind` (engl. a lega) pentru că în terminologia de rețea, procesul de a asculta un port este cunoscut ca „asocierea cu un port” ("binding to a port").

Funcția `bind` returnează un `Result<T, E>`, sugerând că există posibilitatea ca asocierile să fie nereușite. De exemplu, pentru a asculta pe portul 80 sunt necesare privilegii de administrator (non-administratorii pot asculta doar pe porturi mai mari de 1023), deci dacă am încerca să ascultăm pe portul 80 fără a fi administratori, asocierea nu ar reuși. Asocierea ar eșua de asemenea dacă am rula două instanțe ale programului nostru și astfel două programe ar asculta același port. Întrucât construim un server simplu doar în scop educațional, nu ne vom preocupa de gestionarea acestor tipuri de erori; în schimb, vom folosi `unwrap` pentru a opri programul dacă ele apar.

Metoda `incoming` de pe `TcpListener` ne întoarce un iterator care ne oferă o secvență de fluxuri (mai exact, fluxuri de tip `TcpStream`). Un singur *flux* constituie o conexiune deschisă între client și server. O *conexiune* este denumirea pentru tot procesul de cerere și răspuns, în care clientul se conectează la server, serverul generează un răspuns și apoi serverul închide conexiunea. Astfel, vom citi din `TcpStream` pentru a afla ce a trimis clientul și vom scrie răspunsul nostru în flux pentru a transmite datele înapoi clientului. Per total, acest ciclu `for` va procesa fiecare conexiune pe rând, producând o serie de fluxuri pe care trebuie să le gestionăm.

Pentru moment, gestionarea fluxului se rezumă la utilizarea funcției `unwrap` pentru a opri programul nostru dacă fluxul întâmpină erori; dacă nu există erori, programul va afișa un mesaj. Vom extinde funcționalitatea pentru situația de succes în următoarea listare. Motivul pentru care s-ar putea să primim erori de la metoda `incoming` atunci când un client încearcă să se conecteze la server este faptul că nu iterăm propriu-zis peste conexiuni, ci peste *încercări de conectare*. Conexiunea poate eșua din diverse motive, multe fiind specifice sistemului de operare. De exemplu, multe sisteme de operare limitează numărul de conexiuni deschise simultan pe care le pot suporta; încercările de a stabili noi conexiuni peste acest număr vor rezulta într-o eroare până când anumite conexiuni deschise sunt închise.

Să încercăm să executăm acest cod! Rulează `cargo run` în terminal, apoi deschide *127.0.0.1:7878* într-un browser web. Browserul ar trebui să afișeze un mesaj de eroare cum ar fi "Conexiunea s-a resetat", deoarece serverul, în acest moment, nu trimite niciun fel de date înapoi. Dar, privind în terminal, ar trebui să observi mai multe mesaje care s-au afișat când browserul s-a conectat la server!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Uneori, ai putea observa că sunt afișate mai multe mesaje pentru o singură cerere făcută de browser; cauza ar putea fi aceea că browserul solicită atât pagina, cât și alte resurse, precum iconița *favicon.ico* care apare în tab-ul browserului.

De asemenea, este posibil ca browserul să încerce să se conecteze de mai multe ori la server deoarece serverul nu transmite nicio dată. Atunci când `stream` iese din domeniul de vizibilitate și este abandonat la sfârșitul buclei, conexiunea se închide automat, acesta fiind parte a comportamentului metodei `drop`. Browserul poate gestiona astfel de conexiuni închise prin efectuarea de noi încercări, presupunând că problema ar putea fi una temporară. Elementul cheie este că am obținut cu succes un descriptor pentru o conexiune TCP!

Nu uita să oprești programul apăsând <span class="keystroke">ctrl-c</span> când ai finalizat rularea unei versiuni particulare de cod. Ulterior, repornește programul utilizând comanda `cargo run` după aplicarea fiecărui set de schimbări în cod, pentru a te asigura că rulezi versiunea actualizată a codului.

### Citirea cererii

Să implementăm funcționalitatea pentru citirea cererii din partea browserului! Pentru a separa responsabilitățile de a obține mai întâi o conexiune și apoi a face o acțiune cu acea conexiune, vom iniția o nouă funcție de procesare a conexiunilor. În această nouă funcție `handle_connection`, vom citi date din fluxul TCP și le vom afișa pentru a vedea informațiile trimise de browser. Actualizează codul să corespundă cu Listarea 20-2.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-02/src/main.rs}}
```

<span class="caption">Listarea 20-2: Citirea datelor din `TcpStream` și afișarea lor</span>

Introducem în context `std::io::prelude` și `std::io::BufReader` pentru a obține acces la trăsăturile și tipurile care ne permit să citim și să scriem pe flux. În bucla `for` din funcția `main`, în loc să afișăm un mesaj care să confirme că s-a realizat o conexiune, acum invocăm noua funcție `handle_connection` și îi pasăm `stream`-ul.

În interiorul funcției `handle_connection`, creăm o nouă instanță de `BufReader` care cuprinde o referință mutabilă către `stream`. `BufReader` adaugă un buffer prin gestionarea apelurilor la metodologiile trăsăturii `std::io::Read`.

Definim o variabilă numită `http_request` pentru a acumula liniile cererii pe care browserul le trimite către serverul nostru. Specificăm că dorim să adunăm aceste linii într-un vector prin adăugarea adnotării de tip `Vec<_>`.

`BufReader` implementează trăsătura `std::io::BufRead`, care pune la dispoziție metoda `lines`. Metoda `lines` furnizează un iterator de `Result<String, std::io::Error>`, despărțind fluxul de date la fiecare întâlnire a unui octet de sfârșit de linie. Pentru a extrage fiecare `String`, aplicăm `map` și `unwrap` la fiecare `Result`. `Result` poate fi o eroare dacă datele nu sunt UTF-8 valide sau dacă a apărut o problemă în timpul citirii de pe flux. Într-un context de producție, ar trebui abordate aceste erori mai elegant, dar optăm pentru simplificare prin oprirea programului în cazul unei erori.

Browserul indică sfârșitul unei cereri HTTP trimițând două caractere de tip newline în succesiune, așadar pentru a extrage o cerere din flux, citim liniile până când întâlnim o linie care este un string gol. După ce am adunat liniile în vector, le afișăm utilizând formatarea de debug, astfel încât să putem analiza instrucțiunile pe care browserul web le trimite serverului nostru.

Să încercăm acest cod! Pornim programul și facem o cerere în browserul web din nou. Observă că în browser vom întâlni în continuare o pagină de eroare, dar ieșirea programului nostru în terminal va arăta acum similar cu acesta:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

În funcție de browserul tău, este posibil să obții un output ușor diferit. Acum că afișăm datele cererii, putem determina motivele pentru care primim multiple conexiuni dintr-o singură cerere de browser analizând calea după `GET` din prima linie a cererii. Dacă conexiunile repetate sunt toate pentru calea `*/*`, înseamnă că browserul încearcă să acceseze `*/*` de mai multe ori deoarece nu primește un răspuns de la programul nostru.

În continuare să analizăm aceste date pentru a înțelege mai bine ce solicită browserul de la programul nostru.

### Examinăm mai detaliat o cerere HTTP

HTTP este un protocol bazat pe text și o cerere are formatul următor:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

Prima linie reprezintă *linia de cerere* și conține informații despre ce solicită clientul. Prima parte a liniei de cerere precizează *metoda* utilizată, precum `GET` sau `POST`, ce descrie cum clientul efectuează această cerere. Clientul nostru a folosit o cerere de tip `GET`, ceea ce semnifică că el solicită informații.

Următorul segment al liniei de cerere este */*, care indică *Uniform Resource Identifier (URI)* pe care clientul dorește să-l acceseze. Un URI este similar, dar nu identic cu un *Uniform Resource Locator (URL)*. Distincția dintre URI și URL nu este semnificativă în contextul acestui capitol, însă specificația HTTP utilizează termenul URI, astfel putem înlocui mental termenul URL pentru URI.

Ultimul segment este versiunea HTTP folosită de client și apoi linia de cerere se încheie cu o *secvență CRLF*. (CRLF înseamnă *carriage return* și *line feed*, termeni din epoca mașinilor de scris!) Secvența CRLF poate fi, de asemenea, reprezentată prin `\r\n`, unde `\r` este carriage return și `\n` este line feed. Secvența CRLF separă linia de cerere de restul datelor cererii. De remarcat că atunci când CRLF este afișat, începe o linie nouă în loc să vedem `\r\n`.

Analizând datele liniei de cerere pe care le-am obținut până în acest moment prin rularea programului nostru, observăm că `GET` este metoda, */* este URI-ul solicitat și `HTTP/1.1` este versiunea utilizată.

După linia de cerere, următoarele linii începând cu `Host:` și continuând sunt anteturile, sau header-ele. Cererile de tip `GET` nu includ un corp.

Încearcă să faci o cerere folosind un alt browser sau solicitând o adresă diferită, de exemplu, *127.0.0.1:7878/test*, pentru a vedea cum se modifică datele cererii.

Acum că înțelegem ce solicită browser-ul, să-i răspundem cu unele date!

### Scrierea unui răspuns

Pornim să implementăm transmiterea de date ca răspuns la o solicitare din partea clientului. Răspunsurile urmează acest format:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

Prima linie, cunoscută ca *linie de status*, include versiunea HTTP utilizată în răspuns, un cod de status numeric ce sumarizează rezultatul solicitării și o frază explicativă ce oferă o descriere în text a codului de status. După secvența CRLF, urmează anteturile, încă o secvență CRLF și corpul răspunsului.

Avem aici un exemplu de răspuns folosind versiunea HTTP 1.1, cu un cod de status 200, o frază OK explicativă, fără anteturi și fără corp:

```text
HTTP/1.1 200 OK\r\n\r\n
```

Status code 200 reprezintă răspunsul standard pentru o operațiune de succes. Acest text este un mic exemplu de răspuns HTTP reușit. Să trimitem acest răspuns în fluxul nostru ca reacție la o solicitare îndeplinită cu succes! În funcția `handle_connection`, eliminăm comanda `println!` care afișa datele solicitării și o înlocuim cu codul din Listarea 20-3.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-03/src/main.rs:here}}
```

<span class="caption">Listarea 20-3: Crearea unui răspuns HTTP succint și eficient în fluxul de date</span>

Prima linie nouă stabilește variabila `response` care conține datele mesajului nostru de succes. În continuare, folosim `as_bytes` pe `response` pentru a converti string-ul în octeți. Metoda `write_all` de pe `stream` acceptă un `&[u8]` și trimite acești octeți direct prin conexiune. Folosim `unwrap` pentru a gestiona eventualele erori ce pot apărea în timpul operației `write_all`, ca și în cazurile anterioare. Desigur, într-o aplicație reală ar fi necesară implementarea unei gestionări adecvate a erorilor.

Cu aceste modificări implementate, să rulăm codul și să inițiem o solicitare. Cum nu vom mai afișa date în terminal, singura ieșire vizibilă va fi cea de la Cargo. Accesând într-un browser adresa *127.0.0.1:7878*, ar trebui să apară o pagină albă, semn că nu mai există nicio eroare. Felicitări, tocmai ai realizat manual o solicitare HTTP și ai trimis un răspuns corespunzător!

### Returnarea de HTML real

Să dezvoltăm funcționalitatea de a returna mai mult decât o simplă pagină goală. Creează noul fișier *hello.html* în rădăcina directoriului tău de proiect, nu în
directorul *src*. Poți introduce orice cod HTML dorești; în Listarea 20-4 este prezentată o variantă.

<span class="filename">Numele fișierului: hello.html</span>

```html
{{#include ../listings/ch20-web-server/listing-20-05/hello.html}}
```

<span class="caption">Listarea 20-4: Un exemplu de fișier HTML pentru a fi returnat ca răspuns</span>

Acesta este un document minimal HTML5 cu un titlu și ceva text. Pentru a-l returna de pe server atunci când se primește o cerere, vom modifica `handle_connection` conform celor prezentate în Listarea 20-5 pentru a citi fișierul HTML, adăugându-l la răspuns în calitate de corp, apoi trimițându-l.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-05/src/main.rs:here}}
```

<span class="caption">Listarea 20-5: Trimiterea conținutului fișierului *hello.html* ca parte a corpului răspunsului</span>

Am introdus `fs` în instrucțiunea `use` pentru a aduce modulul de sistem de fișiere din biblioteca standard în sfera noastră de aplicabilitate. Codul pentru citirea conținuturilor unui fișier într-un string ți-ar putea fi cunoscut; l-am utilizat în Capitolul 12 când am citit conținuturile unui fișier pentru proiectul nostru I/O, prezentate în Listarea 12-4.

Apoi, utilizăm `format!` pentru a include conținutul fișierului ca parte a corpului răspunsului de succes. Pentru a asigura un răspuns HTTP valid, includeem antetul `Content-Length`, care este stabilit la dimensiunea corpului nostru de răspuns, în acest caz dimensiunea lui `hello.html`.

Execută acest cod folosind `cargo run` și accesează *127.0.0.1:7878* în browser; ar trebui să îți vezi codul HTML afișat!

În momentul de față, ignorăm datele din cererea `http_request` și pur și simplu returnăm conținutul fișierului HTML indiferent de alți parametri. Acest lucru înseamnă că dacă încerci să accesezi *127.0.0.1:7878/ceva-altceva* în browserul tău, vei primi același răspuns HTML. În stadiul actual, serverul nostru este destul de limitat și nu efectuează operațiunile pe care le realizează majoritatea serverelor web. Vrem să personalizăm răspunsurile noastre în funcție de cereri și să returnăm fișierul HTML doar pentru cereri corect formulate spre */*.

### Validarea cererii și răspuns selectiv

Deocamdată, serverul nostru web va returna codul HTML din fișier indiferent de solicitarea clientului. Să introducem o funcționalitate care să verifice dacă navigatorul web solicită */* înainte de a oferi fișierul HTML și să trimitem o eroare dacă se solicită altceva. Pentru acest lucru, este necesar să modificăm `handle_connection`, așa cum e ilustrat în Listarea 20-6. Acest cod nou compară conținutul cererii primite cu formatul unei cereri pentru */* și utilizează construcții `if` și `else` pentru a gestiona diferit cererile.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-06/src/main.rs:here}}
```

<span class="caption">Listarea 20-6: Gestionarea diferită a cererilor la */* în comparație cu alte cereri</span>

Ne vom concentra doar asupra primei linii din cererea HTTP, astfel, în loc să citim întreaga cerere într-un vector, folosim `next` pentru a accesa primul element din iterator. Primul `unwrap` tratează `Option` și oprește execuția programului dacă iteratorul nu conține niciun element. Al doilea `unwrap` se ocupă de `Result`, având același rol ca `unwrap`-ul adăugat în `map` din Listarea 20-2.

În continuare, verificăm dacă `request_line` corespunde cu linia specifică unei cereri GET către calea */*. Dacă este așa, blocul `if` furnizează conținutul fișierului nostru HTML.

Dacă `request_line` *nu* corespunde cu cererea GET către calea */*, atunci e clar că am primit o solicitare diferită. Vom adăuga cod blocului `else` în curând, pentru a răspunde la orice altă cerere.

Execută acest cod acum și accesează *127.0.0.1:7878*; ar trebui să primești codul HTML din *hello.html*. Dacă inițiezi o cerere diferită, de exemplu la *127.0.0.1:7878/ceva-altceva*, te vei confrunta cu o eroare de conexiune similară cu cea întâlnită atunci când codul din Listarea 20-1 și Listarea 20-2 era executat.

Acum să includem codul din Listarea 20-7 în blocul `else` pentru a emite un răspuns cu codul de status 404, semnalând astfel că nu a fost găsit conținutul solicitat. De asemenea, vom returna un HTML pentru o pagină ce va fi afișată în browser, indicând utilizatorului final acest răspuns.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-07/src/main.rs:here}}
```

<span class="caption">Listarea 20-7: Emiterea unui răspuns cu codul de status 404 și o pagină de eroare când se solicită altceva în loc de */*</span>

Aici, linia de status a răspunsului nostru include codul de status 404 și fraza `NOT FOUND`. Corpul răspunsului va reprezenta HTML-ul conținut în fișierul *404.html*. Trebuie să creezi un fișier *404.html* în directoriul fișierului *hello.html* pentru pagina de eroare; ești liber să folosești orice cod HTML vrei sau să preiei exemplul de HTML din Listarea 20-8.

<span class="filename">Numele fișierului: 404.html</span>

```html
{{#include ../listings/ch20-web-server/listing-20-07/404.html}}
```

<span class="caption">Listarea 20-8: Conținut exemplu pentru pagina returnată odată cu răspunsul de 404</span>

După aceste actualizări, rulează din nou serverul tău. Accesarea *127.0.0.1:7878* ar trebui să îți prezinte conținutul din *hello.html*, iar orice altă solicitare, precum *127.0.0.1:7878/foo*, va genera răspunsul HTML de eroare din *404.html*.

### Un strop de refactorizare

La momentul actual, blocurile `if` și `else` conțin mult cod repetitiv: amândouă citesc fișiere și scriu conținutul în flux. Diferențele sunt doar linia de status și numele fișierului. Vom simplifica codul, extrăgând aceste diferențe în linii de `if` și `else` separate, care vor atribui valorile pentru linia de status și numele fișierului la variabile; apoi vom utiliza variabilele pentru a citi fișierul și a scrie răspunsul în mod necondiționat. Listarea 20-9 prezintă codul rezultat după înlocuirea blocurilor mari de `if` și `else`.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-09/src/main.rs:here}}
```

<span class="caption">Listarea 20-9: Refactorizarea blocurilor `if` și `else` pentru a conține numai codul care diferă între cele două cazuri</span>

În prezent, blocurile `if` și `else` returnează valorile potrivite pentru linia de status și numele fișierului sub formă de tuplă; apoi folosim destructurarea pentru a atribui aceste valori variabilelor `status_line` și `filename`, utilizând un pattern în instrucțiunea `let`, după cum a fost discutat în Capitolul 18.

Codul care era duplicat anterior este acum plasat în afara blocurilor `if` și `else` și utilizează variabilele `status_line` și `filename`. Aceasta ne facilitează observarea diferențelor dintre cele două cazuri și înseamnă că avem acum un singur punct unde trebuie să facem modificări dacă dorim să schimbăm modul de citire a fișierelor și de redactare a răspunsurilor. Comportamentul codului din Listarea 20-9 este identic cu cel din Listarea 20-8.

Extraordinar! Deținem acum un server web simplu în aproximativ 40 de linii de cod Rust, care răspunde la o cerere cu o pagină de conținut și la toate celelalte cereri cu un răspuns 404.

Serverul nostru operează momentan într-un singur fir de execuție, ceea ce implică faptul că poate procesa o singură solicitare consecutiv. Să explorăm cum acest aspect poate deveni o problemă simulând câteva cereri lente. Pe urmă, vom îmbunătăți serverul astfel încât să poată gestiona multiple cereri în paralel.
