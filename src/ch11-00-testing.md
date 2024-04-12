# Testare automatizată în Rust

În eseu din 1972 intitulat "The Humble Programmer", Edsger W. Dijkstra remarcă faptul că
"Testarea programelor poate fi un mijloc foarte eficace de a evidenția prezența defectelor, însă
este total inadecvată pentru a garanta absența lor." Aceasta nu înseamnă că
nu ar trebui să ne străduim să testăm tot ce putem!

Corectitudinea în programele noastre se referă la măsura în care codul nostru efectuează ceea ce ne-am propus
să realizeze. Rust este conceput cu o atenție sporită asupra corectitudinii
programelor, dar această corectitudine este complexă și dificil de atestat. Sistemul de tipuri din Rust
poartă o bună parte din această responsabilitate, însă sistemul de tipuri nu poate intercepta
toate problemele. Din acest motiv, Rust oferă suport pentru scrierea de teste automate.

Luăm cazul în care compunem o funcție `add_two` care adaugă 2 la numărul primit ca argument.
Această funcție are o semnătură care primește un integer ca parametru și returnează un
integer ca rezultat. Implementând și compilând această funcție, Rust efectuează toate
verificările de tip și de împrumut pe care le-am studiat până acum pentru a ne asigura că, de exemplu, nu introducem un `String` sau o referință nevalidă
în această funcție. Dar, Rust *nu poate* confirma dacă funcția va executa exact
ce dorim noi, care este să returneze parametrul adunat cu 2 în locul parametrului adunat cu 10 sau scăzut cu 50! Pentru acest lucru sunt indispensabile testele.

Putem crea teste care susțin, de pildă, că atunci când pasăm `3` la
funcția `add_two`, ieșirea este `5`. Aceste teste le putem executa ori de câte ori
facem schimbări la cod pentru a verifica faptul că niciun comportament corect deja existent
nu a fost afectat.

Testarea este o capacitate complexă: deși nu putem acoperi toate detaliile despre cum se
scriu teste de înaltă calitate într-un singur capitol, vom trata mecanismele sistemului de testare din Rust. Vom discuta despre adnotațiile și macro-urile pe care le aveți la dispoziție când redactați teste, comportamentul implicit și opțiunile de rulare a testelor, și cum să clasificăm testele în teste de unitate și teste de integrare.
