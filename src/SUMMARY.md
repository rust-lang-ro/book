# Limbajul de programare Rust

[Limbajul de programare Rust](title-page.md)
[Cuvânt înainte](foreword.md)
[Introducere](ch00-00-introduction.md)

## Să începem

- [Să începem](ch01-00-getting-started.md)
    - [Instalare](ch01-01-installation.md)
    - [Salut, lume!](ch01-02-hello-world.md)
    - [Salut, Cargo!](ch01-03-hello-cargo.md)

- [Programarea unui joc de ghicit](ch02-00-guessing-game-tutorial.md)

- [Concepte comune de programare](ch03-00-common-programming-concepts.md)
    - [Variabile și mutabilitate](ch03-01-variables-and-mutability.md)
    - [Tipuri de date](ch03-02-data-types.md)
    - [Funcții](ch03-03-how-functions-work.md)
    - [Comentarii](ch03-04-comments.md)
    - [Controlul fluxului](ch03-05-control-flow.md)

- [Înțelegerea posesiunii](ch04-00-understanding-ownership.md)
    - [Ce este posesiunea?](ch04-01-what-is-ownership.md)
    - [Referințe și procesul de împrumutare](ch04-02-references-and-borrowing.md)
    - [Tipul Slice](ch04-03-slices.md)

- [Folosirea structurilor pentru organizarea datelor interconectate](ch05-00-structs.md)
    - [Definirea și crearea instanțelor de structuri](ch05-01-defining-structs.md)
    - [Un program exemplu ce utilizează structurile](ch05-02-example-structs.md)
    - [Sintaxa pentru metode](ch05-03-method-syntax.md)

- [Enumerări și potrivirea modelelor](ch06-00-enums.md)
    - [Definirea unei enumerări](ch06-01-defining-an-enumeration.md)
    - [Structura de control `match`](ch06-02-match.md)
    - [Control concis al executării cu `if let`](ch06-03-if-let.md)

## Cultură de bază în Rust

- [Administrarea proiectelor în expansiune cu ajutorul pachetelor, crate-urilor și modulelor](ch07-00-managing-growing-projects-with-packages-crates-and-modules.md)
    - [Pachete și crate-uri](ch07-01-packages-and-crates.md)
    - [Stabilirea modulelor pentru a gestiona domeniul de vizibilitate și confidențialitatea](ch07-02-defining-modules-to-control-scope-and-privacy.md)
    - [Utilizarea căilor pentru a face referire la un element în structura de module](ch07-03-paths-for-referring-to-an-item-in-the-module-tree.md)
    - [Utilizarea cuvântului cheie `use` pentru a aduce căile în domeniul de vizibilitate](ch07-04-bringing-paths-into-scope-with-the-use-keyword.md)
    - [Separarea modulelor în diferite fișiere](ch07-05-separating-modules-into-different-files.md)

- [Colecțiile comune în Rust](ch08-00-common-collections.md)
    - [Păstrarea listelor de valori folosind vectori](ch08-01-vectors.md)
    - [Manipularea textelor codate UTF-8 utilizând string-uri](ch08-02-strings.md)
    - [Utilizarea hash map-urilor pentru a asocia chei cu valori](ch08-03-hash-maps.md)

- [Tratarea erorilor](ch09-00-error-handling.md)
    - [Gestionarea erorilor irecuperabile cu `panic!`](ch09-01-unrecoverable-errors-with-panic.md)
    - [Gestionarea erorilor recuperabile cu `Result`](ch09-02-recoverable-errors-with-result.md)
    - [Să `panic!`-ăm sau nu?](ch09-03-to-panic-or-not-to-panic.md)

- [Tipuri generice, trăsături și durate de viață](ch10-00-generics.md)
    - [Tipuri de date generice](ch10-01-syntax.md)
    - [Traits: definirea comportamentului partajat](ch10-02-traits.md)
    - [Validarea referințelor cu ajutorul lifetimes](ch10-03-lifetime-syntax.md)

- [Testare automatizată în Rust](ch11-00-testing.md)
    - [Cum să scriem teste](ch11-01-writing-tests.md)
    - [Controlul modului în care testele sunt executate](ch11-02-running-tests.md)
    - [Organizarea testării](ch11-03-test-organization.md)

- [Un proiect de I/O: Construirea unei aplicații de linie de comandă](ch12-00-an-io-project.md)
    - [Acceptarea argumentelor liniei de comandă](ch12-01-accepting-command-line-arguments.md)
    - [Citirea fișierului](ch12-02-reading-a-file.md)
    - [Refactorizarea pentru îmbunătățirea modularității și gestionarea erorilor](ch12-03-improving-error-handling-and-modularity.md)
    - [Dezvoltarea funcționalității bibliotecii cu Test-Driven Development](ch12-04-testing-the-librarys-functionality.md)
    - [Interacționând cu variabilele de mediu](ch12-05-working-with-environment-variables.md)
    - [Afișarea mesajelor de eroare pe eroare standard în loc de ieșire standard](ch12-06-writing-to-stderr-instead-of-stdout.md)

## Gândire în Rust

- [Elemente de programare funcțională în Rust: închideri și iteratori](ch13-00-functional-features.md)
    - [Închideri: funcții anonime care captează contextul](ch13-01-closures.md)
    - [Procesarea unei serii de elemente cu ajutorul iteratorilor](ch13-02-iterators.md)
    - [Îmbunătățim proiectul nostru I/O](ch13-03-improving-our-io-project.md)
    - [Comparând performanța buclelor și a iteratorilor](ch13-04-performance.md)

- [Mai multe informații despre Cargo și Crates.io](ch14-00-more-about-cargo.md)
    - [Personalizarea build-urilor cu profile de release](ch14-01-release-profiles.md)
    - [Publicarea unui crate pe crates.io](ch14-02-publishing-to-crates-io.md)
    - [Spațiile de lucru Cargo](ch14-03-cargo-workspaces.md)
    - [Instalarea binarelor cu `cargo install`](ch14-04-installing-binaries.md)
    - [Extinderea Cargo cu Comenzi Personalizate](ch14-05-extending-cargo.md)

- [Pointeri inteligenți](ch15-00-smart-pointers.md)
    - [Utilizarea `Box<T>` pentru a arăta spre date situate pe heap](ch15-01-box.md)
    - [Tratarea pointerilor inteligenți la fel ca referințele obișnuite cu trăsătura `Deref`](ch15-02-deref.md)
    - [Executarea codului în etapa de curățare cu trăsătura `Drop`](ch15-03-drop.md)
    - [`Rc<T>`, pointerul inteligent cu numărare a referențelor](ch15-04-rc.md)
    - [`RefCell<T>` și mutabilitatea internă](ch15-05-interior-mutability.md)
    - [Ciclurile de referințe pot provoca scurgeri de memorie](ch15-06-reference-cycles.md)

- [Concurență fără temeri](ch16-00-concurrency.md)
    - [Utilizarea firelor de execuție pentru a executa cod simultan](ch16-01-threads.md)
    - [Transferul datelor între fire de execuție cu pasare de mesaje](ch16-02-message-passing.md)
    - [Concurență cu stare partajată](ch16-03-shared-state.md)
    - [Extinderea concurenței cu trăsăturile `Sync` și `Send`](ch16-04-extensible-concurrency-sync-and-send.md)

- [Caracteristicile programării orientate pe obiecte în Rust](ch17-00-oop.md)
    - [Caracteristicile limbajelor orientate pe obiecte](ch17-01-what-is-oo.md)
    - [Utilizarea obiectelor-trăsătură pentru valori de diferite tipuri](ch17-02-trait-objects.md)
    - [Implementarea unui pattern de design orientat pe obiecte](ch17-03-oo-design-patterns.md)

## Subiecte avansate

- [Pattern-urile și potrivirea](ch18-00-patterns.md)
    - [Toate locurile unde pattern-urile pot fi utilizate](ch18-01-all-the-places-for-patterns.md)
    - [Refutabilitatea: când un pattern ar putea eșua la potrivire](ch18-02-refutability.md)
    - [Sintaxa pattern-urilor](ch18-03-pattern-syntax.md)

- [Caracteristici avansate](ch19-00-advanced-features.md)
    - [Rust unsafe](ch19-01-unsafe-rust.md)
    - [Trăsături avansate](ch19-03-advanced-traits.md)
    - [Tipuri avansate](ch19-04-advanced-types.md)
    - [Funcții și închideri avansate](ch19-05-advanced-functions-and-closures.md)
    - [Macrocomenzi](ch19-06-macros.md)

- [Proiect final: Dezvoltarea unui server web multi-thread](ch20-00-final-project-a-web-server.md)
    - [Construirea unui server web cu un singur fir de execuție](ch20-01-single-threaded.md)
    - [Transformarea serverului nostru single-threaded într-unul multithreaded](ch20-02-multithreaded.md)
    - [Închiderea ordonată și curățarea](ch20-03-graceful-shutdown-and-cleanup.md)

- [Anexă](appendix-00.md)
    - [Anexa A: Cuvinte cheie](appendix-01-keywords.md)
    - [Anexa B: Operatori si simboluri](appendix-02-operators.md)
    - [Anexa C: Trăsături derivabile](appendix-03-derivable-traits.md)
    - [Anexa D - Unelte utile de dezvoltare](appendix-04-useful-development-tools.md)
    - [Anexa E - Edițiile](appendix-05-editions.md)
    - [Anexa F: Traduceri ale cărții](appendix-06-translation.md)
    - [Anexa G - Procesul de dezvoltare al Rust și „Nightly Rust”](appendix-07-nightly-rust.md)