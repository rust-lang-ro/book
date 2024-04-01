## Anexa E - Edițiile

În Capitolul 1, ai observat că `cargo new` adaugă un pic de metadate în *Cargo.toml* referitoare la o anumită ediție. Această anexă te va ajuta să înțelegi ce semnificație are acest aspect.

Limbajul de programare Rust și compilatorul său au un ciclu stabil de lansare o dată la fiecare șase săptămâni, astfel încât utilizatorii primesc constant functionalități noi. În comparație, alte limbaje de programare lansează modificări substanțiale mai rar; Rust, în schimb, preferă actualizări mai mici, dar mai dese. De-a lungul timpului, toate aceste actualizări mici adunate reușesc să schimbe în mod semnificativ limbajul. Fără o repere clare, poate fi dificil să realizezi cât de mult a progresat Rust de-a lungul mai multor versiuni cum ar fi de la Rust 1.10 la Rust 1.31.

La intervale de câte doi sau trei ani, echipa Rust lansează o nouă ediție a limbajului. O ediție înglobează toate funcționalitățile noi într-un format ușor de abordat, împreună cu documentație completă și instrumente actualizate. Aceste noi ediții sunt parte integrantă a ritmului standard de lansare.

Pentru diferite grupuri de utilizatori, edițiile îndeplinesc scopuri distincte:

* Pentru cei ce folosesc Rust activ, o nouă ediție aduce schimbările incrementale într-un mod compact și accesibil.
* Pentru non-utilizatorii de Rust, o nouă ediție indică schimbări de amploare și poate incita la acordarea unei noi șanse limbajului.
* Pentru contribuitorii la Rust, o ediție nouă este un moment de sărbătoare ce evidențiază evoluția proiectului și a comunității acestuia.

În momentul redactării, sunt disponibile trei ediții Rust: Rust 2015, Rust 2018 și Rust 2021. Acest text se conformează idiomurilor ediției Rust 2021.

Cheia `edition` din *Cargo.toml* îi spune compilatorului Rust ce ediție să utilizeze pentru codul tău. Dacă această cheie lipsește, Rust va folosi ediția `2015` ca implicită, pentru a păstra compatibilitatea cu versiunile anterioare.

Proiectele Pot opta pentru orice ediție disponibilă, diferită de cea implicită din 2015. O ediție poate introduce schimbări ce nu sunt compatibile în versiuni anterioare, de exemplu noi cuvinte cheie ce ar putea conflicta cu identificatorii din cod. Cu toate acestea, compilatorul va continua să compileze codul existent chiar și după actualizarea versiunii de compilator Rust, atâta timp cât nu alegi să adopți schimbările noi.

Toate versiunile de compilator Rust suportă fiecare ediție apărută înaintea versiunii acelui compilator și pot interacționa cu dependențele indiferent de ediția în care sunt scrise. Modificările aduse de o nouă ediție alterează doar modul în care compilatorul interpretează codul inițial. Astfel, dacă ai un proiect în Rust 2015 și o dependență care utilizează Rust 2018, proiectul va compila și va lucra fără probleme cu respectiva dependență. La fel, dacă proiectul tău este în Rust 2018 și folosește o dependență în Rust 2015, totul va funcționa corespunzător.

Să fim clari: cele mai multe caracteristici sunt disponibile indiferent de ediția Rust folosită. Dezvoltatorii vor continua să beneficieze de îmbunătățiri, indiferent de ediția Rust pe care o utilizează, pe măsură ce sunt lansate noi versiuni stabile. În anumite situații, mai ales atunci când sunt introduse cuvinte cheie noi, anumite funcționalități noi pot fi limitate doar la ediții mai recente. Pentru a accesa aceste caracteristici, va trebui să treci la o ediție mai nouă.

Pentru detalii suplimentare, [*Ghidul Edițiilor*](https://doc.rust-lang.org/stable/edition-guide/) reprezintă o resursă exhaustivă despre ediții, prezentând diferențele dintre ele și detaliind cum poți actualiza codul la o nouă ediție simplu și automat cu `cargo fix`.