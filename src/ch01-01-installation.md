## Instalare

Primul pas este să instalezi Rust. Vom descărca Rust folosind `rustup`, un instrument de linie de comandă pentru gestionarea versiunilor Rust și a uneltelor asociate. Vei avea nevoie de o conexiune la internet pentru descărcare.

> Notă: Dacă preferi să nu folosești `rustup` dintr-un motiv oarecare, te rugăm
> să consulți [pagina cu Alte metode de instalare Rust][otherinstall] pentru
> mai multe opțiuni.

Următorii pași instalează cea mai recentă versiune stabilă a compilatorului Rust. Rust oferă garanții de stabilitate care asigură că toate exemplele din carte compatibile cu versiuni anterioare vor rămâne funcționale și pe versiunile Rust ulterioare. Ieșirea s-ar putea să difere ușor între versiuni deoarece Rust îmbunătățește adesea mesajele de eroare și avertismentele. Cu alte cuvinte, orice versiune stabilă mai nouă de Rust pe care o instalezi folosind acești pași ar trebui să funcționeze așa cum este de așteptat cu conținutul acestei cărți.

> ### Notația pentru linia de comandă
>
> De-a lungul acestui capitol și a cărții, vom indica diferite comenzi
> utilizate în terminal. Liniile ce urmează să le tastezi într-un terminal
> încep cu `$`. Nu trebuie să tastezi caracterul `$`; acesta semnalizează
> începutul unei comenzi în terminal. Liniile care nu încep cu `$` reprezintă
> în mod obișnuit output-ul comenzii anterioare. În plus, exemplele destinate
> PowerShell vor utiliza `>` în loc de `$`.

### Instalarea `rustup` pe Linux sau macOS

Dacă folosești Linux sau macOS, deschide un terminal și introduce următoarea comandă:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Comanda descarcă un script și începe instalarea instrumentului `rustup`, care instalează cea mai recentă versiune stabilă a Rust. S-ar putea să ți se ceară parola. Dacă instalarea este reușită, va apărea următoarea linie:

```text
Rust is installed now. Great!
```

De asemenea, va trebui să ai un *linker*, care este un program pe care Rust îl folosește pentru a uni toate afișajele sale compilate într-un singur fișier. Probabil deja ai unul. Dacă primești erori legate de linker, ar trebui să instalezi un compilator C, acesta include de obicei un linker. Un compilator C este util și pentru că unele pachete comune din Rust depind de codul C și vor avea nevoie de un compilator C.

Pe macOS, poți obține un compilator C cu următoarea comandă:

```console
$ xcode-select --install
```

Utilizatorii de Linux ar trebui să instaleze, în general, GCC sau Clang, conform documentației distribuției lor. De exemplu, dacă folosești Ubuntu, poți instala pachetul `build-essential`. 

### Instalarea `rustup` pe Windows

Pe Windows, accesează [https://www.rust-lang.org/tools/install][install] și urmează instrucțiunile pentru instalarea Rust. La un moment dat în procesul de instalare, vei primi un mesaj care îți explică faptul că este necesar să ai și instrumentele de compilare MSVC pentru Visual Studio 2013 sau mai târziu.

Pentru a obține instrumentele de compilare, trebuie să instalezi [Visual Studio 2022][visualstudio]. Când îți va fi solicitat să alegi ce utilități de lucru să instalezi, asigură-te că incluzi:

* „Desktop Development with C++”
* SDK-ul pentru Windows 10 sau 11
* Componenta pachetului de limbă engleză, împreună cu orice alte pachete de limbă după preferință

Restul acestei cărți utilizează comenzi compatibile atât cu *cmd.exe*, cât și cu PowerShell. Dacă există diferențe specifice între acestea, noi vom explica care versiune trebuie utilizată.

### Depanare

Pentru a verifica dacă ai instalat corect Rust, deschide o consolă și introdu această linie:

```console
$ rustc --version
```

Ar trebui să vezi numărul versiunii, hash-ul commit-ului și data commit-ului pentru cea mai recent lansată versiune stabilă, în următorul format:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Dacă vezi aceste informații, ai instalat cu succes Rust! Dacă nu vezi aceste informații, verifică dacă Rust se află în variabila de sistem %PATH%` în felul următor.

În CMD Windows, folosește:

```console
> echo %PATH%
```

În PowerShell, folosește:

```powershell
> echo $env:Path
```

În Linux și macOS, folosește:

```console
$ echo $PATH
```

Dacă totul este corect și Rust tot nu funcționează, există mai multe locuri în care poți obține ajutor. Află cum să iei legătura cu alți Rustaceani (un pseudonim amuzant pe care noi îl folosim) pe [pagina comunității][community].

### Actualizare și dezinstalare

Odată ce Rust este instalat prin intermediul `rustup`, actualizarea la o versiune nou lansată este ușoară. Din terminalul tău, rulează următorul script de actualizare:

```console
$ rustup update
```

Pentru a dezinstala Rust și `rustup`, rulează următorul script de dezinstalare din terminal:

```console
$ rustup self uninstall
```

### Documentație locală

Instalarea Rust include și o copie locală a documentației, astfel încât să o poți citi offline. Rulează `rustup doc` pentru a deschide documentația locală în browserul tău.

De fiecare dată când un tip sau o funcție este furnizată de biblioteca standard și nu ești sigur ce face sau cum să o folosești, apelează la documentația interfeței de programare a aplicațiilor (API) pentru a afla!

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[visualstudio]: https://visualstudio.microsoft.com/downloads/
[community]: https://www.rust-lang.org/community
