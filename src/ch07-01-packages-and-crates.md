## Pachete și crate-uri

Începem explorarea sistemului de module cu pachete și crate-uri.

Un *crate* reprezintă unitatea minimă de cod pe care compilatorul Rust o procesează la un moment dat. Chiar dacă alegi să rulezi `rustc` în loc de `cargo` și transmiți un singur fișier sursă (așa cum am făcut în capitolul 1, la secțiunea "Scrierea și Rularea unui Program în Rust"), compilatorul tratează acel fișier ca fiind un crate. Un crate poate include module, iar aceste module pot fi definite în alte fișiere care sunt compilate împreună cu crate-ul, așa cum vom vedea în secțiunile următoare.

Există două tipuri de crate-uri: crate-uri binare și crate-uri de bibliotecă. *Crate-urile binare* sunt programe pe care le poți compila în fișiere executabile pe care poți să le rulezi, precum ar fi un program de linie de comandă sau un server. Fiecare crate binar trebuie să aibă o funcție numită `main` care definește comportamentul său atunci când este rulat. Toate crate-urile pe care le-am creat până acum au fost crate-uri binare.

*Crate-urile de bibliotecă* nu dispun de o funcție `main` și nu se compilează sub formă de executabil. Acestea conțin funcționalități concepute pentru a fi utilizate în comun de diferite proiecte. De pildă, crate-ul `rand`, pe care l-am utilizat în [Capitolul 2][rand]<!-- ignore -->, furnizează funcționalitatea de a genera numere aleatorii. De cele mai multe ori, când utilizatorii de Rust spun “crate”, se referă la crate-uri de bibliotecă. De asemenea, ei folosesc termenul "crate" alternativ cu conceptul general de "bibliotecă" în programare.

*Radacina unui crate* este un fișier sursă principal, punctul de start al compilatorului Rust. El alcătuiește modulul rădăcină al crate-ului tău (vom discuta în detaliu despre module în secțiunea [„Definirea modulelor pentru controlul domeniului de vizibilitate și a confidențialității”][modules]<!-- ignore -->).

Un *pachet* este un ansamblu ce poate cuprinde unul sau mai multe crate-uri, având rolul de a oferi diverse funcționalități. Fiecare pachet conține un fișier numit *Cargo.toml*, care servește drept ghid pentru construcția respectivelor crate-uri. Un exemplu concret este Cargo, care nu este altceva decât un pachet. Acesta include un crate de tip binar utilizat pentru instrumentul de linie de comandă cu care ai lucrat pentru a construi propriul cod. Pachetul Cargo găzduiește și un crate de tip bibliotecă, de care depinde crate-ul binar. Alte proiecte pot recurge la acest crate de tip bibliotecă pentru a folosi aceeași logică pe care o implementează instrumentul de linie de comandă.

Reflectând liber, un pachet are capacitatea de a găzdui oricâte crate-uri binare dorești, cu însă o limitare: poate conține cel mult un singur crate de tip bibliotecă. Este obligatoriu ca un pachet să aibă cel puțin un crate, indiferent dacă este vorba de unul de tip bibliotecă sau de tip binar.

În continuare, vom descrie pașii ce intervin în procesul de creare a unui pachet. Mai întâi, vom introduce comanda `cargo new`:

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

După ce executăm comanda `cargo new`, putem folosi `ls` pentru a vedea ceea ce a generat comanda Cargo. În directoriul proiectului, vom găsi un fișier numit *Cargo.toml*, care ne creează un pachet. Tot acolo, se află și un directoriu *src* ce conține fișierul *main.rs*. 

Dacă deschizi fișierul *Cargo.toml* într-un editor de text, vei observa că nu există nicio referire la *src/main.rs*. De fapt, Cargo urmează o convenție specifică prin care *src/main.rs* reprezintă rădăcina unui crate binar având același nume ca și pachetul. În același timp, Cargo recunoaște că dacă directoriul pachetului conține *src/lib.rs*, înseamnă că pachetul include un crate de bibliotecă cu același nume ca pachetul, iar *src/lib.rs* reprezintă rădăcina acestui tip de crate. Cargo transmite fișierele rădăcină ale crate-urilor către compilatorul `rustc` pentru a construi fie biblioteca, fie binarul.

În exemplul nostru, avem un pachet care include doar *src/main.rs*, ceea ce semnifică faptul că conține un singur crate binar numit `my-project`. Dacă un pachet ar conține atât *src/main.rs* cât și *src/lib.rs*, ar însemna că are două crate-uri: unul binar și unul de bibliotecă, ambele având același nume ca pachetul. Un pachet poate conține mai multe crate-uri binare dacă fișierele lor sunt plasate în directoriul *src/bin*: fiecare fișier reprezentând un crate binar separat.

[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
