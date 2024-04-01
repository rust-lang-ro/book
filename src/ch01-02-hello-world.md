## Salut, lume!

Acum că ai instalat Rust, este timpul să scrii primul tău program în Rust. Tradițional, când învățăm un limbaj nou, scriem un mic program care afișează textul `Salut, lume!` pe ecran. Așadar, vom face la fel și aici!

> Notă: Această carte presupune cunoștințe de bază a liniei de comandă. 
> Rust nu are cerințe specifice despre editarea, uneltele tale sau locul unde
> se află codul tău, așa că dacă preferi să folosești un mediu de dezvoltare
> integrat (IDE) în locul liniei de comandă, nu ezita să folosești IDE-ul tău
> preferat. Multe IDE-uri au acum un anumit grad de suport Rust; verifică
> documentația IDE-ului pentru detalii. Echipa Rust s-a concentrat pe
> facilitarea unui  suport IDE excelent prin `rust-analyzer`. 
> Vezi [Anexa D][devtools] <!-- ignore --> pentru mai multe detalii.

### Crearea unui directoriu pentru proiect

O să începem prin a crea un directoriu unde să îți stochezi codul Rust. Pentru Rust nu contează unde este situat codul tău, însă pentru exercițiile și proiectele din această carte, recomandăm crearea unui directoriu *projects* în directoriu tău home și să menții toate proiectele acolo.

Deschide un terminal și execută următoarele comenzi pentru a crea un directoriu *projects* și un directoriu pentru proiectul "Hello, world!" în cadrul directoriu *projects*.

Pentru Linux, macOS și PowerShell pe Windows, execută acestea:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Pentru CMD pe Windows, introdu asta:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

### Scrierea și rularea unui program Rust

În continuare, creează un nou fișier sursă și numește-l *main.rs*. Fișierele Rust se termină întotdeauna cu extensiunea *.rs*. Dacă folosești mai mult de un cuvânt în numele fișierului tău, convenția este să folosești un underscore pentru a le separa. De exemplu, folosește *hello_world.rs* în loc de *helloworld.rs*.

Acum deschide fișierul *main.rs* pe care tocmai l-ai creat și introdu codul din Listarea 1-1.

<span class="filename">Numele fișierului: main.rs</span>

```rust
fn main() {
    println!("Salut, lume!");
}
```

<span class="caption">Listarea 1-1: Un program care afișează `Salut, lume!`</span>

Salvează fișierul și întoarce-te la fereastra terminalului în directorul *~/projects/hello_world*. Pe Linux sau macOS, introdu următoarele comenzi pentru a compila și a rula fișierul:

```console
$ rustc main.rs
$ ./main
Salut, lume!
```

Pe Windows, introdu comanda `.\main.exe` în loc de `./main`:

```powershell
> rustc main.rs
> .\main.exe
Salut, lume!
```

Indiferent de sistemul tău de operare, `Salut, lume!` ar trebui să fie afișat în terminal. Dacă nu vezi acest rezultat, verifică din nou partea de [„Depanare”][troubleshooting]<!-- ignore --> a secțiunii de instalare pentru a obține ajutor.

Dacă `Salut, lume!` a apărut, felicitări! Ai scris oficial un program Rust. Asta te face un programator Rust - bun venit!

### Anatomia unui program Rust

Să revizuim în detaliu acest program "Salut, lume!". Iată prima piesă a puzzle-ului:

```rust
fn main() {

}
```

Aceste linii definesc o funcție numită `main`. Funcția `main` este specială: este întotdeauna primul cod care rulează în fiecare program Rust executabil. Aici, prima linie declară o funcție numită `main` care nu are parametri și nu returnează nimic. Dacă ar fi existat parametri, aceștia ar fi fost specificați între parantezele `()`.

Corpul funcției este înconjurat de `{}`. Rust necesită acolade în jurul tuturor corpurilor de funcție. Este considerat un stil bun să plasezi acolada de deschidere pe aceeași linie cu declarația funcției, adăugând un spațiu între acestea.

> Notă: Dacă vrei să urmezi un stil standard în toate proiectele tale Rust,
> poți folosi un utilitar de formatare automată denumit `rustfmt` pentru a-ți
> formata codul într-o manieră specifică (vei găsi mai multe detalii despre
> `rustfmt` în [Anexa D][devtools]<!-- ignore -->). Acest utilitar este inclus
> de echipa Rust în distribuția standard de Rust, la fel ca `rustc`, așa că cel
> mai probabil este deja instalat pe computerul tău!

Corpul funcției `main` conține următorul cod:

```rust
    println!("Salut, lume!");
```

Această linie efectuează toată munca în acest program simplu: tipărește textul pe ecran. Sunt patru detalii importante de observat aici.

În primul rând, stilul Rust este de a indenta cu patru spații, nu cu un tab.

În al doilea rând, `println!` apelează un macro Rust. Dacă ar fi apelat o funcție în loc, ar fi fost introdus ca `println` (fără `!`). Vom discuta macrourile Rust în detaliu în Capitolul 19. Deocamdată, trebuie doar să știi că utilizarea unui `!` înseamnă că apelezi un macro în locul unei funcții normale și că macrourile nu urmează întotdeauna aceleași reguli ca funcțiile.

În al treilea rând, observăm string-ul "Salut, lume!". Noi trecem acest string ca un argument către `println!`, iar string-ul este tipărit pe ecran.

În al patrulea rând, încheiem linia cu un punct și virgulă (`;`), care indică faptul că această expresie s-a încheiat și următoarea este gata să înceapă. Cele mai multe linii de cod Rust se încheie cu un punct și virgulă.

### Compilarea și rularea sunt etape separate

Tocmai ai rulat un program nou creat, așadar să examinăm fiecare pas în proces.

Înainte de a rula un program Rust, trebuie să îl compilezi folosind compilatorul Rust, introducând comanda `rustc` în consolă și trecând numele fișierului tău sursă, astfel:

```console
$ rustc main.rs
```

Dacă ai experiență cu C sau C++, vei observa că acest lucru este similar cu `gcc` sau `clang`. După o compilare reușită, Rust generează un executabil binar.

Pe Linux, macOS, și în PowerShell pe Windows, poți vedea executabilul introducând comanda `ls` în consolă:

```console
$ ls
main  main.rs
```

Pe Linux și macOS, vei vedea două fișiere: 'main' și 'main.rs'. Pe Windows, folosind PowerShell sau CMD, vei vedea, de asemenea, fișierul 'main.exe' și un fișier de depanare cu extensia *.pdb*, alături de *main.rs*. Cu CMD pe Windows, ai introduce următoarea comandă:

```cmd
> dir /B %= opțiunea /B indică doar numele fișierelor =%
main.exe
main.pdb
main.rs
```

Aceasta afişează fișierul sursă cu extensia *.rs*, fișierul executabil (*main.exe* pe Windows, dar *main* pe toate celelalte platforme), și, atunci când folosești Windows, un fișier care conține informații de depanare cu extensia *.pdb*. De aici, rulezi fișierul *main* sau *main.exe*, astfel:

```console
$ ./main # sau .\main.exe pe Windows
```

Dacă *main.rs* este programul tău „Salut, lume!”, această linie va afișa în consolă `Salut, lume!`.

Dacă ai început cu un limbaj dinamic, așa cum sunt Ruby, Python sau JavaScript, este posibil să nu-ți fie clar faptul de a compila și rula un program în pași separați. Rust este un limbaj *compilat înainte de execuție*, ceea ce înseamnă că poți să compilezi un program și să dai executabilul cuiva altcuiva, și aceștia vor putea să îl ruleze chiar și fără să aibă Rust instalat. În schimb, dacă dai cuiva un fișier *.rb*, *.py*, sau *.js*, persoana respectivă trebuie să aibă instalată o variantă corespunzătoare de Ruby, Python sau JavaScript. Cu toate acestea, în limbaje precum Ruby, Python sau JavaScript, compilezi și rulezi programul tău folosind o singură comandă. Fiecare decizie în designul limbajului de programare reprezintă un compromis.

Compilarea doar cu `rustc` este adecvată pentru programele simple, dar pe măsură ce proiectul tău se extinde, vei dori să controlezi toate opțiunile și să îți facilitezi partajarea codului. În continuare, vom explora utilizarea utilitarului Cargo, care ne va sprijini în scrierea programelor Rust pentru scenarii din lumea reală.

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.md
