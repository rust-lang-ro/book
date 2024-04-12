## Implementarea unui pattern de design orientat pe obiecte

*Pattern-ul de stare* (state pattern) este un concept în designul orientat pe obiecte care implică definirea unui set de stări posibile pentru o anumită valoare, pe care aceasta le poate asuma intern. Aceste stări sunt reprezentate de diverse *obiecte de stare*, schimbând comportamentul respectivei valori în funcție de starea în care se află. Analizăm aici un exemplu de structură `blog post` având un câmp destinat stării sale, putând fi una dintre obiectele de stare *schiță*, *la recenzie* sau *publicată*.

În Rust, aceste obiecte de stare dețin funcționalități asemănătoare, pentru care utilizăm structuri și trăsături în loc de obiecte și moșteniri. Fiecare obiect de stare este auto-suficient în ceea ce privește comportamentul său și determină momentele de tranziție către alte stări. Valoarea care găzduiește obiectul de stare nu conține informație referitoare la comportamentele specifice stărilor sau cum să tranziționeze între ele.

Avantajul folosirii pattern-ului de stare se oglindește în flexibilitatea cu care se pot adapta noilor necesități ale aplicațiilor: atunci când apar schimbări, nu este necesară revizuirea codului valorii ce deține starea, nici a celui care o utilizează. Este suficientă modificarea codului aferent unor obiecte de stare pentru a ajusta reguli sau pentru a adăuga noi obiecte de stare, după caz.

Inițiem implementarea pattern-ului de stare cu o abordare tipic orientată obiect, urmând să transitionăm spre o metodologie mai armonioasă cu paradigma Rust. Vom dezvolta gradual un workflow pentru postările de blog, utilizând pattern-ul de stare.

Funcționalitatea obținută în final va cuprinde următoarele aspecte:

1. Un articol pe blog începe ca un schiță inițial vidă.
2. Odată completată schița, se solicită evaluarea postării.
3. Cu aprobarea postării, aceasta este gata de publicare.
4. Numai articolele de blog publicate sunt apte de a afișa conținut, facilitând astfel prevenția publicării neintenționate a celor neaprobate.

Orice alte încercări de modificare a unei postări nu ar trebui să aibă efect. De exemplu, dacă încercăm să aprobăm o schiță de articol pe blog înainte de a fi solicitat o recenzie, acest articol ar trebui să rămână în starea de schiță nepublicat.

Listarea 17-11 ilustrează acest flux de lucru sub formă de cod: acesta este un exemplu al utilizării API-ului pe care urmează să îl implementăm într-un crate de bibliotecă numit `blog`. Momentan, acest cod nu va compila deoarece crate-ul `blog` nu a fost încă implementat.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-11/src/main.rs:all}}
```

<span class="caption">Listarea 17-11: Codul care demonstrează comportamentul dorit pentru crate-ul nostru `blog`.</span>

Intenționăm să oferim posibilitatea utilizatorului de a crea o nouă postare schiță pe blog cu ajutorul `Post::new`. Vrem să permitem adăugarea textului la postarea blogului. Dacă încercăm să accesăm conținutul postării imediat, înainte de aprobare, nu ar trebui să primim niciun text, deoarece postarea este încă schiță. Pentru demonstrație, am folosit `assert_eq!` în cod. Un test unitar excelent ar fi să verificăm că o postare schiță pe blog returnează un șir de caractere gol pentru metoda `content`, dar nu vom scrie teste pentru acest exemplu.

Mai departe, ne propunem să implementăm cererea de recenzie a articolului și dorim ca `content` să furnizeze un șir de caractere gol în timpul așteptării recenziei. Când articolul este aprobat, ar trebui să fie publicat, ceea ce înseamnă că textul articolului va fi returnat la apelul lui `content`.

Este important de observat că singurul tip cu care interacționăm din crate este `Post`. Acest tip va utiliza pattern-ul de stare și va conține o valoare care va fi una dintre cele trei obiecte de stare, reprezentând stările prin care poate trece o postare: schiță, în așteptarea recenziei sau publicată. Trecerea de la o stare la alta este gestionată intern în tipul `Post`. Schimbările de stare se produc ca răspuns la metodele apelate de către utilizatorii bibliotecii asupra instanței `Post`, ei nefiind nevoiți să gestioneze direct aceste schimbări de stare. Totodată, utilizatorii nu pot să greșească stările, de exemplu, nu pot publica o postare înainte să fie recenzată.

### Definirea lui `Post` și crearea unei instanțe noi în starea de schiță

Să începem implementarea bibliotecii! Avem nevoie de o structură `Post` publică care să conțină conținut, așa că vom începe cu definirea structurii și o funcție asociată publică `new` pentru a crea o instanță de `Post`, așa cum este arătat în Listarea 17-12. Vom crea și o trăsătură privată `State` care va defini comportamentul necesar tuturor obiectelor de stare pentru `Post`.

În continuare, `Post` va conține în interior un obiect-trăsătură `Box<dyn State>` încapsulat într-un `Option<T>` într-un câmp privat numit `state` pentru a stoca obiectul de stare. Motivul pentru care este necesar `Option<T>` va deveni evident în curând.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-12/src/lib.rs}}
```

<span class="caption">Listarea 17-12: Definirea structurii `Post` și a funcției `new` care creează o nouă instanță de `Post`, o trăsătură `State`, și o structură `Draft`</span>

Trăsătura `State` stabilește comportamentul partajat de diferitele stări ale unei postări. Stările obiectelor sunt `Draft`, `PendingReview` și `Published`, toate urmând să implementeze trăsătura `State`. Deocamdată, trăsătura nu are metode definite, și vom începe prin a defini doar starea `Draft`, deoarece asta este starea inițială dorită pentru o postare.

La crearea unei noi `Post`, setăm câmpul `state` la o valoare `Some` care conține un `Box`. Acest `Box` face referire la o nouă instanță a structurii `Draft`, asigurând că orice instanță nouă de `Post` va începe ca o schiță. Fiindcă câmpul `state` din `Post` este privat, nu este posibilă crearea unui `Post` într-o altă stare! În funcția `Post::new`, inițializăm câmpul `content` ca fiind un `String` gol nou.

### Stocarea textului din conținutul postării

Am observat în Listarea 17-11 că dorim să fim capabili să apelăm o metodă numită `add_text` și să-i transmitem un `&str` care urmează să fie adăugat ca text al conținutului postării pe blog. Implementăm această funcționalitate sub forma unei metode, în loc să expunem câmpul `content` drept `pub`, pentru a putea ulterior implementa o metodă ce va controla modul în care se accesează datele câmpului `content`. Metoda `add_text` este destul de directă, așadar să adăugăm implementarea sa în Listarea 17-13, în blocul `impl Post`:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-13/src/lib.rs:here}}
```

<span class="caption">Listarea 17-13: Implementarea metodei `add_text` pentru adăugarea de text la `content`-ul unei postări</span>

Metoda `add_text` necesită o referință mutabilă la `self`, întrucât modificăm instanța de `Post` pe care o folosim pentru a apela `add_text`. Apoi invocăm `push_str` pe `String` din `content` și transmitem argumentul `text` pentru a adăuga la conținutul deja salvat. Acest comportament nu este influențat de starea în care se află postarea, deci nu este parte din pattern-ul de stare. Metoda `add_text` nu interacționează cu câmpul `state`, dar constituie o parte din comportamentul pe care îl dorim să îl suportăm.

### Asigurarea conținutului gol a unei postări schiță

Chiar dacă am invocat `add_text` și am adăugat anumit conținut postării noastre, vrem totuși ca metoda `content` să returneze un șir de caractere gol, fiindcă postarea se află încă în starea de schiță, așa cum se vede la linia 7 din Listarea 17-11. Pentru moment, să implementăm metoda `content` în cel mai simplu mod posibil care să satisfacă această cerință: returnând mereu un șir de caractere gol. Vom modifica acest lucru mai târziu, o dată ce implementăm funcționalitatea de a schimba starea unei postări pentru a permite publicarea acesteia. Până în acest punct, postările pot fi doar în starea de schiță, astfel că conținutul unei postări ar trebui să fie întotdeauna gol. Listarea 17-14 prezintă această implementare temporară:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-14/src/lib.rs:here}}
```

<span class="caption">Listarea 17-14: Implementarea temporală a metodei `content` pentru `Post`, care returnează întotdeauna un șir de caractere gol</span>

Cu această metodă `content` adăugată, tot ce se găsește în Listarea 17-11 până la linia 7 funcționează exact așa cum dorim.

### Solicitarea unei recenzii schimbă starea postării

Următorul pas este să adăugăm funcționalitatea de a solicita o recenzie a unei postări, care ar trebui să-i schimbe starea din `Draft` în `PendingReview`. Listarea 17-15 prezintă codul aferent:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-15/src/lib.rs:here}}
```

<span class="caption">Listarea 17-15: Implementarea metodelor `request_review` pentru `Post` și trăsătura `State`</span>

Îi atribuim clasei `Post` o metodă publică numită `request_review` care primește o referință mutabilă la `self`. Apoi invocăm o metodă internă `request_review` pe starea actuală a `Post`, iar această a doua metodă `request_review` consumă starea curentă și returnează o nouă stare.

Metoda `request_review` este adăugată trăsăturii `State`. Toate tipurile care implementează această trăsătură vor trebui acum să implementeze metoda `request_review`. De observat că metoda nu are ca prim parametru `self`, `&self`, sau `&mut self`, ci `self: Box<Self>`. Această sintaxă indică faptul că metoda este validă doar când este invocată pe un `Box` care deține respectivul tip. Sintaxa ia în posesie `Box<Self>`, invalidând starea veche, permițând astfel ca valoarea stării `Post` să evolueze către o nouă stare.

Pentru a renunța la vechea stare, metoda `request_review` trebuie să preia posesiunea valorii stării. Aici intervine rolul `Option` în câmpul `state` al `Post`: apelăm metoda `take` pentru a extrage valoarea `Some` din `state` și a lăsa un `None` în loc, de vreme ce Rust nu ne lasă să avem câmpuri neinițializate în structuri. Aceasta ne permite să permutăm valoarea stării din `Post`, în loc de a o împrumuta. Ulterior, stabilim valoarea stării postării la rezultatul acestei operații.

Este necesar să setăm temporar `state` la `None`, în loc să setăm direct valoarea cu un cod de genul `self.state = self.state.request_review();` pentru a deține valoarea stării. Acest lucru asigură că `Post` nu poate utiliza vechea valoare `state` după ce am convertit-o într-o nouă stare.

Metoda `request_review` de pe `Draft` returnează o instanță nouă, Box<dyn Draft>, a structurii `PendingReview`, care simbolizează starea în care o postare așteaptă recenzia. Structura `PendingReview` implementează, de asemenea, metoda `request_review`, dar nu are operațiuni de transformare. În schimb, își returnează propria instanță, deoarece atunci când solicităm o recenzie pentru o postare aflată deja în starea `PendingReview`, ea trebuie să rămână în această stare.

Acum începem să observăm avantajele modelului de stare: metoda `request_review` a clasei `Post` este identică, indiferent de valoarea lui `state`. Fiecare stare își determină propriile sale reguli.

Metoda `content` a clasei `Post` este lăsată neschimbată, returnând o secțiune de string goală. Acum putem avea o postare nu doar în starea `Draft`, dar și în `PendingReview`, dorind același comportament în ambele stări. Listarea 17-11 este acum aplicabilă până la linia 10!

<!-- Old headings. Do not remove or links may break. --> <a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>

### Adăugarea metodei `approve` pentru a schimba comportamentul `content`

Metoda `approve` va fi similară metodei `request_review`: va seta `state` la valoarea indicată de starea curentă ca fiind necesară după ce e aprobată, conform Listării 17-16:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-16/src/lib.rs:here}}
```

<span class="caption">Listarea 17-16: Implementarea metodei `approve` pe `Post` și trăsătura `State`</span>

Adăugăm metoda `approve` la trăsătura `State` și introducem o nouă structură ce implementează `State`, starea `Published`.

Similar cu funcționarea lui `request_review` pe `PendingReview`, dacă invocăm metoda `approve` pe un `Draft`, aceasta nu va avea efect fiindcă `approve` va returna `self`. Când invocăm `approve` pe `PendingReview`, metoda returnează o nouă instanță a structurii `Published` încapsulată în `Box`. Structura `Published` implementează trăsătura `State`, iar pentru metodele `request_review` și `approve` returnează propria instanță, pentru că articolul ar trebui să rămână în starea `Published` în acele situații.

Acum trebuie să actualizăm metoda `content` a lui `Post`. Vrem ca valoarea întoarsă de `content` să depindă de starea curentă a lui `Post`, așadar vom delega responsabilitatea unei metode `content` definite pe `state`, așa cum e prezentat în Listarea 17-17:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-17/src/lib.rs:here}}
```

<span class="caption">Listarea 17-17: Actualizarea metodei `content` de pe `Post` pentru a delega la o metodă `content` definită pe `State`</span>

Cu scopul de a menține toate regulile în interiorul structurilor care implementează `State`, invocăm o metodă `content` pe valoarea din `state` și pasăm instanța postului (adică `self`) ca argument. Apoi, returnăm valoarea primită din aplicarea metodei `content` asupra valorii `state`.

Folosim metoda `as_ref` pe `Option` pentru că dorim o referință la conținutul `Option`, nu posesiunea acestei valori. Deoarece `state` este un `Option<Box<dyn State>>`, prin apelul lui `as_ref`, obținem un `Option<&Box<dyn State>>`. Fără utilizarea lui `as_ref`, am întâmpina o eroare pentru că nu putem muta `state` din `&self` împrumutat, care este parametrul funcției.

Apelăm apoi metoda `unwrap`, metodă despre care știm că nu va provoca panică, pentru că metodele definite pe `Post` garantează că `state` va conține întotdeauna o valoare `Some` la finalul executării lor. Acesta este unul dintre scenariile menționate în secțiunea [„Situatii în care deții mai multe informații decât compilatorul”][more-info-than-rustc] din Capitolul 9, când suntem siguri că o valoare `None` nu este niciodată posibilă, chiar dacă compilatorul nu are capacitatea să recunoască acest lucru.

În momentul în care folosim `content` pe `&Box<dyn State>`, va interveni coerciția de dereferențiere asupra `&` și `Box` astfel încât metoda `content` va fi apelată, în ultimă instanță, pe tipul ce implementează trăsătura `State`. Acest lucru ne obligă să adăugăm `content` în definiția trăsăturii `State`, unde vom defini logica privind conținutul ce trebuie returnat în funcție de starea curentă, conform Listării 17-18:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-18/src/lib.rs:here}}
```

<span class="caption">Listarea 17-18: Adăugarea metodei `content` în definiția trăsăturii `State`</span>

Introducem o implementare implicită pentru metoda `content` care oferă un șir gol. Acest lucru înseamnă că nu mai este necesar să implementăm `content` pentru structurile `Draft` și `PendingReview`. Structura `Published` va înlocui metoda `content` și va oferi valoarea din `post.content`.

Să notăm că este nevoie de adnotări de durată de viață pentru această metodă, aspect abordat în Capitolul 10. Primim o referință la `post` ca argument și returnăm o referință la o parte din acest `post`, astfel că durata de viață a referinței întoarse este corelată cu durata de viață a argumentului `post`.

Și astfel am încheiat — întreaga Listare 17-11 este acum funcțională! Am implementat pattern-ul stării conform regulilor pentru publicarea unui articol pe blog. Logica asociată cu aceste reguli este encapsulată în obiectele de stare, în loc să fie dispersată prin `Post`.

> #### De ce nu o Enumerare?
>
> Probabil te întrebi de ce nu am ales să utilizăm un `enum` cu diferite stări
> posibile ale postării ca și variante. Aceasta ar fi fost sigur o soluție
> fezabilă; încearcă și compară cu rezultatele finale pentru a vedea care
> variantă îți convine mai mult! Un neajuns al utilizării unei enumerări este
> că ar necesita un `match` sau o structură similară pentru a gestiona fiecare
> posibilă variantă oriunde se verifică valoarea enumerării. Implementare ce
> ar putea fi mai redundantă decât soluția bazată pe obiect-trăsătură.

### Compromisurile pattern-ului de stare

Am demonstrat că Rust poate să implementeze pattern-ul de stare specific orientării obiectuale pentru a încapsula diferite comportamente ale unei postări conform stării în care se află. Metodele pe `Post` sunt neinformate despre varietatea comportamentelor. Prin modul în care am structurat codul, e destul să privim într-un singur loc pentru a cunoaște comportamentele diverse ale unei postări publicate: implementarea trăsăturii `State` în structura `Published`.

Dacă am alege să realizăm o implementare alternativă care nu recurge la pattern-ul de stare, probabil am folosi expresii `match` fie în metodele pe `Post`, fie chiar în codul din `main`, pentru a verifica starea postării și a modifica comportamentul acolo unde este necesar. Asta ar însemna că ar trebui să ne uităm în mai multe locuri pentru a înțelege toate implicațiile unei postări în starea de *publicat*! Și complexitatea ar crește odată cu adăugarea de noi stări: fiecare expresie `match` ar necesita o nouă ramură.

Folosind pattern-ul de stare, metodele lui `Post` și contextele în care este folosit nu necesită expresii `match`; pentru a adăuga o nouă stare, trebuie doar să introducem o nouă structură și să implementăm metodele respective ale trăsăturii.

O implementare care folosește pattern-ul de stare este simplu de extins pentru a adăuga funcționalități noi. Pentru a aprecia ușurința mentenanței codului ce utilizează acest pattern, încearcă următoarele sugestii:

* Introdu o metodă `reject` ce schimbă starea postării de la `PendingReview` înapoi la `Draft`.
* Impune necesitatea efectuării a două apeluri la `approve` pentru schimbarea stării în `Published`.
* Permite utilizatorilor să adauge conținut text doar când o postare este în starea `Draft`. Indiciu: lasă obiectul de stare să fie responsabil pentru ceea ce ar putea schimba din conținut, dar fără a modifica `Post`.

Un neajuns al pattern-ului de stare este acela că, având în vedere implementarea transițiilor între stări în interiorul stărilor, anumite stări devin interdependente. Dacă am decide să adăugăm o stare intermediară între `PendingReview` și `Published`, ca de exemplu `Scheduled`, am fi nevoiți să modificăm codul din `PendingReview` pentru a tranziționa spre `Scheduled` și nu direct spre `Published`. Ar fi mai simplu dacă `PendingReview` nu ar necesita adaptări când se adaugă o nouă stare, dar asta ar implica alegerea unui alt pattern de design.

O altă problemă este duplicarea unor logici: pentru a reduce redundanța, am putea încerca să stabilim implementări implicite ale metodelor `request_review` și `approve` în trăsătura `State` care să returneze `self`. Cu toate acestea, am încălca siguranța obiectelor, deoarece trăsătura nu poate determina cu precizie ce va fi `self`. Vrem ca `State` să poată fi folosit ca un obiect-trăsătură, deci e esențial ca metodele sale să respecte siguranța obiectelor.

Alte forme de duplicare includ implementări asemănătoare ale metodelor `request_review` și `approve` din `Post`, în care ambele metode se bazează pe implementarea aceleiași metode pe valoarea din câmpul `state` al `Option`, stabilind noua valoare a câmpului `state`. Dacă metodele din `Post` sunt numeroase și urmează acest pattern, am putea lua în considerare crearea unui macro pentru a elimina repetițiile (consultă secțiunea "Macrouri" în Capitolul 19).

Prin adoptarea pattern-ului de stare așa cum este el definit în limbajele cu orientare obiectuală, nu valorificăm toate avantajele lui Rust. Să analizăm unele modificări pe care le-am putea aplica crate-ului `blog` pentru a transforma stările și tranzițiile invalide în erori de timpul compilării.

#### Codificarea stării și comportamentului în tipuri de date

Vom arăta cum să reconceptualizăm pattern-ul de stare pentru a accesa un set diferit de compromisuri. În loc să încapsulăm complet stările și tranzițiile astfel încât codul din exterior să nu le cunoască, vom codifica stările în diferite tipuri de date. Ca rezultat, sistemul de verificare a tipurilor din Rust va împiedica încercările de a utiliza schițe de postări acolo unde sunt permise doar postările publicate, generând o eroare de compilare.

Să analizăm prima parte a funcției `main` din Listarea 17-11:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-11/src/main.rs:here}}
```

Permiterea creării de noi postări în starea de schiță prin intermediul `Post::new` și abilitatea de a adăuga text conținutului postării rămân neschimbate. Dar, în loc să avem o metodă `content` pe o postare schiță care să întoarcă un string gol, vom aranja astfel încât schițele de postări pur și simplu să nu dispună de metoda `content`. În acest mod, dacă încercăm să accesăm conținutul unei postări schițe, vom primi o eroare de compilare care ne indică faptul că metoda nu există. Prin urmare, ne va fi imposibil să afișăm din greșeală conținutul unei schițe de postare în producție, deoarece codul respectiv nu ar trece de compilare. Listarea 17-19 oferă definiția unei structuri `Post` și a unei structuri `DraftPost`, împreună cu metodele fiecăreia:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-19/src/lib.rs}}
```

<span class="caption">Listarea 17-19: Un `Post` cu metoda `content` și un `DraftPost` fără metoda `content`</span>

Atât structura `Post`, cât și `DraftPost` includ un câmp `content` privat, care stochează textul postării de pe blog. Structurile nu mai conțin câmpul `state`, deoarece acțiunea de codificare a stării se mută la nivelul tipurilor de date ale structurilor. Structura `Post` va reprezenta o postare publicată și are metoda `content` care returnează `content`.

Avem încă funcția `Post::new`, dar în loc să returneze o instanță de `Post`, ea returnează o instanță de `DraftPost`. Fiind privat și fără funcții care să returneze un `Post`, nu este posibilă crearea unei instanțe de `Post` în acest moment.

Structura `DraftPost` dispune de metoda `add_text`, permițându-ne să adăugăm text la `content` ca și înainte, însă este de remarcat faptul că `DraftPost` nu are definită metoda `content`! Așadar, programul garantează că toate postările încep ca schițe, iar conținutul schițelor nu este disponibil pentru afișare. Oricărei încercări de a evita aceste restricții i se va opune o eroare de compilator.

#### Implementarea tranzițiilor ca transformări în diferite tipuri

Deci, cum obținem un post publicat? Ne dorim să impunem regula că o schiță de post trebuie să fie revizuită și aprobată înainte să poată fi publicată. Un post aflat în stadiul de revizuire în așteptare nu ar trebui să afișeze vreun conținut. Să realizăm aceste constrângeri prin adăugarea unei noi structuri, `PendingReviewPost`, definind metoda `request_review` în `DraftPost` pentru a returna un `PendingReviewPost`, și creând metoda `approve` în `PendingReviewPost` pentru a returna un `Post`, așa cum este ilustrat în Listarea 17-20:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-20/src/lib.rs:here}}
```
<span class="caption">Listarea 17-20: Un `PendingReviewPost` care e creat prin apelarea `request_review` asupra unui `DraftPost` și o metodă `approve` care transformă un `PendingReviewPost` într-un `Post` publicat</span>

Metodele `request_review` și `approve` preiau controlul asupra variabilei `self`, consumând astfel instanța de `DraftPost`, respectiv `PendingReviewPost`, și transformând-o într-un `PendingReviewPost` și ulterior într-un `Post` publicat. Astfel, nu rămânem cu instanțe de `DraftPost` după ce folosim `request_review` pe acestea și așa mai departe. Structura `PendingReviewPost` nu are o metodă `content` definită, astfel, încercarea de a citi conținutul său duce la o eroare de compilare, similar cu `DraftPost`. Fiindcă singura cale de a avea o instanță de `Post` publicat, ce are o metodă `content`, este prin apelarea `approve` pe un `PendingReviewPost`, și singura metodă de a obține un `PendingReviewPost` este prin `request_review` aplicată la un `DraftPost`. Aceste proceduri ne permit să codificăm fluxul de lucru pentru postările de blog în cadrul sistemului de tipuri.

Totuși, sunt necesare unele ajustări în `main`. Metodele `request_review` și `approve` generează noi instanțe, în loc să modifice structurile asupra cărora sunt invocate, astfel avem nevoie să adăugăm noi atribuiri `let post =` pentru a salva instanțele rezultate. Nu mai este posibil să avem aserțiuni despre conținuturi goale pentru schițe sau posturi aflate în revizuire, și nici nu sunt necesare: codul care încerca să acceseze conținutul posturilor în acele stări nu mai poate fi compilat. Codul actualizat din `main` este prezentat în Listarea 17-21:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-21/src/main.rs}}
```
<span class="caption">Listarea 17-21: Modificările necesare în `main` pentru a folosi noua abordare a procesului de postare pe blog</span>

Modificările pe care le-am făcut în `main` pentru reatribuirea variabilei `post` indică faptul că această implementare nu mai respectă cu strictețe modelul de stare orientat pe obiecte: transformările între stări nu sunt complet încapsulate în implementarea lui `Post`. Cu toate acestea, avantajul obținut este că stările invalide devin imposibile datorită sistemului de tipuri și a verificării tipurilor efectuate la compilare! Aceasta garantează că anumite erori, cum ar fi afișarea conținutului unui post nepublicat, sunt detectate înainte de lansarea în producție.

Aplică sarcinile sugerate la începutul acestei secțiuni pe crate-ul `blog` după cum arată în urma Listării 17-21 pentru a evalua designul acestei versiuni de cod. Vei vedea că unele sarcini ar putea fi deja îndeplinite de acest design.

Am observat că, deși Rust permite implementarea modelelor de design orientate pe obiecte, sunt și alte pattern-uri disponibile, cum ar fi codificarea stării în sistemul de tipuri, ce pot fi exploatate. Aceste pattern-uri au diverse compromisuri. Deși s-ar putea să fii obișnuit cu pattern-urile orientate obiect, reconsiderarea problemei pentru a valorifica caracteristicile Rust poate aduce beneficii suplimentare, cum ar fi prevenirea anumitor erori chiar în faza de compilare. Nu întotdeauna modelele obiectuale vor reprezenta cea mai optimă soluție în Rust, având în vedere trăsături unice ale limbajului, precum sistemul de posesiune, care nu sunt întâlnite în limbajele obiectuale tradiționale.

## Sumar

Indiferent dacă ai concluzionat sau nu că Rust este un limbaj orientat pe obiecte după lectura acestui capitol, acum știi că poți folosi obiecte-trăsătură pentru a accesa unele dintre caracteristicile orientate pe obiecte în Rust. Invocarea dinamică oferă codului flexibilitate, costând însă puțin din performanța la rulare. Această flexibilitate îți permite să implementezi pattern-uri orientate pe obiecte care pot îmbunătăți întreținerea codului. Rust dispune și de alte caracteristici, cum ar fi posesiunea, ce nu sunt prezente în limbajele orientate pe obiecte. Utilizarea unui pattern orientat pe obiecte nu va fi mereu cea mai bună metodă de a valorifica puterea Rust, însă este o alternativă posibilă.

În continuare, ne vom aprofunda în studiul pattern-urilor, care reprezintă o altă facilitate a Rust ce permite o gamă largă de flexibilitate. Am abordat pattern-urile în treacăt pe tot parcursul cărții, dar acum urmează să le vedem întregul potențial. Să avansăm în explorare!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch19-06-macros.html#macros
