## Anexa C: Trăsături derivabile

În diverse părți din carte, am discutat despre atributul `derive`, pe care poți să-l aplici la o definiție de structură sau enumerare. Atributul `derive` generează codul care va implementa o trăsătură cu propria implementare implicită pe tipul pe care l-ai adnotat cu sintaxa `derive`.

În această anexă, oferim o referință a tuturor trăsăturilor din biblioteca standard pe care le poți utiliza cu `derive`. Fiecare secțiune acoperă:

* Ce operatori și metode activează derivarea acestei trăsături
* Ce face implementarea trăsăturii furnizată prin `derive`
* Ce semnifică implementarea trăsăturii despre tipul 
* Condițiile în care ți se permite sau nu implementezi trăsătura
* Exemple de operații care necesită trăsătura

Dacă dorești un comportament diferit de cel oferit de atributul `derive`, consultă [documentația bibliotecii standard](../std/index.html)<!-- ignore --> pentru fiecare trăsătură, pentru detalii despre cum să le implementezi manual.

Trăsăturile enumerate aici sunt singurele definite de biblioteca standard care pot fi implementate pe tipurile tale folosind `derive`. Alte trăsături definite în biblioteca standard nu au un comportament implicit bine definit, așa că este de datoria ta să le implementezi în modul care are sens pentru ceea ce încerci să realizezi.

Un exemplu de trăsătură care nu poate fi derivată este `Display`, care se ocupă de formatarea pentru utilizatorii finali. Ar trebui să iei în considerare întotdeauna modalitatea potrivită de a afișa un tip către un utilizator final. Ce părți ale tipului ar trebui să aibă permisiunea utilizatorilor finali de a vedea? Ce părți ar considera acestea relevante? Ce format al datelor ar fi cel mai relevant pentru ei? Compilatorul Rust nu are aceste cunoștințe, așa că nu îți poate oferi un comportament implicit corespunzător.

Lista de trăsături derivabile furnizate în această anexă nu este exhaustivă:
librăriile pot implementa `derive` pentru propriile lor trăsături, 
făcând lista de trăsături cu care poți folosi `derive` cu adevărat deschisă.
Implementarea `derive` implică utilizarea unei macrocomenzi procedurale, ce este acoperită în secțiunea [„Macrcomenzi”][macros]<!-- ignore --> a Capitolului 19.

### `Debug` pentru afișare de depanare

Trăsătura `Debug` permite formatarea pentru depanare în string-uri formatate, pe care o indicăm prin adăugarea `:?` în acoladele `{}`.

Trăsătura `Debug` îți permite să printezi instanțe ale unui tip pentru scopuri de depanare, astfel încât tu și alți programatori care utilizează tipul tău să puteți inspecta o instanță la un anumit punct în execuția unui program.

Trăsătura `Debug` este necesară, de exemplu, în utilizarea macrcomenzii `assert_eq!`. Această macrocomandă afișează valorile instanțelor date ca argumente dacă aserțiunea de egalitate eșuează, astfel încât programatorii să poată vedea de ce cele două instanțe nu au fost egale.

### `PartialEq` și `Eq` pentru compararea de egalitate

Trăsătura `PartialEq` îți permite să compari instanțe ale unui tip pentru a verifica egalitatea și permite utilizarea operatorilor `==` și `!=`.

Derivarea `PartialEq` implementează metoda `eq`. Când `PartialEq` este derivat pe structuri, două instanțe sunt egale doar dacă *toate* câmpurile sunt egale, iar instanțele nu sunt egale dacă orice câmpuri nu sunt egale. Când este derivat pe enumerări, fiecare variantă este egală cu ea însăși și nu este egală cu celelalte variante.

Trăsătura `PartialEq` este necesară, de exemplu, cu utilizarea macrocomenzii `assert_eq!`, care are nevoie să poată compara două instanțe ale unui tip pentru egalitate.

Trăsătura `Eq` nu are metode. Scopul său este de a semnala că pentru fiecare valoare a tipului adnotat, valoarea este egală cu ea însăși. Trăsătura `Eq` poate fi aplicată doar la tipuri care implementează de asemenea `PartialEq`, deși nu toate tipurile care implementează `PartialEq` pot implementa `Eq`. Un exemplu în acest sens este tipurile cu numere cu virgulă mobilă: implementarea numerelor cu virgulă mobilă afirmă că două instanțe ale valorii care nu este un număr (`NaN`) nu sunt egale între ele.

Un exemplu de când este necesar `Eq` este pentru cheile într-un `HashMap<K, V>` astfel încât `HashMap<K, V>` să poată spune dacă două chei sunt la fel.

### `PartialOrd` și `Ord` pentru Compararea Ordinelor

Trăsătura `PartialOrd` îți permite să compari instanțe ale unui tip în vederea sortării. Un tip care implementează `PartialOrd` poate fi folosit cu operatorii `<`, `>`, `<=`, și `>=`. Poți aplica trăsătura `PartialOrd` doar la tipurile care implementează și `PartialEq`.

Derivarea lui `PartialOrd` implementează metoda `partial_cmp`, care returnează un `Option<Ordering>` ce va fi `None` când valorile date nu produc o ordonare. Un exemplu de valoare care nu produce o ordonare, chiar dacă majoritatea valorilor de acel tip pot fi comparate, este valoarea numărului în virgulă mobilă care nu este un număr (`NaN`). Apelarea `partial_cmp` cu orice număr în virgulă mobilă și valoarea `NaN` a unui număr în virgulă mobilă va returna `None`.

Când este derivată la structuri, `PartialOrd` compară două instanțe prin compararea valorii în fiecare câmp în ordinea în care câmpurile apar în definiția structurii. Când este derivată la enumerări, variantele enumerării declarate mai devreme în definiția enumerării sunt considerate mai mici decât variantele listate mai târziu.

Trăsătura `PartialOrd` este necesară, de exemplu, pentru metoda `gen_range` din crate-ul `rand` care generează o valoare aleatorie în intervalul specificat de o expresie de interval.

Trăsătura `Ord` îți permite să știi că, pentru oricare două valori ale tipului adnotat, va exista o ordonare validă. Trăsătura `Ord` implementează metoda `cmp`, care returnează un `Ordering` în loc de un `Option<Ordering>`, deoarece o ordonare validă va fi întotdeauna posibilă. Poți aplica trăsătura `Ord` doar la tipurile care implementează și `PartialOrd` și `Eq` (iar `Eq` necesită `PartialEq`). Când este derivată la structuri și enumerări, `cmp` se comportă în același mod ca și implementarea derivată pentru `partial_cmp` cu `PartialOrd`.

Un exemplu de când `Ord` e necesar este când stochezi valori într-un `BTreeSet<T>`, o structură care stochează date bazate pe ordinea de sortare a valorilor.

### `Clone` și `Copy` pentru duplicarea valorilor

Trăsătura `Clone` îți permite să creezi explicit o copie profundă a unei valori, iar procesul de duplicare ar putea implica rularea unui cod arbitrar și copierea datelor de pe heap. Vezi secțiunea [„Modalități prin care variabilele și datele interacționează: Clone”][ways-variables-and-data-interact-clone]<!-- ignore --> în Capitolul 4 pentru mai multe informații despre `Clone`.

Derivarea `Clone` implementează metoda `clone`, care atunci când este implementată pentru întregul tip, apelează `clone` pe fiecare parte a tipului. Acest lucru înseamnă că toate câmpurile sau valorile din tip trebuie să implementeze, de asemenea, `clone` pentru a deriva `Clone`.

Un exemplu în care este necesară `Clone` este atunci când apelezi metoda `to_vec` pe un slicing. Slicing-ul nu deține instanțele tipului pe care îl conține, dar vectorul returnat de `to_vec` va trebui să își dețină instanțele, așa că `to_vec` apelează
`clone` la fiecare element. Astfel, tipul stocat în slicing trebuie să implementeze `Clone`.

Trăsătura `Copy` îți permite să dublezi o valoare doar prin copierea biților stocați în stivă; nu este necesar un cod arbitrar. Vezi secțiunea [„Date doar pe stivă: Copy”][stack-only-data-copy]<!-- ignore --> în Capitolul 4 pentru mai multe informații despre `Copy`.

Trăsătura `Copy` nu definește nicio metodă pentru a preveni programatorii să supraîncarce acele metode și să încalce presupunerea că nu rulează niciun cod arbitrar. În acest fel, toți programatorii pot presupune că copierea unei valori va fi foarte rapidă.

Puteți deriva `Copy` pe orice tip ale cărui părți implementează toate `Copy`. Un tip care implementează `Copy` trebuie să implementeze, de asemenea, `Clone`, deoarece un tip care implementează `Copy` are o implementare trivială a `Clone` care efectuează aceeași sarcină ca `Copy`.

Trăsătura `Copy` este rareori necesară; tipurile care implementează `Copy` au optimizări disponibile, ceea ce înseamnă că nu trebuie să apelezi `clone`, făcând astfel codul mai concis.

Orice este posibil cu `Copy` se poate realiza și cu `Clone`, dar codul ar putea fi mai lent sau ar putea fi necesar să folosești `clone` în anumite locuri.

### `Hash` pentru maparea unei valori către o valoare de dimensiune fixă

Trăsătura `Hash` îți permite să iei o instanță a unui tip de dimensiune arbitrară și să mappezi această instanță la o valoare de dimensiune fixă folosind o funcție de hash. Derivarea `Hash` implementează metoda `hash`. Implementarea derivată a metodei `hash` combină rezultatul apelării `hash` pe fiecare parte a tipului, adică toate câmpurile sau valorile trebuie să implementeze de asemenea `Hash` pentru a deriva `Hash`.

Un exemplu de când `Hash` este necesară este la păstrarea cheilor într-un `HashMap<K, V>` pentru a stoca date eficient.

### `Default` pentru valori implicite

Trăsătura `Default` îți permite să creezi o valoare implicită pentru un tip. Derivarea `Default` implementează funcția `default`. Implementarea derivată a funcției
`default` apelează funcția `default` pe fiecare parte a tipului, adică toate câmpurile sau valorile din tip trebuie să implementeze de asemenea `Default` pentru a deriva `Default`.

Funcția `Default::default` este folosită în mod obișnuit în combinație cu sintaxa de actualizare a structurilor discutată în [“Crearea instanțelor din alte instanțe cu sintaxa de actualizare a structurii”][creating-instances-from-other-instances-with-struct-update-syntax]<!-- ignore --> section în Capitolul 5. Poți personaliza câteva câmpuri ale unei struct-uri și apoi setează și utilizează o valoare implicită pentru restul câmpurilor folosind `..Default::default()`. 

Trăsătura `Default` este necesară când folosești metoda `unwrap_or_default` pe instanțe `Option<T>`, de exemplu. Dacă `Option<T>` este `None`, metoda `unwrap_or_default` va returna rezultatul `Default::default` pentru tipul `T` stocat în `Option<T>`.

[creating-instances-from-other-instances-with-struct-update-syntax]:
ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
[stack-only-data-copy]:
ch04-01-what-is-ownership.html#stack-only-data-copy
[ways-variables-and-data-interact-clone]:
ch04-01-what-is-ownership.html#ways-variables-and-data-interact-clone
[macros]: ch19-06-macros.html#macros
