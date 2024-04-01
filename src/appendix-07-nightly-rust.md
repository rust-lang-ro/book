## Anexa G - Procesul de dezvoltare al Rust și „Nightly Rust”

Acest apêndice explică modul în care este dezvoltat Rust și impactul acestui proces asupra ta ca dezvoltator Rust.

### Stabilitate fără stagnare

Rust acordă o *importanță mare* stabilității codului tău. Vrem ca Rust să fie o bază solidă de dezvoltare pentru tine și dacă lucrurile ar fi în schimbare continuă, nu ar fi posibil. Totodată, dacă nu putem experimenta cu funcționalități noi, s-ar putea să descoperim defecte semnificative abia după lansarea acestora, când nu ne mai este permis să modificăm ceva.

Soluția noastră la această dilemă se numește "stabilitate fără stagnare", iar principiul călăuzitor este următorul: nu ar trebui să îți fie teamă vreodată să efectuezi un upgrade la o nouă versiune stabilă de Rust. Fiecare upgrade ar trebui să fie simplu și să îți ofere noi funcționalități, un număr mai mic de erori, precum și o viteză mai mare de compilare.

### Choo, Choo! Canale de lansare și metoda trenurilor

Dezvoltarea Rust se derulează conform unui *program de tren*. Adică, întreaga dezvoltare se realizează pe ramura `master` al depozitului de cod Rust. Lansările sunt efectuate pe baza modelului trenurilor software, folosit de Cisco IOS precum și de alte proiecte. Există trei *canale de lansare* în cadrul Rust:

* Nightly (Nocturn)
* Beta
* Stable (Stabil)

Majoritatea dezvoltatorilor Rust preferă canalul stabil, dar cei interesați să exploreze funcții noi și experimentale pot opta pentru Nightly Rust sau Beta.

Să examinăm un exemplu specific despre cum se desfășoară procesul de dezvoltare și lansare: să presupunem că echipa Rust lucrează la lansarea versiunii 1.5. Deși lansarea a avut loc în decembrie 2015, ne va oferi exemple adecvate de numere de versiune. O caracteristică nouă este introdusă în Rust: un nou commit este adăugat pe ramura `master`. Fiecare noapte se creează o nouă versiune Nightly Rust. Fiecare zi înseamnă o lansare, iar aceste lansări sunt generate în mod automat de infrastructura noastră de lansări. Astfel, odată cu trecerea timpului, lansările arată cam așa, noapte de noapte:

```text
nightly: * - - * - - *
```

La fiecare șase săptămâni, este timpul să pregătim o nouă lansare! Ramura `beta` a depozitului Rust se ramifică din ramura `master` utilizată de Nightly. Acum, există două lansări:

```text
nightly: * - - * - - *
                     |
beta:                *
```

Cei mai mulți utilizatori Rust nu folosesc activ lansările beta, dar testează împotriva beta în sistemul lor de integrare continuă pentru a ajuta Rust să descopere posibile regresii. Între timp, tot apare o lansare Nightly în fiecare noapte:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Să presupunem că se găsește o regresie. Bine că am avut ceva timp să testăm lansarea beta înainte ca regresia să se strecoare într-o lansare stabilă! Remediul se aplică pe `master`, astfel încât Nightly este reparat, apoi remedierea este retroportată la ramura `beta`, și se produce o nouă lansare beta:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

La șase săptămâni după crearea primei beta, este timpul pentru o lansare stable! Ramura `stable` este produsă din ramura `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Excelent! Rust 1.5 este gata! Totuși, am uitat un lucru: pentru că au trecut șase săptămâni, avem nevoie și de o nouă beta a versiunii *următoare* de Rust, 1.6. Așadar, după ce `stable` se ramifică din `beta`, următoarea versiune de `beta` se ramifică din nou din `nightly`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Acesta se numește modelul „de tren” pentru că la fiecare șase săptămâni, o lansare „părăsește stația”, dar tot trebuie să parcurgă un drum prin canalul beta înainte de a ajunge ca o lansare stabilă.

Rust se lansează la fiecare șase săptămâni, cu precizia unui ceasornic. Dacă cunoști data unei lansări Rust, poți determina data celei următoare: va fi peste șase săptămâni. Un avantaj al acestei programări la fiecare șase săptămâni este că „următorul tren” va sosi în curând. Dacă o funcționalitate nu reușește să ajungă într-o anumită lansare, nu e cazul să te îngrijorezi: o nouă lansare va fi disponibilă în curând! Asta diminuează presiunea de a adăuga funcționalități poate nefinisate în apropierea datei de lansare.

Datorită acestui proces, mereu poți testa următoarea construcție a Rust și să te convingi că este simplu de trecut la ea: dacă o versiune beta nu funcționează conform așteptărilor, poți să raportezi asta echipei și să obții corectarea înainte de următoarea lansare stabilă! Incidența problemelor într-o versiune beta este relativ rară, dar `rustc` rămâne o piesă de software, așa că erorile pot apărea.

### Caracteristici instabile

Există un aspect suplimentar de luat în seamă în ce privește modelul de lansare menționat: caracteristicile instabile. Rust utilizează o tehnică numită „flag-uri de funcționalitate” pentru a defini ce caracteristici sunt disponibile într-o versiune anume. Când o nouă caracteristică este încă în dezvoltare, aceasta este adăugată în `master` și, astfel, este disponibilă în Nightly Rust, dar protejată de un *flag de funcționalitate*. Dacă dorești să testezi o funcționalitate aflată în dezvoltare, ai posibilitatea, însă este necesar să utilizezi versiunea Nightly Rust și să adnotezi codul sursă cu flag-ul potrivit pentru a exprima acordul tău.

În cazul în care folosești o versiune beta sau stabilă Rust, nu ai posibilitatea de a folosi flag-uri de funcționalitate. Acest lucru reprezintă cheia ce ne permite să testăm în mod practic noile caracteristici înainte de a fi declarate stabile pe termen lung. Cei care doresc să fie la curent cu tehnologia de ultimă oră pot adopta variantele instabile, în timp ce cei doritori de o experiență robustă pot să rămână pe versiunea stabilă și să fie siguri că codul lor nu va avea probleme. Stabilitate fără stagnare.

Cartea de față include doar informații despre caracteristicile stabile pentru că cele în curs de elaborare sunt încă în schimbare și cu certitudine vor fi diferite față de cum sunt descrise aici, la momentul când vor fi implementate în versiunile stabile. Poți găsi documentație pentru caracteristicile disponibile doar în versiunea Nightly Rust pe internet.

### Rustup și rolul lui Rust Nightly

Rustup facilitează trecerea între diferite canale de release ale Rust, fie la nivel global, fie pentru fiecare proiect în parte.În mod implicit, ai instalat Rust în versiunea stabilă. Pentru a instala Nightly Rust, de exemplu:

```console
$ rustup toolchain install nightly
```

Poți vedea, de asemenea, toate *toolchain-urile* (lansările de Rust și componentele asociate) instalate cu `rustup`. Iată un exemplu de pe computerul cu Windows al unuia dintre autorii cărții:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Cum poți observa, toolchain-ul stabil este cel selectat implicit. Majoritatea utilizatorilor Rust preferă versiunea stabilă în cele mai multe situații. Este posibil să dorești să folosești în principal versiunea stabilă, dar să alegi Nightly Rust pentru un proiect anume, pentru că ai nevoie de o caracteristică avansată. În acest caz, poți utiliza `rustup override` în directoriul respectivului proiect pentru a specifica toolchain-ul Nightly ca fiind cel dorit de `rustup` atunci când lucrezi în acel directoriu:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Astfel, ori de câte ori rulezi `rustc` sau `cargo` în *~/projects/needs-nightly*, `rustup` va asigura că folosești Nightly Rust, în loc de Rust stabil, care este setarea default. Aceasta este o facilitare importantă atunci când lucrezi cu mai multe proiecte Rust.

### Procesul RFC și echipele

Cum afli despre aceste noi caracteristici? Dezvoltarea Rust se bazează pe un *proces de Request For Comments (RFC)*. Dacă dorești o îmbunătățire în Rust, poți scrie o propunere, denumită RFC.

Oricine poate redacta RFC-uri pentru a aduce îmbunătățiri limbajului Rust, iar aceste propuneri sunt revizuite și discutate de echipa Rust, alcătuită din multiple subechipe specializate pe diverse teme. O listă completă a acestor echipe este disponibilă [pe website-ul Rust](https://www.rust-lang.org/governance), care include echipe pentru fiecare domeniu al proiectului: design de limbaj, implementarea compilatorului, infrastructură, documentație și altele. Echipa relevantă analizează propunerea și comentariile, contribuie cu observații proprii și, în final, se ajunge la un consens pentru acceptarea sau respingerea caracteristicii propuse.

Dacă funcția este acceptată, se deschide un incident în repository-ul Rust, iar cineva poate să o implementeze. Persoana care o implementează nu trebuie neapărat să fie aceea care a propus funcția la început! Când implementarea este terminată, aceasta este adăugată în ramura `master` sub controlul unui flag de funcționalitate, așa cum am explicat în secțiunea [„Caracteristici instabile”](#unstable-features)<!-- ignore -->.

După ce dezvoltatorii Rust care folosesc versiunea Nightly Rust au avut posibilitatea să experimenteze noua caracteristică, membrii echipei o evaluează, discutând despre performanța ei în cadrul versiunii Nightly, și iau o decizie privind includerea sa în versiunea stabilă de Rust sau nu. Dacă se decide promovarea caracteristicii, flag-ul acesteia este îndepărtat, și astfel devine o funcție stabilă! Astfel, este integrată în următoarea versiune stabilă a Rust.
