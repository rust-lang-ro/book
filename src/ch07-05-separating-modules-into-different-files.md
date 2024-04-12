## Separarea modulelor în diferite fișiere

Toate exemplele prezentate până acum în acest capitol au învățat cum să definim mai multe module într-un singur fișier. Când modulele devin mai complexe, ai putea dori să separi definițiile acestora în fișiere diferite pentru a naviga mai ușor prin cod.

Ca punct de plecare, vom lua codul din Listarea 7-17 care conține multiple module *restaurant*. Scopul este să extragem modulele în diferite fișiere, în loc să le păstrăm toate în fișierul rădăcină al crate-ului. În cazul de față, fișierul rădăcină este *src/lib.rs*. Cu toate acestea, același proces se aplică și crate-urilor binare, unde fișierul rădăcină este *src/main.rs*.

Începem prin a delega modulul `front_of_house` în propriul fișier. Pentru asta, eliminăm codul din interiorul acoladelor modulului `front_of_house`, lăsând doar declarația `mod front_of_house;`. Astfel, *src/lib.rs* va conține codul prezentat în Listarea 7-21. Atenție: acest cod nu va putea fi compilat până când nu vom crea fișierul *src/front_of_house.rs*, așa cum este prezentat în Listarea 7-22.

<span class="filename">Numele fișierului: src/lib.rs</span>

```rust, nu_se_compileaza,ignore
{{#includefile_rustdoc ../lista/ch07-gestionand-proiecte-in-crestere/listare-07-21-si-22/src/lib.rs}}
```

<span class="caption">Listarea 7-21: Declararea modulului `front_of_house` care va fi conținut în *src/front_of_house.rs*</span>

În etapa următoare, trebuie să mută codul din interiorul acoladelor într-un fișier nou, numit *src/front_of_house.rs*, conform Listării 7-22. Compilatorul va găsi acest fișier datorită declarației de modul din rădăcina crate-ului, unde se specifică numele `front_of_house`.


<span class="filename">Numele fișierului: src/front_of_house.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

<span class="caption">Listarea 7-22: Definim anumite aspecte în interiorul modulului `front_of_house` localizat în *src/front_of_house.rs*</span>

Reține că trebuie să încarci un fișier folosind o declarație `mod` *o singură dată* în arborele tău de module. Odată ce compilatorul este informat că fișierul aparține proiectului (și cunoaște locul în care se găsește codul datorită poziției declarației `mod` ), restul fișierelor din proiect trebuie să se refere la codul fișierului încărcat utilizând o cale spre locul declarării acestuia. Această metodă este descrisă în secțiunea [„Cum să referi un element în structura modulelor”][paths]<!-- ignore -->. În cuvinte mai simple, `mod` *nu este* o comandă de "include", așa cum este utilizată în alte limbaje de programare.

Următorul pas va fi mutarea modulului `hosting` în propriul său fișier. Procesul este ușor diferit deoarece `hosting` este un submodul al `front_of_house`, nu un modul rădăcină. Noua locație a fișierului va fi un nou directoriu, care va purta numele submodulului în structura de module - în cazul nostru, *src/front_of_house/*.

Pentru mutarea `hosting`, modificăm *src/front_of_house.rs* pentru a găzdui doar declarația modulului `hosting`:

<span class="filename">Numele fișierului: src/front_of_house.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

Apoi, creăm directoriul *src/front_of_house* și fișierul *hosting.rs*, care va conține definițiile din cadrul modulului `hosting`:

<span class="filename">Numele fișierului: src/front_of_house/hosting.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

Daca am plasa *hosting.rs* în directoriul *src*, compilatorul s-ar aștepta ca fișierul *hosting.rs* să fie un modul rădăcină `hosting`  din structura proiectului, și nu un submodul al `front_of_house`. Regulile interne ale compilatorului referitoare la asocierea fișierelor cu modulele specifice fac ca structura de directorii și fișiere să urmeze cu exactitate structura de module.

> ### Căi alternative de fișiere
>
> Până în prezent, am discutat despre căile de fișiere cel mai des utilizate în
> Rust, dar trebuie să știi că acest limbaj de programare acceptă și un stil mai
> vechi de definire a acestora. Să luăm exemplul unui modul numit
> `front_of_house`, pe care îl avem declarat la rădăcina unui crate -
> compilatorul Rust va căuta codul acestuia în următoarele locații:
> * *src/front_of_house.rs* (am discutat deja despre aceasta)
> * *src/front_of_house/mod.rs* (o variantă mai veche, care este încă acceptată)
> 
> Referitor la un modul numit `hosting`, definit ca submodul al
> `front_of_house`, compilatorul va căuta codul său în:
> * *src/front_of_house/hosting.rs* (am menționat deja acesta)
> * *src/front_of_house/hosting/mod.rs* (încă o versiune veche, încă valabilă)
>
> Dacă alegi să utilizezi ambele stiluri pentru același modul, vei obține o
> eroare din partea compilatorului. Deși ești liber să utilizezi o combinație a
> celor două stiluri pentru diferite module din același proiect, aceasta ar
> putea genera confuzii pentru cei care explorează proiectul.
>
> Unul dintre dezavantajele metodei care implică utilizarea de fișiere numite
> *mod.rs* este că proiectul poate ajunge să conțină un număr mare de astfel de
> fișiere, ceea ce poate fi confuz atunci când le avem deschise simultan în
> editor.
>
> Noutatea este că acum avem codul fiecărui modul într-un fișier separat, dar
> structura modulelor în sine nu s-a schimbat. Chiar dacă acum definițiile lor
> se află în fișiere diferite, apelurile funcției în `eat_at_restaurant` vor
> funcționa în mod normal, fără a necesita vreo modificare. Aceasta ne permite
> să mutăm modulele în fișiere noi pe măsură ce acestea se măresc în dimensiune.
>
> Observăm că linia `pub use crate::front_of_house::hosting` din *src/lib.rs* nu
> a fost afectată, la fel cum nici folosirea cuvântului cheie `use` nu
> influențează fișierele care sunt compilate ca parte a crate-ului. Cuvântul
> cheie `mod` este folosit pentru a declara module, iar Rust va căuta codul
> acestora în fișiere care au același nume precum modulul în sine.

## Sumar

Rust îți oferă posibilitatea de a diviza un pachet în crate-uri multiple și un crate în diverse module. Acest lucru permite referirea la elemente definite într-un anumit modul dintr-un altul. Această acțiune poate fi realizată prin specificarea unor căi absolute sau relative. Cu ajutorul instrucțiunii `use`, aceste căi pot fi aduse în domeniul de vizibilitate, permițând astfel un acces mai rapid și mai eficient. Deși codul unui modul este inițial privat, acesta poate fi făcut public adăugând termenul `pub`.

În capitolul următor, vom explora unele structuri de colecție din librăria standard, pe care le vei putea folosi în codul tău bine organizat.

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
