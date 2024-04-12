## Extinderea concurenței cu trăsăturile `Sync` și `Send`

Interesant este că limbajul Rust include *foarte* puține funcții native de concurență. Majoritatea caracteristicilor de concurență menționate până acum în acest capitol sunt parte a bibliotecii standard, nu a limbajului în sine. Modalitățile de gestionare a concurenței nu se limitează doar la limbaj sau biblioteca standard; poți dezvolta funcții de concurență proprii sau să utilizezi cele dezvoltate de alții.

Cu toate acestea, două concepte fundamentale de concurență sunt integrante în limbaj: trăsăturile `std::marker` `Sync` și `Send`.

### Permiterea transferului posesiunii între fire cu `Send`

Trăsătura de marcaj `Send` indică faptul că posesiunea valorilor ce implementează `Send` poate fi transferată între diferite fire de execuție. Aproape toate tipurile din Rust sunt `Send`, cu unele excepții, printre care se numără `Rc<T>`: acesta nu este `Send` deoarece, după clonarea unei valori `Rc<T>` și încercarea de a transfera clona către un alt fir, există riscul ca ambele fire să actualizeze în același timp contorul de referințe. Așadar, `Rc<T>` este proiectat pentru utilizare în scenarii cu un singur fir de execuție, când vrei să eviți penalitățile de performanță legate de siguranța utilizării firelor multiple.

Deci, sistemul de tipuri din Rust și delimitările de trăsături garantează că nu poți transmite accidental o valoare `Rc<T>` între fire de execuție în mod nesigur. De exemplu, în Listarea 16-14, când am încercat acest lucru, am întâmpinat eroarea `the trait Send is not implemented for Rc<Mutex<i32>>`. Schimbarea la `Arc<T>`, care este `Send`, a făcut ca programul să compileze cu succes.

Orice tip alcătuit doar din elemente de tip `Send` este automat etichetat ca fiind `Send`. Cu excepția câtorva cazuri specifice, ca pointerii raw care vor fi analizați în Capitolul 19, majoritatea tipurilor primitive sunt `Send`.

### Permiterea accesului din multiple fire de execuție cu `Sync`

Trăsătura de marcaj `Sync` semnalează că tipul ce o implementează e sigur pentru a fi referit din multiple fire de execuție. Adică, un tip `T` este `Sync` dacă `&T` (o referință imutabilă spre `T`) poate fi `Send`, sugerând că referința poate fi transferată în siguranță spre un alt fir. La fel ca `Send`, tipurile primitive sunt `Sync` și, similare lor, tipurile formate exclusiv din elemente `Sync` sunt de asemenea `Sync`.

Pointerul inteligent `Rc<T>` nu este `Sync`, din aceleași motive că nu este nici `Send`. Tipul `RefCell<T>` (discutat în Capitolul 15) și familia sa de tipuri `Cell<T>` nu sunt `Sync`. Modul în care `RefCell<T>` implementează verificarea împrumuturilor în timpul execuției nu este compatibil cu firele. Pe de altă parte, pointerul inteligent `Mutex<T>` este `Sync` și pot fi utilizate pentru a oferi acces partajat între mai multe fire, așa cum am văzut în secțiunea [„Partajarea unui `Mutex<T>` între fire multiple”][sharing-a-mutext-between-multiple-threads]<!-- ignore -->.

### Implementarea manuală a `Send` și `Sync` implică riscuri

Fiindcă tipurile compuse din trăsături `Send` și `Sync` devin automat și ele `Send` și `Sync`, nu e necesară implementarea manuală a acestor trăsături. În calitate de trăsătură de marcaj, acestea nici nu necesită implementarea de metode. Ele sunt importante pentru menținerea invarianților legați de concurență.

Implementarea manuală a acestor trăsături presupune folosirea codului Rust nesigur. Vom discuta despre codul Rust nesigur în Capitolul 19; deocamdată, informația cheie este că dezvoltarea de noi tipuri orientate spre concurență care nu sunt formate din componentele `Send` și `Sync` necesită o analiză riguroasă pentru a respecta garanțiile de siguranță. Mai multe detalii despre aceste garanții și cum pot fi respectate se găsesc în [„Rustonomicon”][nomicon].

## Sumar

Nu este ultima data când vom vedea concurența în această carte: proiectul din Capitolul 20 va aplica conceptele prezentate în acest capitol într-un context mai realist decât cel restrâns discutat aici.

După cum am menționat mai devreme, deoarece o mică parte din felul în care Rust abordează concurența este inclusă direct în limbaj, multe soluții pentru concurență sunt oferite sub formă de crate-uri. Acestea avansează mai agil decât biblioteca standard, așa că este esențial să căutați online crate-urile moderne și sofisticate pe care să le utilizați în scenarii cu fire de execuție multiple.

Biblioteca standard Rust pune la dispoziție canale pentru schimbul de mesaje și tipuri de pointere inteligente, cum ar fi `Mutex<T>` și `Arc<T>`, ce pot fi folosite în siguranță în contexte concurente. Sistemul de tipuri și verificatorul de împrumut garantează că orice cod care încorporează aceste soluții nu va suferi de curse de date sau de referințe nevalide. Odată ce codul tău compilează cu succes, poți fi liniștit știind că acesta va rula fiabil pe fire multiple de execuție, fără a provoca genul acelor erori dificil de localizat, frecvente în alte limbaje. Programarea concurentă nu mai este un concept intimidant: avansează cu încredere și implementează concurența în programele tale!

În capitolul următor, vom aborda metode idiomatice de a conceptualiza problemele și de a structura soluțiile pe măsură ce aplicațiile tale Rust se măresc. De asemenea, vom discuta cum se compară idiomele Rust cu cele la care ai putea fi obișnuit din paradigma programării orientate pe obiecte.

[sharing-a-mutext-between-multiple-threads]:
ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads
[nomicon]: ../nomicon/index.html
