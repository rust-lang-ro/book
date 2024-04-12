# Concurență fără temeri

Unul dintre principalele obiective ale Rust este să faciliteze o programare concurentă sigură și eficientă. *Programarea concurentă*, care permite executarea independentă a diferitelor componente ale unui program, și *programarea paralelă*, care permite execuția simultană a acestora, devin tot mai relevante pe măsură ce computerele extind utilizarea procesoarelor multiple. În trecut, aceste forme de programare au prezentat dificultăți semnificative și erau predispuse la greșeli: Rust intenționează să revoluționeze acest domeniu.

La început, echipa Rust era de părere că asigurarea siguranței memoriei și prevenirea problemelor de concurență sunt două provocări distincte, ce ar trebui soluționate cu metode diferite. Cu timpul, echipa a realizat că sistemele de posesiune și de tipizare sunt un set de unelte extrem de eficiente în gestionarea atât a siguranței memoriei *cât și* a problemelor de concurență! Folosindu-se de conceptul de posesiune și verificarea tipurilor, multe erori legate de concurență se transformă în erori de compilare în Rust, în loc de erori de rulare. Deci, în loc să pierzi timp încercând să reproduci condițiile exacte în care apare un bug de concurență la execuție, codul eronat pur și simplu nu va compila, afișând o eroare ce explică problema. Prin urmare, poti rectifica codul în timp ce lucrezi la el, nu după ce acesta a ajuns deja în producție. Această abordare din Rust este cunoscută sub numele de *concurența fără teamă*. Concurența fără teamă îți oferă posibilitatea de a scrie cod fără bug-uri subtile și care poate fi refactorizat cu ușurință fără a introduce noi erori.

> Notă: Pentru a menține simplitatea, vom numi multe dintre probleme ca fiind
> *concurente* în loc de a fi mai preciși și a spune *concurente și/sau
> paralele*. Dacă subiectul acestei cărți ar fi fost concurența și/sau
> paralelismul, am fi fost mai expliciți. Pentru acest capitol, te rog să
> substitui mental *concurente și/sau paralele* de fiecare dată când folosim
> termenul *concurente*.

Multe limbaje sunt rigide când vine vorba de soluțiile pe care le propun pentru gestionarea problemelor de concurență. De exemplu, Erlang oferă facilități elegante pentru concurența prin pasare de mesaje, dar are doar câteva metode obtuze pentru a împărtăși starea între fire de execuție. A susține exclusiv o gamă limitată de soluții este o strategie acceptabilă pentru limbajele de nivel înalt, dat fiind că aceste limbaje promit avantaje prin sacrificarea unei părți din control în schimbul abstracțiilor. În schimb, limbajele de nivel jos sunt așteptate să furnizeze soluția cu cea mai bună performanță în fiecare situație specifică și oferă mai puține abstracții ale hardware-ului. Așadar, Rust pune la dispoziție o gamă variată de instrumente pentru modelarea problemelor în orice mod se potrivește cel mai bine situației și necesităților tale.

Iată temele pe care le vom aborda în acest capitol:

* Cum să inițiezi thread-uri pentru a executa simultan mai multe porțiuni de cod
* Concurența prin *pasarea de mesaje*, unde canale comunică mesaje între fire de execuție
* Concurența cu *stare partajată*, unde mai multe fire de execuție au acces la același fragment de date
* Trăsăturile `Sync` și `Send`, care extind garanțiile de concurență oferite de Rust către tipurile definite de utilizator, precum și către cele oferite de biblioteca standard
