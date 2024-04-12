# Pointeri inteligenți

Un *pointer* este un concept general pentru o variabilă ce conține o adresă de memorie. Această adresă referă sau „indică” către alte date. Tipul cel mai comun de pointer în Rust este referința, pe care am învățat-o în Capitolul 4. Referințele sunt semnalate de simbolul `&` și împrumută valoarea la care se referă. În afara acestei funcții, ele nu au capabilități speciale și nu implică un supra-cost.

În contrast, *pointerii inteligenți* sunt structuri de date care funcționează ca niște pointeri, dar au, de asemenea, metadate suplimentare și capacitatea de a efectua mai multe operațiuni. Conceptul de pointeri inteligenți nu este unic pentru Rust; aceștia provin din C++ și se regăsesc și în alte limbaje de programare. Biblioteca standard Rust include o selecție de pointeri inteligenți care oferă mai mult decât simple referințe. Pentru a explora acest concept în general, vom examina diferite tipuri de pointeri inteligenți, inclusiv un tip care se folosește de *numărarea referințelor*. Aceast pointer permite datelor să fie deținute de mai mulți posesori și, atunci când nu mai există niciun posesor, se ocupă de eliberarea acestora.

Având în vedere conceptele de posesiune și împrumut proprii lui Rust, există o diferență esențială între referințe și pointerii inteligenți: în timp ce referințele doar împrumută datele, pointerii inteligenți, deseori, *dețin* datele la care se referă.

Deși până acum nu i-am numit în mod explicit ca atare, am întâlnit deja câțiva pointeri inteligenți în această carte, printre care tipurile `String` și `Vec<T>` prezentate în Capitolul 8. Aceste tipuri sunt considerate pointeri inteligenți pentru că dețin memorie și permit manipularea acesteia. În plus, dispun de metadate și facilități sau garanții suplimentare. De pildă, `String` stochează capacitatea sa ca metadată și garantează că datele sale vor fi întotdeauna un șir de coduri UTF-8 valide.

Pointerii inteligenți sunt de obicei implementați ca structuri. Spre deosebire de o structură obișnuită, pointerii inteligenți implementează trăsăturile `Deref` și `Drop`. Trăsătura `Deref` le permite instanțelor structurii pointer înteligent să se comporte ca referințe, făcând posibilă scrierea codului compatibil atât cu referințele cât și cu pointerii inteligenți. Trăsătura `Drop` este folosită pentru personalizarea codului executat când o instanță a pointerului inteligent iese din domeniul de vizibilitate. În acest capitol, vom lua în discuție ambele trăsături și vom arăta importanța lor pentru pointerii inteligenți.

Dat fiind că pointerul inteligent este un model de proiectare des utilizat în Rust, acest capitol nu va aborda toți pointerii inteligenți existenți. Multe biblioteci oferă variante proprii de pointeri inteligenți, iar tu poți să-ți creezi chiar versiuni proprii. Ne vom concentra asupra celor mai uzuali pointeri inteligenți din biblioteca standard:

* `Box<T>` pentru alocarea valorilor pe heap
* `Rc<T>`, un tip ce gestionează numărarea referințelor și facilitează posesiunea multiplă
* `Ref<T>` și `RefMut<T>`, accesibile prin `RefCell<T>`, un tip care aplică regulile de împrumut la timpul de execuție, nu la compilare

De asemenea, vom explora modelul de *mutabilitate interioară*, prin care un tip imutabil oferă o interfață API pentru modificarea unei valori interne. Vom discuta, totodată, despre *ciclurile de referințe*: cum pot ele să genereze scurgeri de memorie și cum le putem preveni.

Să ne scufundăm în subiect!
