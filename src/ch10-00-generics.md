# Tipuri generice, trăsături și durate de viață

Fiecare limbaj de programare dispune de unelte specifice pentru a gestiona eficient duplicarea conceptelor. În Rust, un asemenea instrument sunt *genericele*: substituenți abstracți pentru tipuri concrete sau alte caracteristici. Ne permite să descriem comportamentul genericelor sau relațiile acestora cu alte generice fără a cunoaște ce le va înlocui atunci când codul este compilat și executat.

Funcțiile pot accepta parametri de orice tip generic, în locul unui tip concret precum `i32` sau `String`, similar modului în care o funcție acceptă parametri cu valori necunoscute pentru a rula același cod peste multiple valori concrete. De fapt, am utilizat genericele în Capitolul 6 cu `Option<T>`, în Capitolul 8 cu `Vec<T>` și `HashMap<K, V>`, și în Capitolul 9 cu `Result<T, E>`. În acest capitol, vei explora cum să-ți definești propriile tipuri, funcții și metode utilizând generice!

Vom începe cu o recapitulare a metodei de extragere a unei funcții în scopul reducerii duplicării de cod. Vom folosi aceeași tehnică pentru a deriva o funcție generică din două funcții care diferă numai prin tipurile parametrilor lor. Îți vom arăta cum se aplică tipurile generice în definițiile structurilor și enumerărilor.

Mai departe, vei învăța să folosești *trăsături* (trait) pentru a defini comportamente într-un mod generic. Combinând trăsăturile cu tipuri generice, putem limita un tip generic să accepte doar acele tipuri care prezintă un anumit comportament, spre deosebire de orice tip.

La final, vom discuta despre *duratele de viață*: o categorie specială de generice ce furnizează compilatorului date privind modul în care referințele interacționează unele cu altele. Duratele de viață ne permit să înzestrăm compilatorul cu informații suficiente despre valorile împrumutate, permițându-i acestuia să asigure că referințele sunt valide într-o paletă mai largă de scenarii decât ar fi posibil fără sprijinul nostru.

## Reducerea duplicării prin extragerea unei funcții

Genericele ne permit să substituim tipuri specifice cu un placeholder, ce reprezintă mai multe tipuri, și astfel să eliminăm duplicarea în cod. Înainte de a ne familiariza cu sintaxa de generice, să vedem prima dată cum putem înlătura duplicările într-un mod care nu implică tipuri generice. Acest lucru se realizează prin extragerea unei funcții care înlocuiește valorile specifice cu un placeholder pentru multiple valori. Apoi, vom aplica aceeași abordare pentru a defini o funcție generică! Înțelegând cum să recunoaștem codul duplicat care se poate transforma într-o funcție, vei începe să identifici și codul duplicat care poate beneficia de generice.

Pornim cu un program simplu prezentat în Listarea 10-1, care determină cel mai mare număr dintr-o listă.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

<span class="caption">Listarea 10-1: Identificarea celui mai mare număr dintr-o listă de numere</span>

Noi salvăm o listă de numere întregi în variabila `number_list` și atribuim o referință la primul număr din listă variabilei `largest`. Procedăm apoi la iterarea prin fiecare număr din listă, iar dacă numărul curent este mai mare decât cel referențiat de `largest`, actualizăm referința din această variabilă. Pe de altă parte, dacă numărul curent este mai mic sau egal cu cel mai mare număr întâlnit până în acel moment, variabila `largest` rămâne neschimbată, iar codul continuă cu următorul număr din listă. După ce toate numerele din listă sunt evaluate, `largest` va indica cel mai mare număr, care în acest exemplu este 100.

Ne-am propus acum să găsim cel mai mare număr din două seturi diferite de numere. Putem alege să duplicăm codul din Listarea 10-1 sau să aplicăm aceeași logică în două locuri diferite din program, așa cum este prezentat în Listarea 10-2.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

<span class="caption">Listarea 10-2: Cod pentru identificarea celui mai mare număr din *două* liste de numere</span>

Chiar dacă acest cod este funcțional, duplicarea sa este monotonă și predispusă la erori. În plus, trebuie să ne amintim să actualizăm codul în diverse locuri atunci când dorim să facem modificări.

Pentru a elimina această duplicare, vom introduce o abstracție prin definirea unei
funcții care operează pe orice array de numere întregi dat ca parametru. Această metodă îmbunătățește claritatea codului și ne permite să formulăm conceptul de identificare a celui mai mare număr dintr-un array într-un mod abstract.

În Listarea 10-3, extragem codul care identifică cel mai mare număr într-o funcție numită `largest`. Apoi invocăm funcția pentru a determina cel mai mare număr din cele două liste prezentate în Listarea 10-2. Totodată, putem utiliza funcția pentru orice alte array-uri de valori `i32` pe care le-am putea avea în viitor.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

<span class="caption">Listarea 10-3: Cod abstractizat pentru găsirea celui mai mare număr din două liste</span>

Funcția `largest` are un parametru denumit `list`, care poate reprezenta orice secțiune concretă de valori `i32` introdusă în funcție. În consecință, când apelăm funcția, codul se execută folosind valorile specifice furnizate.

Rezumând, aceștia sunt pașii pe care i-am urmat pentru a transforma codul din Listarea 10-2 în Listarea 10-3:

1. Identifică codul duplicat.
2. Extrage codul duplicat în corpul funcției și specifică
   intrările și valorile de retur ale acestui cod în semnătura funcției.
3. Actualizează cele două instanțe de cod duplicat pentru a apela funcția în locul acestuia.

În continuare, vom folosi acești pași împreună cu genericele pentru a reduce și mai mult duplicarea codului. La fel cum corpul unei funcții poate opera pe o `list` abstractă și nu pe valori concrete, genericele ne permit să lucrăm cu tipuri abstracte.

De pildă, să ne imaginăm că avem două funcții: una care determină elementul cel mai mare dintr-o secțiune de valori `i32` și una care găsește cel mai mare element într-o secțiune de valori `char`. Cum am putea elimina această duplicare? Să explorăm!