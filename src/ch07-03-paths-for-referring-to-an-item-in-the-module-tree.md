##  Utilizarea căilor pentru a face referire la un element în structura de module

Când dorim să arătăm lui Rust unde să găsească un anumit element în structura de module, folosim o cale, exact cum am proceda atunci când navigăm printr-un sistem de fișiere. Pentru a apela o funcție este necesar să cunoaștem calea către aceasta.

Există două forme pe care o cale le poate lua:

* O *cale absolută* reprezintă calea completă începând de la rădăcina unui crate; în cazul codului ce provine dintr-un crate extern, calea absolută debutează cu numele crate-ului. Pentru codul din crate-ul actual, calea începe cu literalul `crate`.
* O *cale relativă* pornește de la modulul curent și utilizează `self`, `super`, sau un identificator din cadrul modulului curent.

Fie că vorbim despre căi absolute sau relative, acestea sunt urmate de unul sau mai multe identificatori separați de patru puncte (`::`).

Revenind la Lista 7-1, presupunem că vrem să convocăm funcția `add_to_waitlist`. Asta este echivalent cu a întreba: care este calea către funcția `add_to_waitlist`? Lista 7-3 include Lista 7-1, dar unele dintre module și funcții au fost eliminate.

Vom prezenta în cele ce urmează două moduri de a apela funcția `add_to_waitlist` din cadrul unei noi funcții, `eat_at_restaurant`, definită la rădăcina crate-ului. Desi aceste căi sunt corecte, există o altă problemă și care va împiedica compilarea cu succes a acestui exemplu în forma lui actuală. Vom explica motivul în scurt timp.

Funcția `eat_at_restaurant` este parte componentă a interfeței de programare a aplicației (API) publice a crate-ului nostru de tip bibliotecă, motiv pentru care o marcam cu cuvântul cheie `pub`. În secțiunea [“Expunerea căilor cu ajutorul cuvântului cheie `pub`”][pub]<!-- ignore -->, vom discuta mai pe larg despre `pub`.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

<span class="caption">Listarea 7-3: Apelul funcției `add_to_waitlist` utilizând căi absolute și relative</span>

În momentul în care apelăm pentru prima dată funcția `add_to_waitlist` în interiorul `eat_at_restaurant`, folosim o cale absolută. Funcția `add_to_waitlist` se află în același crate cu `eat_at_restaurant`, ceea ce înseamnă că avem posibilitatea de a folosi cuvântul cheie `crate` pentru a începe calea absolută. Includem apoi toate modulele succesive până ajungem la `add_to_waitlist`. Poți vizualiza această structură ca pe un sistem de fișiere: am specifica calea `/front_of_house/hosting/add_to_waitlist` pentru a executa programul `add_to_waitlist`; utilizarea termenului `crate` pentru a porni de la rădăcina unui crate este similară cu utilizarea `/` pentru a porni de la rădăcina sistemului de fișiere în shell-ul tău.

A doua oară când apelăm `add_to_waitlist` în cadrul `eat_at_restaurant`, utilizăm o cale relativă. Această cale începe cu `front_of_house`, numele modulului care este definit la același nivel în ierarhia de module ca și `eat_at_restaurant`. Echivalentul în contextul unui sistem de fișiere ar fi utilizarea căii `front_of_house/hosting/add_to_waitlist`. Pornirea de la numele unui modul indică faptul că calea este relativă.

Decizia de a utiliza o cale relativă sau una absolută va depinde de specificul proiectului tău și de probabilitatea de a mișca codul de definiție a unui element separat  sau împreună cu codul care face referire la acel element. De exemplu, dacă am muta modulul `front_of_house` și funcția `eat_at_restaurant` într-un modul numit `customer_experience`, am fi nevoiți să actualizăm calea absolută către `add_to_waitlist`, însă calea relativă ar rămâne validă. În contrast, dacă am muta funcția `eat_at_restaurant` într-un modul numit `dining`, calea absolută către apelul `add_to_waitlist` nu s-ar schimba, doar calea relativă ar necesita o actualizare. În general, preferăm să folosim căi absolute deoarece este mai probabil să dorim să mișcăm definițiile de cod și apelurile de funcții independent unul de celălalt.

Să încercăm să compilăm Listarea 7-3 pentru a înțelege motivul pentru care nu se compilează! Eroarea pe care o întâlnim este expusă în Listarea 7-4.

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

<span class="caption">Listarea 7-4: Erorile de compilare a codului din Listarea 7-3</span>

Mesajele de eroare indică faptul că modulul `hosting` este declarat ca fiind privat. Astfel, noi avem referințele corecte pentru modulul `hosting` și funcția `add_to_waitlist`, însă Rust nu acceptă utilizarea acestora datorită accesului restricționat la secțiunile private ale codului. În Rust, implicit, toate elementele - funcțiile, metodele, structurile, enumerările, modulele și constantele - sunt private în raport cu modulele părinte. Dacă vrei să faci un element anume, precum o funcție sau o structură, privat, îl introduci într-un modul.

Elementele dintr-un modul părinte nu pot utiliza elementele private din interiorul modulelor copil. În schimb, elementele din modulele copil au permisiunea de a folosi elementele prezentate în modulele lor ancestrale. Acest lucru se întâmplă deoarece modulele copil încapsulează și ascund detaliilor lor de implementare. Cu toate acestea, aceste module copil pot vedea contextul în care sunt definite. Într-un mod analog, putem vedea regulile de confidențialitate ca fiind similare cu gestionarea unei bucătării dintr-un restaurant: tot ceea ce se întâmplă în interior este privat din perspectiva clienților, dar managerii au capacitatea de a vedea și a coordona toate activitățile din restaurantul pe care îl administrează.

Decizia Rust de a proiecta sistemul de module în acest mod are la bază intenția de a ascunde, prin definiție, detaliile interne de implementare. În consecință, te vei putea ghida mai ușor în cunoașterea părților din cod pe care le poți modifica fără a afecta funcționarea restului de cod. Deși, Rust îți oferă posibilitatea de a dezvălui părți din interiorul codului modulelor copil către modulele ancestrale externe prin folosirea cuvântului cheie `pub` pentru a face un element anume public.

### Expunerea căilor private cu cuvântul cheie `pub`

Să ne reamintim de eroarea din Listarea 7-4, care ne indica faptul că modulul `hosting` este privat. Obiectivul nostru este ca funcția `eat_at_restaurant`, din modulul părinte, să aibă acces la funcția `add_to_waitlist`, localizată în modulul copil. Pentru a realiza acest lucru, vom marca modulul `hosting` cu cuvântul cheie `pub`, după cum observăm în Listarea 7-5.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs}}
```

<span class="caption">Listarea 7-5: Declararea modulului `hosting` ca fiind `pub` pentru a-l putea folosi în `eat_at_restaurant`</span>

Totuși, codul din Listarea 7-5 încă generează o eroare, așa cum vedem în Listarea 7-6.

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

<span class="caption">Listarea 7-6: Erorile de compilare a codului din Listarea 7-5</span>

Ce s-a întâmplat, de fapt? Adăugând cuvântul cheie `pub` în fața lui `mod hosting`, acest modul devine public. Cu această schimbare, dacă putem accesa `front_of_house`, atunci putem accesa și `hosting`. Cu toate acestea, conținutul modulului `hosting` rămâne în continuare privat. Transformarea unui modul într-unul public nu implică și transformarea conținutului său în public. Cuvântul cheie `pub` pentru un modul le permite doar modulelor ancestrale să facă referire la acesta, fără a-i putea accesa codul intern. Deoarece modulele sunt practic containere, simpla lor transformare în publice nu este de ajuns; este necesar și să decidem dacă transformăm unul sau mai multe elemente din modul în publice.

Erorile din Listarea 7-6 ne indică faptul că funcția `add_to_waitlist` are un statut privat. Aceste reguli privind confidențialitatea se aplică atât structurilor, enumerărilor, funcțiilor și metodelor, cât și modulelor.

Să transformăm funcția `add_to_waitlist` într-una publică, adăugând cuvântul cheie `pub` înainte de definiția sa, așa cum este ilustrat în Listarea 7-7.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs}}
```

<span class="caption">Listarea 7-7: Prin adăugarea cuvântului cheie `pub` înainte de `mod hosting`
și `fn add_to_waitlist`, putem apela funcția
din `eat_at_restaurant`</span>

Acum codul se va compila cu succes! Ca să înțelegem de ce utilizarea cuvântului cheie `pub` ne permite să folosim aceste căi în cadrul funcției `add_to_waitlist`, respectând astfel regulile de confidențialitate, să examinăm căile atât absolute, cât și cele relative.

În calea absolută, pornim de la `crate`, rădăcina arborelui de module ale crate-ului nostru. Modulul `front_of_house` este definit chiar la rădăcina crate-ului. Deși `front_of_house` nu are statutul de public, putem totuși referi `front_of_house` în cadrul funcției `eat_at_restaurant`, întrucât ambele sunt definite în același modul (adică, sunt "surori"). Următorul pas este modulul `hosting` marcat cu `pub`. Putem accesa modulul părinte al `hosting`, ceea ce înseamnă că putem accesa `hosting`. În final, funcția `add_to_waitlist` este marcată cu `pub` și putem accesa modulul ei părinte, deci apelul funcției noastre funcționează!

În calea relativă, procesul este identic cu cel din calea absolută, cu excepția primului pas: în loc să înceapă de la rădăcina crate-ului, calea demarează de la `front_of_house`. Modulul `front_of_house` este definit în cadrul aceluiași modul cu `eat_at_restaurant`, așadar calea relativă ce pornește din modulul unde `eat_at_restaurant` este definit este funcțională. Ulterior, pentru că `hosting` și `add_to_waitlist` sunt etichetate cu `pub`, restul căii este funcțional și acest apel de funcție este valid!

Dacă vrei să îți distribui crate-ul de librărie pentru ca alte proiecte să utilizeze codul tău, API-ul tău public este contractul tău cu utilizatorii crate-ului tău. Acesta va stabili modul în care aceștia pot interacționa cu codul tău. E mult de luat în considerare atunci când vine vorba de gestionarea modificărilor la API-ul tău public pentru a facilita dependența utilizatorilor față de crate-ul tău. Aceste aspecte nu sunt cuprinse în sfera acestei cărți; dacă acest subiect te interesează, consultă [Ghidul pentru API-ul Rust][api-guidelines].

> #### Practici recomandate pentru pachete cu un executabil și o bibliotecă
>
> Am precizat anterior că un pachet poate include atât un crate de tip
> executabil în *src/main.rs*, cât și un crate de tip bibliotecă în
> *src/lib.rs*, ambele având în mod implicit numele pachetului. De obicei,
> pachetele construite astfel încât să dispună de un crate de tip bibliotecă și
> unul de tip executabil, vor avea un cod minim în crate-ul executabil, necesar
> doar pentru a iniția un executabil care va apela codul din crate-ul de tip
> bibliotecă. Aceasta abordare permite altor proiecte să beneficieze de
> funcționalitățile oferite la maximum de pachet, întrucât codul crate-ului de
> tip bibliotecă poate fi distribuit. Structura arborelui de module trebuie
> definită în *src/lib.rs*. Apoi, elementele publice pot fi folosite în
> crate-ul de tip executabil, prin specificarea numelui pachetului la începutul
> căilor. Astfel, crate-ul de tip executabil devine un utilizator al crate-ului
> de tip bibliotecă, exact cum un crate complet extern ar utiliza crate-ul de
> tip bibliotecă: are acces doar la API-ul public. Acest lucru ajută la
> conceperea unui API eficient; nu numai că ești autor, dar ești și utilizator!
> În [Capitolul 12][ch12]<!-- ignore -->, vom ilustra această bună practică
> organizatorică printr-un program de linie de comandă, care va include atât un
> crate de tip executabil, cât și unul de tip bibliotecă.


### Inițierea căilor relative cu `super`

Putem crea căi relative care încep direct cu modulul părinte, în loc să le începem cu modulul curent sau cu rădăcina crate-ului. Aceasta se realizează utilizând `super` la începutul căii. Acest concept este similar cu începutul unei căi de sistem de fișiere cu sintaxa `..`. Prin utilizarea `super` avem capacitatea de a referi un element de care știm că se găsește în modulul părinte. Aceasta poate facilita procesul de rearanjare a structurii modulelor, în special dacă modulul este strâns legat de modulul părinte, dar acesta din urmă ar putea fi mutat în altă parte a structurii de module în viitor.

Luăm în considerare codul din Listarea 7-8, în care este prezentată situația în care un bucătar corectează o comandă greșită și o duce personal clientului. Funcția `fix_incorrect_order` definită în modulul `back_of_house` apelează funcția `deliver_order` definită în modulul părinte, indicația către funcția `deliver_order` începând cu `super`:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

<span class="caption">Listare 7-8: Apelul unei funcții folosind o cale relativă, care începe cu `super`</span>

Funcția `fix_incorrect_order` se află în modulul `back_of_house`, așa că folosim `super` pentru a ajunge la modulul părinte al `back_of_house`, care, în acest caz, este `crate`, rădăcina. De acolo, căutăm `deliver_order` și îl găsim. Succes! Considerăm că modulul `back_of_house` și funcția `deliver_order` vor păstra aceeași relație și că acestea vor fi mutate împreună în cazul în care decidem să reorganizăm structura modulelor în crate. Prin urmare, utilizăm `super` pentru a avea mai puține locuri de actualizat în codul sursă în viitor, dacă acest cod va fi mutat într-un modul diferit.

### Setarea structurilor și enumerărilor ca fiind publice

Avem posibilitatea de a folosi `pub` pentru a seta structurile și enumerările ca fiind publice. Cu toate acestea, există câteva aspecte suplimentare referitoare la modul în care `pub` interacționează cu structurile și enumerările. Dacă aplicăm `pub` înaintea definiției unei structuri, facem structura publică, dar câmpurile acesteia vor rămâne private. Putem decide individual dacă fiecare câmp în parte va fi sau nu public. În Listarea 7-9, am definit o structură publică `back_of_house::Breakfast` care conține un câmp public `toast`, dar și un câmp privat `seasonal_fruit`. Acest lucru este similar cu situația dintr-un restaurant în care clientul poate alege tipul de pâine care însoțește masa, însă bucătarul hotărăște care fruct va acompania masa, în funcție de fructele de sezon disponibile. Deoarece varietatea de fructe disponibile se schimbă frecvent, clienții nu au posibilitatea de a alege fructul sau chiar de a vedea ce fruct vor primi.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

<span class="caption">Listarea 7-9: O structură cu anumite câmpuri publice și altele
private</span>

Datorită faptului că în structura `back_of_house::Breakfast`, câmpul `toast` este public, putem citi și scrie în câmpul `toast` folosind notația cu punct, în funcția `eat_at_restaurant`. Rețineți că nu putem utiliza câmpul 'seasonal_fruit' în `eat_at_restaurant`, deoarece 'seasonal_fruit' este privat. Încearcă să de-comentezi linia care modifică valoarea câmpului `seasonal_fruit` pentru a vedea ce eroare apare!

Trebuie remarcat și faptul că, deoarece `back_of_house::Breakfast` conține un câmp privat, structura trebuie să ofere o funcție asociată publică pentru a construi o instanță a `Breakfast` (în acest caz, am numit-o `summer`). Dacă `Breakfast` nu ar avea o astfel de funcție, în funcția `eat_at_restaurant` nu am putea crea o instanță a `Breakfast`, deoarece nu am putea seta valoarea câmpului privat `seasonal_fruit`.

În contrast, dacă stabilim o enumerare ca fiind publică, toate variantele acesteia devin publice. Ne este suficient să aplicăm `pub` înainte de cuvântul-cheie `enum`, așa cum este ilustrat în Listarea 7-10.

<span class="filename">Numnele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

<span class="caption">Listarea 7-10: Selectarea unei enumerări ca publică face ca toate variantele acesteia să fie accesibile</span>

Am declarat enum-ul `Appetizer` ca fiind public, ceea ce ne permite să folosim variantele `Soup` și `Salad` în codul `eat_at_restaurant`.

Enumerările nu sunt foarte utile dacă variantelor lor nu le este permis accesul public; ar fi incomod să trebuiască să adnotăm fiecare variantă a enumerării cu `pub` în fiecare caz. Din acest motiv, modul implicit pentru variantele de enumerări este acela de a fi public. Pe de altă parte, structurile sunt frecvent folosite fără ca câmpurile lor să fie publice, astfel că câmpurile unei structuri urmează regula generală unde totul este privat în mod implicit, cu excepția cazului în care este adnotat cu `pub`.

Există încă o situație în care este implicată utilizarea `pub` și pe care nu am discutat-o încă, aceasta fiind ultima caracteristică a sistemului de module, și anume cuvântul cheie `use`. Vom discuta mai întâi `use` în mod independent, apoi vom demonstra cum să combinăm `pub` și `use`.

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
