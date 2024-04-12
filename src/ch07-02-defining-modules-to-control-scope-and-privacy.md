## Stabilirea modulelor pentru a gestiona domeniul de vizibilitate și confidențialitatea

In cadrul acestei secțiuni, vom discuta despre module și despre alte componente ale sistemului de module, în special *calea* care permite denumirea elementelor; cuvântul cheie `use` care introduce o cale în domeniul de vizibilitate; și cuvântul cheie `pub`, utilizat pentru a face elemente publice. Vom aborda, de asemenea, cuvântul cheie `as`, pachete externe și operatorul glob.

Înainte de toate, vom începe cu o listă de reguli pe care le poți consulta ușor atunci când vă organizați codul în viitor. Ulterior, vom explica fiecare regulă în detaliu.

### Ghid rapid despre Module

Următoarea recapitulare oferă o privire lucrurilor esențiale despre cum modulele, căile, cuvântul-cheie `use`, precum și cuvântul-cheie `pub` colaborează cu compilatorul și cum, de obicei, dezvoltatorii își structurează codul. Deși vom discuta fiecare din aceste reguli, pe parcursul acestui capitol, referirea la acest ghid este un mod eficient de reamintire a modului în care funcționează modulele.

- **Începând de la rădăcina crate-ului**: În momentul compilării unui crate, compilatorul începe de la fișierul rădăcina al crate-ului (care este, de obicei, *src/lib.rs* pentru un crate de tip bibliotecă sau *src/main.rs* pentru un crate binar) în căutare de cod pentru a-l compila.  
- **Declararea modulelor**: În fișierul rădăcină, poți introduce noi module. Să zicem că declarăm un modul numit "garden" cu `mod garden;`. Compilatorul va căuta codul asociat acestui modul în următoarele locații:  
  - În mod direct în cod, folosind acolade în locul semicolonului, după `mod garden`  
  - În fișierul *src/garden.rs*  
  - În fișierul *src/garden/mod.rs*
- **Declararea submodulelor**: În orice alt fișier în afara celui rădăcină, poți declara submodule. De exemplu, ai putea declara `mod vegetables;` în *src/garden.rs*. Compilatorul va căuta codul asociat acestui submodul în directoriul numit după modulul părinte în locațiile următoare:
  - Direct în cod, folosind acolade în locul semicolonului după `mod vegetables`,  
  - În fișierul *src/garden/vegetables.rs*  
  - În fișierul *src/garden/vegetables/mod.rs*  
- **Referirea către codul în module**: Odată ce un modul este inclus în crate-ul tău, poți face referință la codul acestuia din orice alt loc în același crate, atât timp cât regulile de confidențialitate permit, folosind calea către acel cod. De exemplu, un tip `Asparagus` din modulul de legumă a grădinii ar putea fi referit folosind următoarea cale: `crate::garden::vegetables::Asparagus`.
- **Privat versus public**: În mod implicit, codul din cadrul unui modul este privat față de modulele părinte. Pentru a face un modul public, se declară folosind `pub mod` în loc de `mod`. Pentru a face item-urile din interiorul unui modul public să fie, de asemenea, publice, folosește `pub` înaintea declarațiilor acestora.
- **Cuvântul-cheie `use`**: Utilizat într-un domeniu de vizibilitate, cuvântul-cheie `use` creează scurtături spre item-uri pentru a minimiza repetarea de căi lungi. În orice domeniu de vizibilitate care face referire la `crate::garden::vegetables::Asparagus`, poți crea o scurtătură folosind `use crate::garden::vegetables::Asparagus;`. După aceea, vei avea nevoie doar să scrii `Asparagus` pentru a putea folosi acest tip în cadrul domeniului de vizibilitate.

În acest exemplu, vom crea un crate binar numit "backyard" pentru a ilustra aceste reguli. Directoriul asociat cu crate-ul, care se numește și el "backyard", va conține următoarele fișiere și directoare:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

Fișierul principal al acestui crate se numește *src/main.rs* și conține următorul cod:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

Linia `pub mod garden;` le indică compilatorului să adauge codul pe care îl găsește în fișierul
*src/garden.rs*. Codul din acest fișier este următorul:

<span class="filename">Numele fișierului: src/garden.rs</span>

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

De asemenea, linia `pub mod vegetables;` anunță compilatorul că trebuie să includă codul din fișierul *src/garden/vegetables.rs*. Codul din respectivul fișier este:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

Acum, să detaliem aceste reguli și să le ilustrăm în acțiune!

### Organizarea codului în module

*Modulele* ne oferă posibilitatea de a structura codul în interiorul unui crate, în scopul facilitării citirii și reutilizării acestuia. Prin intermediul modulelor, putem controla *gradul de confidențialitate* al elementelor, deoarece, în mod implicit, codul din interiorul unui modul este privat. Elementele private reprezintă detalii de implementare interne și nu pot fi utilizate în afara modulului. Cu toate acestea, avem opțiunea de a transforma modulele și elementele lor în entități publice, ceea ce le face accesibile pentru a fi utilizate și interconectate cu alte coduri externe.

Ca exemplu de utilizare, să presupunem că dorim să scriem un crate de bibliotecă care să reprezinte funcționalitatea unui restaurant. Vom defini semnăturile funcțiilor și vom lăsa corpurile lor goale, pentru a ne concentra pe structurarea codului, mai degrabă decât pe detaliile de implementare ale funcționării restaurantului.

În industria restaurantelor, unele zone ale unui restaurant sunt denumite *zona din față a casei* și, respectiv, *zona din spate a casei*. Zona din față a casei include spațiile în care sunt primiți clienții: locul în care ospătarii îi îndrumă pe aceștia la mese, preiau comenzi și încasează contravaloarea serviciilor, iar barmanii prepară băuturile. Zona din spate a casei este locul în care chefii și bucătarii își desfășoară activitatea în bucătărie, personalul se ocupă de curățenie, iar managerii își îndeplinesc sarcinile administrative.

Pentru a structura crate-ul nostru într-un mod optim, avem posibilitatea de a organiza funcțiile sale în module încorporate. Începe prin a crea o nouă bibliotecă pe care o vom numi `restaurant`. Acest lucru se poate realiza prin executarea comenzi `cargo new restaurant --lib`. Apoi, introdu codul afișat în Listarea 7-1 în fișierul *src/lib.rs*. Acest pas ne va permite definirea unor module și semnăturilor funcțiilor aferente. Să aruncăm o privire asupra secțiunii frontale a casei:

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

<span class="caption">Listarea 7-1: Un modul `front_of_house` ce include alte
module ce conțin, la rândul lor, diverse funcții</span>

Un modul se definește folosind cuvântul-cheie `mod`, urmat de numele ales pentru modulul respectiv -ăm făcut acest lucru mai sus cu `front_of_house`. Corpul modulului este închis între acolade. În interiorul modulelor, avem posibilitatea de a plasa alte module, exact cum am procedat cu modulele `hosting` și `serving`. Nu în ultimul rând, modulele pot găzdui definiții pentru alte tipuri de elemente precum structuri, enumerări, constante, trăsături și, așa cum am ilustrat în Listarea 7-1, funcții.

Utilizând module, avem posibilitatea de a grupa definițiile corelate și de a justifica această corelare. În acest mod, programatorii care vor folosi codul vor putea naviga mai ușor în funcție de grupele de definițiile create, în loc să fie nevoiți să parcurgă toate definițiile. Astfel devine mai simplu de identificat definițiile care sunt relevante pentru ei. De asemenea, când adaugă noi funcționalități în cod, vor ști exact unde să le plaseze pentru a menține organizarea programului.

Am menționat anterior că fișierele *src/main.rs* și *src/lib.rs* le denumim rădăcinile crate-urilor. Această denumire vine de la faptul că, conținutul oricăruia dintre aceste două fișiere, formează un modul numit `crate`, situat la baza structurii de module a crate-ului. Aceasta e cunoscută sub denumirea de *arbore de module*.

Listarea 7-2 prezintă arborele de module pentru structura prezentată în Listarea 7-1.
```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

<span class="caption">Listarea 7-2: Structura ierarhică a modulelor pentru codul din Listarea 7-1</span>

Acest arbore ilustrează modul în care anumite module sunt incluse în altele; spre exemplu, modulul `hosting` este inclus în `front_of_house`. De asemenea, structura evidențiază că anumite module sunt *simultane* sau *coexistente*, adică sunt definite în același modul; `hosting` și `serving` sunt astfel de module simultane, definite în cadrul `front_of_house`. Dacă modulul A este inserat în modulul B, spunem că A este *descendent* al lui B iar B este *asendent* sau *părinte* pentru A. Observă că întregul arbore de module își are rădăcina în modulul implicit denumit `crate`.

Structura modulelor s-ar putea să-ți evoce structura ierarhică a directoriilor pe calculatorul tău; este o comparație extrem de adecvată! Așa cum utilizezi directoriile într-un sistem de operare pentru a-ți organiza fișierele, tot astfel folosești modulele pentru a-ți structura codul. Și exact ca în cazul fișierelor dintr-un directoriu, avem nevoie de o modalitate de a localiza modulele noastre.