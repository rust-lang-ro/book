## Caracteristicile limbajelor orientate pe obiecte

Nu există un consens în comunitatea de programatori referitor la ce caracteristici ar trebui să aibă un limbaj de programare pentru a fi considerat orientat pe obiecte. Rust este influențat de numeroase paradigme de programare, inclusiv OOP; de exemplu, în Capitolul 13 am examinat caracteristicile împrumutate din programarea funcțională. Se poate argumenta că limbajele OOP se caracterizează prin anumite trăsături comune, cum ar fi obiecte, încapsulare și moștenire. Să analizăm ce înseamnă aceste caracteristici și dacă Rust le implementează.

### Obiectele conțin date și comportamente

Cartea *Design Patterns: Elements of Reusable Object-Oriented Software* de Erich Gamma, Richard Helm, Ralph Johnson și John Vlissides (Addison-Wesley Professional, 1994), cunoscută informal ca și cartea *Gang of Four* (Banda celor patru), prezintă un catalog de pattern-uri de design orientate pe obiecte. Aceasta definește OOP astfel:

> Programele orientate-obiect sunt alcătuite din obiecte. Un *obiect* include
> atât date cât și procedurile care lucrează cu aceste date. Procedurile sunt
> în mod uzual numite *metode* sau *operații*.

Conform acestei definiții, Rust este un limbaj orientat pe obiecte: structurile și enum-urile conțin date, iar blocurile `impl` oferă metode pentru aceste structuri și enum-uri. Chiar dacă structurile și enum-urile cu metode nu sunt etichetate explcit ca *obiecte*, ele oferă aceeași funcționalitate din punct de vedere al definiției oferite de The Gang of Four.

### Încapsularea care ascunde detalii de implementare

Un alt aspect adesea asociat cu OOP este ideea de *încapsulare*, care presupune că detaliile de implementare ale unui obiect nu sunt accesibile codului ce utilizează acel obiect. Prin urmare, singura cale de a interacționa cu un obiect este prin API-ul său public; codul care îl folosește nu ar trebui să fie capabil să acceseze internul obiectului și să modifice direct datele sau comportamentul acestuia. Acest lucru îi permite programatorului să schimbe și să refacă structura internă a obiectului fără a avea nevoie să modifice codul ce utilizează obiectul.

Am discutat despre cum se poate controla încapsularea în Capitolul 7: putem folosi cuvântul cheie `pub` pentru a hotărî ce module, tipuri, funcții și metode din codul nostru ar trebui să fie accesibile public, iar în mod implicit, tot restul este privat. De exemplu, noi putem defini o structură `AveragedCollection` care are un câmp ce conține un vector cu valori de tip `i32`. Structura mai poate să aibă și un câmp ce reține media valorilor din vector, semnificând că media nu trebuie calculată de fiecare dată când este necesară. Altfel spus, `AveragedCollection` va memora pentru noi media calculată. Listarea 17-1 prezintă definiția structurii `AveragedCollection`:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-01/src/lib.rs}}
```

<span class="caption">Listarea 17-1: O structură `AveragedCollection` care păstrează o listă de valori întregi și media acestor valori</span>

Structura este marcată cu `pub`, astfel încât să poată fi folosită de alte coduri, însă câmpurile din cadrul structurii rămân private. Acest aspect este important întrucât dorim să ne asigurăm că atunci când o valoare este adăugată sau ștearsă din listă, media este de asemenea actualizată. Realizăm acest lucru implementând metodele `add`, `remove` și `average` pe structura respectivă, așa cum este arătat în Listarea 17-2:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-02/src/lib.rs:here}}
```

<span class="caption">Listarea 17-2: Implementarea metodelor publice `add`, `remove` și `average` pe `AveragedCollection`</span>

Metodele publice `add`, `remove` și `average` sunt singurele căi de interacțiune sau modificare a datelor într-o instanță a `AveragedCollection`. Când un element este adăugat în `list` prin metoda `add` sau eliminat utilizând `remove`, implementările acestora invocă metoda privată `update_average`, responsabilă de actualizarea câmpului `average`.

Am păstrat câmpurile `list` și `average` ca fiind private pentru a împiedica codul extern să adauge sau să elimine itemi direct din câmpul `list`; altfel, câmpul `average` ar putea deveni desincronizat atunci când lista se modifică. Metoda `average` returnează valoarea din câmpul `average`, permițând codului extern să acceseze valoarea medie, însă nu și să o modifice.

Întrucât am încapsulat detaliile de implementare ale structurii `AveragedCollection`, în viitor putem schimba cu ușurință diverse aspecte, cum ar fi structura datelor. De pildă, am putea folosi un `HashSet<i32>` în loc de un `Vec<i32>` pentru câmpul `list`. Dacă semnăturile metodelor publice `add`, `remove` și `average` rămân neschimbate, codul care folosește `AveragedCollection` nu va necesita nicio modificare. Dacă `list` ar fi fost public, situația ar fi putut fi diferită: `HashSet<i32>` și `Vec<i32>` au metode diferite pentru adăugarea și eliminarea elementelor, ceea ce ar implica probabil schimbări în codul extern dacă acesta ar modifica `list` direct.

Dacă încapsularea este un criteriu esențial pentru ca un limbaj să fie considerat orientat pe obiecte, atunci Rust satisface și această condiție. Posibilitatea de a utiliza `pub` sau nu pentru diverse secțiuni ale codului facilitează încapsularea detaliilor implementării.

### Moștenirea ca sistem de tipuri și de partajare a codului

*Moștenirea* este un mecanism prin care un obiect poate moșteni elemente definitorii ale altui obiect, câștigând, astfel, datele și comportamentul obiectului părinte fără a necesita o redefinire.

În cazul în care prezența moștenirii este o condiție obligatorie pentru ca un limbaj de programare să fie considerat orientat pe obiecte, Rust nu se încadrează în această categorie. Nu există un mod prin care să definești un struct ce moștenește câmpurile și implementările de metode ale structurii părinte fără utilizarea unui macro.

Totuși, dacă ești obișnuit să utilizezi moștenirea în setul tău de instrumente de programare, Rust îți oferă alte opțiuni, în funcție de motivul pentru care alegi să folosești moștenirea inițial.

Moștenirea este selectată din două motive principale. Primul este reutilizarea codului: poți implementa un anumit comportament pentru un tip și, prin moștenire, ai posibilitatea de a aplica aceiași implementare unui tip diferit. În Rust, acest lucru se poate realiza într-un mod restrâns folosind implementările implicite ale metodelor unei trăsături, aspect observat în Listarea 10-14, când am inclus o implementare implicită a metodei `summarize` pentru trăsătura `Summary`. Orice tip ce implementează trăsătura `Summary` va beneficia de metoda `summarize` fără a fi necesar cod adițional. Aceast aspect se aseamănă cu o clasă părinte care deține o implementare a unei metode și cu o clasă derivată care moștenește implementarea dată. Mai mult, putem suprascrie implementarea implicită a metodei `summarize` în momentul în care implementăm trăsătura `Summary`, fiind analog cu o clasă derivată care suprascrie o metodă primită prin moștenire de la o clasă părinte.

Al doilea motiv pentru a recurge la moștenire este legat de sistemul de tipuri: permite unui tip derivat să fie utilizat în toate contextele în care ar putea fi utilizat tipul părinte. Așa utilizare este cunoscută și sub numele de *polimorfism*, sugerând că, în timpul execuției, poți substitui mai multe obiecte între ele dacă partajează anumite trăsături.

> ### Polimorfism
>
> Pentru mulți, polimorfismul este văzut ca fiind sinonim cu moștenirea. Însă,
> este de fapt un concept mult mai general, care face referire la cod capabil
> să interacționeze cu date de diverse tipuri. Pentru moștenire, aceste tipuri
> sunt, în general, subclase.
>
> Rust preferă să utilizeze generici pentru a generaliza peste diferite tipuri
> posibile și delimitări de trăsătură pentru a impune constrângeri legate de
> funcționalitățile pe care aceste tipuri trebuie să le furnizeze. Așa metodă
> este uneori denumită *polimorfism parametric limitat*.

Recent, moștenirea a devenit o soluție de design mai puțin favorizată în multe limbaje de programare, deoarece adesea există riscul de a partaja mai mult cod decât este necesar. Subclasele nu ar trebui neapărat să moștenească toate caracteristicile clasei părinte, ce exact se întâmplă în cazul moștenirii. Astfel poate reduce flexibilitatea designului unui program. De asemenea, generează posibilitatea de a invoca metode pe subclase care nu sunt adecvate sau care provoacă erori deoarece metodele nu întotdeauna sunt aplicabile la subclasa dată. În plus, unele limbaje permit doar moștenire simplă (o subclasă poate moșteni numai de la o singură clasă), limitând și mai mult flexibilitatea designului de program.

Din aceste motive, Rust alege o altă cale, utilizând obiecte-trăsătură în loc de moștenire. Să analizăm cum obiectele-trăsătură facilitează polimorfismul în Rust.
