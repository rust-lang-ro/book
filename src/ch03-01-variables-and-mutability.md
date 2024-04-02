## Variabile și mutabilitate

Așa cum am menționat în secțiunea [„Păstrarea valorilor cu variabile”][storing-values-with-variables]<!-- ignore -->, în mod implicit, variabilele sunt imutabile. Acesta este unul dintre numeroasele stimulente pe care Rust ți le oferă pentru a scrie codul tău într-un mod care să profite de siguranța și concurența ușoară pe care Rust le oferă. Cu toate acestea, ai încă opțiunea de a-ți face variabilele mutabile. Să explorăm cum și de ce Rust încurajează preferința pentru imutabilitate și de ce uneori ai putea vrea să alegi variabilele mutabile.

Când o variabilă este imutabilă, odată ce o valoare este atribuită unui nume, nu poți schimba acea valoare. Pentru a ilustra acest lucru, generează un nou proiect numit *variables* în directoriul tău *projects* utilizând `cargo new variables`.

Apoi, în noul tău directoriu *variables*, deschide *src/main.rs* și înlocuiește codul său cu următorul cod, care încă nu se va compila:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

Salvează și rulează programul folosind `cargo run`. Ar trebui să primești un mesaj de eroare privind o eroare de imutabilitate, așa cum se arată în această ieșire:
```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

Acest exemplu arată cum compilatorul te ajută să găsești erorile din programele tale. Erorile de compilare pot fi frustrante, dar în realitate ele înseamnă doar că programul tău nu face încă în siguranță ceea ce vrei să facă; ele *nu* înseamnă că nu ești un bun programator! Rustaceanii cu experiență tot primesc erori de compilare.

Ai primit mesajul de eroare ``cannot assign twice to immutable variable `x` `` pentru că ai încercat să atribui o a doua valoare variabilei imutabile `x`.

Este important să primim erori la timpul compilării când încercăm să schimbăm o valoare care este desemnată ca imutabilă, deoarece această situație poate duce la bug-uri. Dacă o parte a codului nostru funcționează pe presupunerea că o valoare nu se va schimba niciodată și o altă parte a codului nostru schimbă acea valoare, este posibil ca prima parte a codului să nu facă ceea ce a fost proiectat să facă. Cauza acestui tip de bug poate fi dificil de urmărit ulterior, mai ales când a doua parte a codului schimbă valoarea doar *uneori*. Compilatorul Rust garantează că atunci când declari că o valoare nu se va schimba, ea cu adevărat nu se va schimba, astfel încât nu trebuie să urmărești asta singur. Astfel, codul tău este mai ușor de înțeles.

Dar mutabilitatea poate fi foarte utilă și poate face codul mai convenabil de scris. Deși variabilele sunt imutabile în mod implicit, le poți face mutabile adăugând `mut` în fața numelui variabilei, așa cum ai făcut în [Capitolul 2][storing-values-with-variables]<!-- ignore -->. Adăugarea `mut` transmite și intenția cititorilor viitori ai codului, indicând că alte părți ale codului vor schimba această valoare a variabilei.

De exemplu, să schimbăm *src/main.rs* în următorul mod:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

Când rulăm acum programul, obținem asta:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

Ni se permite să modificăm valoarea legată la `x` de la `5` la `6` când este utilizat `mut`. În cele din urmă, decizia de a utiliza sau nu mutabilitatea depinde de tine și de ceea ce crezi că este cel mai evident în acea situație particulară.

### Constante

La fel ca variabilele imutabile, *constantele* sunt valori care sunt legate de un nume și nu li se permite să se schimbe, dar există câteva diferențe între constante și variabile.

În primul rând, nu ai voie să folosești `mut` cu constante. Constantele nu sunt doar imutabile în mod implicit - sunt întotdeauna imutabile. Declari constante folosind cuvântul cheie `const`, în loc de cuvântul cheie `let`, iar tipul valorii *trebuie* să fie adnotat. Vom acoperi tipurile și adnotările de tip în următoarea secțiune, [„Tipuri de date”][data-types]<!-- ignore -->, așa că nu-ți face griji despre aceste detalii acum. Trebuie doar să știi că trebuie întotdeauna să adnotezi tipul.

Constantele pot fi declarate în orice domeniul de vizibilitatea, inclusiv în cel global, ceea ce le face utile pentru valorile de care multe părți ale codului au nevoie să știe.

Ultima diferență este că constantele pot fi setate numai la o expresie constantă, nu la rezultatul unei valori care ar putea fi calculate doar în timpul rulării.

Iată un exemplu de declarare a unei constante:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Numele constantei este `THREE_HOURS_IN_SECONDS` și valoarea sa este setată la rezultatul înmulțirii cu 60 (numărul de secunde într-un minut) cu 60 (numărul de minute într-o oră) cu 3 (numărul de ore pe care dorim să le numărăm în acest program). Convenția de denumire a Rust pentru constante este să folosească doar majuscule cu subliniere între cuvinte. Compilatorul este capabil să evalueze un set limitat de operațiuni la momentul compilării, ceea ce ne permite să alegem să scriem această valoare într-un mod mai ușor de înțeles și de verificat, decât să setăm această constantă la valoarea 10,800. Vezi [secțiunea Rust Reference privind evaluarea constantă][const-eval] pentru mai multe informații despre ce operațiuni pot fi utilizate la declararea constantelor.

Constantele sunt valabile pentru întreaga durată a rulării unui program, în cadrul în care au fost declarate. Această proprietate face ca constantele să fie utile pentru valorile din domeniul aplicației tale despre care multe părți ale programului ar putea avea nevoie să știe, cum ar fi numărul maxim de puncte pe care orice participant al unui joc are voie să le câștige, sau viteza luminii.

Specificând valorile particulare folosite într-un program ca fiind constante este util în a transmite semnificația acelei valori către viitorii întreținători ai codului. De asemenea, este util să avem doar un singur loc în codul nostru unde ar trebui să modificăm dacă eventual valoarea codificată ar trebui să fie actualizată în viitor.

### Umbrirea

Cum am văzut în tutorialul jocului de ghicit în capitolul 2, putem declara o nouă variabilă cu același nume ca o variabilă anterioară. Rustacenii spun că prima variabilă este *umbrită* de a doua, ceea ce înseamnă că a doua variabilă este cea pe care compilatorul o va vedea atunci când folosești numele variabilei. Practic, a doua variabilă eclipsează prima, preluând orice utilizări ale numelui variabilei până când este la rândul său umbrită sau până când se termină contextul. Putem umbri o variabilă folosind același nume de variabilă și repetând utilizarea cuvântului cheie `let` astfel:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

Acest program leagă întâi `x` la o valoare de `5`. Apoi creează o nouă variabilă `x` repetând `let x =`, luând valoarea originală și adăugând `1`, astfel încât valoarea `x` este apoi `6`. Apoi, într-un context intern creat cu acolade, a treia declarație `let` umbrește, de asemenea, `x` și creează o nouă variabilă, înmulțind valoarea anterioară cu `2`, pentru a da `x` o valoare de `12`. Când acest context se termină, umbrirea internă se termină și `x` revine la a fi `6`. Când rulăm acest program, va afișa următoarele:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

Umbrirea este diferită de marcarea unei variabile ca `mut`, deoarece vom obține o eroare la compilare dacă încercăm accidental să realocăm această variabilă fără a folosi cuvântul cheie `let`. Folosind `let`, putem efectua câteva transformări pe o valoare, dar avem variabila imutabilă după ce aceste transformări au fost finalizate.

Cea de-a doua diferență între `mut` și umbrire este că, deoarece creăm efectiv o nouă variabilă atunci când folosim din nou cuvântul cheie `let`, putem schimba tipul valorii, dar refolosim același nume. De exemplu, spunem că programul nostru cere unui utilizator să arate câte spații dorește între un text prin introducerea de caractere spațiu și apoi dorim să stocăm această intrare ca număr:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

Prima variabilă `spaces` este de tip string și a doua variabilă `spaces` este de tip număr. Astfel, umbrirea ne scutește să găsim nume diferite, cum ar fi `spaces_str` și `spaces_num`; în schimb, putem reutiliza numele mai simplu `spaces`. Cu toate acestea, dacă încercăm să folosim `mut` pentru acest lucru, așa cum se arată aici, vom obține o eroare la timpul de compilare:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

Eroarea spune că nu avem voie să modificăm tipul unei variabile:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

Acum că am explorat cum funcționează variabilele, să examinăm mai multe tipuri de date pe care le pot avea.

[comparing-the-guess-to-the-secret-number]:
ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html
