# Introducere

> Notă: Această ediție a cărții este la fel ca [The Rust Programming
> Language][nsprust] disponibilă în format print și ebook de la [No Starch
> Press][nsp].

[nsprust]: https://nostarch.com/rust-programming-language-2nd-edition 
[nsp]: https://nostarch.com/

Bine ai venit la *The Rust Programming Language*, o carte introductivă despre limbajul de programare Rust. Rust te ajută să dezvolți software mai rapid și mai fiabil. Ergonomia avansată și controlul fin al detaliilor sunt adesea văzute ca fiind în contradicție în design-ul limbajelor de programare; Rust își propune să armonizeze aceste aspecte. Echilibrând puterea tehnică avansată cu o experiență de dezvoltare agreabilă, Rust te lasă să gestionezi detalii precise ale utilizării memoriei și alte aspecte la nivel scăzut, fără dificultățile obișnuit asociate cu acest tip de control.

## Pentru cine este destinat Rust

Rust este ideal pentru numeroase persoane, din diverse motive. Să vedem câteva dintre cele mai importante categorii.

### Echipe de dezvoltatori

Rust se afirmă drept un instrument eficient pentru colaborarea în cadrul echipelor mari de dezvoltatori cu nivel diferit de experiență în programarea sistemelor. Codul low-level este susceptibil la o multitudine de bug-uri subtile, care în majoritatea celorlalte limbaje pot fi identificate doar prin teste riguroase și recenzii meticuloase ale codului efectuate de dezvoltatori experimentați. În Rust, compilatorul acționează ca o santinelă, refuzând să compileze codul care conține aceste erori subtile, inclusiv erorile de concurență. Colaborând îndeaproape cu compilatorul, echipa poate folosi timpul mai eficient pentru a se concentra pe logica programului decât pe urmărirea defectelor.

Rust contribuie la lumea programării de sistem cu setul său de instrumente de dezvoltare de ultimă oră:

* Cargo, managerul de dependențe și instrumentul de build încorporat, simplifică adăugarea, compilarea și coordonarea dependențelor într-un mod fără dureri de cap și omogen pe tot ecosistemul Rust.
* Instrumentul de formatare Rustfmt asigură un stil unic de codificare dintre dezvoltatori.
* Rust Language Server potențează integrarea în Integrated Development Environment (IDE) pentru auto-completarea codului și mesajele de eroare contextualizate.

Utilizând aceste facilități și altele din ecosistemul Rust, dezvoltatorii pot fi foarte eficienți în timpul scrierii codului la nivel de sistem.

### Studenți

Rust se adresează studenților și tuturor celor interesați să învețe despre conceptele de sistem. Prin Rust, mulți au aprofundat teme legate de dezvoltarea sistemelor de operare. Comunitatea este extrem de primitoare și dornică să răspundă la întrebările studenților. Prin intermediul acestei cărți și alte inițiative, echipele Rust își propun să faciliteze accesul la conceptele de sistem unui număr și mai mare de persoane, în special celor care sunt la începutul drumului în programare.

### Companii

Sute de companii, atât mari, cât și mici, folosesc Rust în producție pentru diverse activități, precum crearea de unelte pentru linia de comandă, servicii web, unelte pentru DevOps, dispozitive embedded, analiza și transcodarea audio-video, criptomonede, bioinformatică, motoare de căutare, aplicații pentru Internet of Things, machine learning și chiar componente esențiale ale browserului web Firefox.

### Dezvoltatori open source

Rust este dedicat celor care vor să contribuie la dezvoltarea limbajului de programare Rust, a comunității, uneltelor pentru dezvoltatori și a bibliotecilor. Te așteptăm cu brațele deschise să contribui la dezvoltarea limbajului Rust.

### Persoanele care prețuiesc viteza și stabilitatea

Rust se adresează celor care tânjesc după viteză și stabilitate într-un limbaj de programare. Prin viteză, ne referim atât la cât de repede poate rula codul Rust, cât și la rapiditatea cu care Rust te ajută să elaborezi programe. Verificările făcute de compilatorul Rust asigură stabilitatea prin introducerea de noi funcționalități și refactorizarea. Această abordare contrastează cu codul legacy fragil din limbaje fără aceste verificări, cod pe care dezvoltatorii adeseori se tem să-l modifice. Prin urmărirea abstracțiilor cu zero supra-cost, funcții de nivel înalt care se compilează la cod de nivel jos la fel de rapid ca și codul scris manual, Rust își propune să asigure că codul sigur este și cod rapid.

Limbajul Rust își propune să sprijine și alți utilizatori, pe lângă cei menționați; aceștia sunt doar unii dintre cei mai importanți. Per ansamblu, cea mai mare ambiție a Rust este să elimine compromisurile la care programatorii au consimțit de decenii, oferind în același timp siguranță *și* productivitate, viteză *și* ergonomie. Încearcă Rust și vezi dacă deciziile sale se potrivesc cu nevoile tale.

## Pentru cine este această carte

Această carte presupune că ai deja experiență în scrierea de cod într-un alt limbaj de programare, dar nu se bazează pe presupunerea că ai folosit unul anume. Am adaptat materialul pentru a fi accesibil persoanelor cu experiențe diverse în programare. Nu ne-am concentrat în detaliu asupra discuției despre natura programării sau despre abordarea gândirii în termeni de programare. Dacă ești începător în domeniul programării, ai beneficia mai mult de pe urma unei cărți care oferă o introducere specifică în acest domeniu.

## Cum să folosești această carte

De regulă, această carte este concepută să fie citită în ordine, de la prima pagină până la ultima. Capitolul următor se bazează pe conceptele prezentate în capitolele anterioare, și capitolele anterioare pot să nu abordeze în detalii un anumit subiect, ci să revină asupra lui într-un capitol mai înaintat.

Vei găsi două tipuri de capitole în această carte: capitole de concepte și capitole de proiecte. În capitolele de concepte, vei afla detalii despre un aspect al limbajului Rust. În capitolele de proiecte, vom dezvolta împreună programe mici, aplicând ceea ce ai învățat până în momentul acela. Capitolele 2, 12 și 20 sunt capitole de proiecte; celelalte sunt capitole de concepte.

Capitolul 1 explică cum să instalezi Rust, cum să scrii un program „Hello, world!” și cum să folosești Cargo, managerul de pachete și uneltele de construire din Rust. Capitolul 2 oferă o introducere practică în scrierea unui program Rust prin construirea unui joc de ghicit numere. Acolo tratăm conceptele la un nivel mai general, iar capitolele viitoare vor intra în mai multe detalii. Dacă ești nerăbdător să începi practic, Capitolul 2 este ideal pentru acest lucru. Capitolul 3 tratează facilitățile din Rust comune cu alte limbaje de programare, iar în Capitolul 4 vei învăța despre sistemul specific Rust de posesiune. Dacă ești o persoană extrem de meticuloasă și preferi să cunoști fiecare detaliu înainte de a avansa, ai putea să sari peste Capitolul 2 și să treci direct la Capitolul 3, revenind la Capitolul 2 când vrei să exersezi pe un proiect, aplicând detaliile învățate.

Capitolul 5 discută structurile și metodele, iar Capitolul 6 prezintă enumerările, expresiile `match` și structura de control `if let`. În Rust, vei folosi structurile și enumerările pentru a defini tipuri de date personalizate.

În Capitolul 7, vei învăța despre sistemul de module al Rust și despre regulile de confidențialitate pentru structurarea codului tău și a Interfeței de Programare a Aplicațiilor (API) publice. Capitolul 8 abordează unele structuri de date de colecții uzuale pe care le furnizează biblioteca standard, cum ar fi vectorii, string-urile și hashmap-urile. Capitolul 9 tratează filosofia și tehnicile de gestionare a erorilor în Rust.

Capitolul 10 aprofundează conceptele de generici, trăsături și durate de viață, ce îți oferă capacitatea de a defini cod util pentru diferite tipuri. Capitolul 11 are în centrul atenției testarea, necesară pentru a verifica corectitudinea raționamentului logic al programului tău, chiar dacă Rust oferă garanții de siguranță. În Capitolul 12, vom crea propria variantă a unei părți din funcționalitățile comenzii `grep`, care permite căutarea textului în fișiere, folosind multe din conceptele prezentate în capitolele precedente.

Capitolul 13 introduce închiderile și iteratorii, elemente din Rust influențate de limbajele de programare funcționale. Capitolul 14 este dedicat unei analize detaliate a Cargo și discuții despre practicile ideale pentru distribuirea bibliotecilor tale către alții. Capitolul 15 prezintă pointerii inteligenți disponibili în biblioteca standard și trăsăturile care le susțin funcționalitatea.

Capitolul 16 te va ghida prin diferitele paradigme de programare concurentă și va explora cum Rust facilitează programarea pe multe fire de execuție fără temeri. Capitolul 17 compară expresiile Rust cu principiile familiale ale programării orientate-obiect. 

Capitolul 18 servește drept ghid pentru pattern-uri și potrivirea pattern-urilor, metode eficace de comunicare a ideilor în cadrul programelor Rust. Capitolul 19 oferă o gamă largă de subiecte avansate captivante, printre care Rust-ul nesigur, macrocomenzile, precum și aspecte suplimentare despre duratele de viață, trăsăturile, tipurile, funcțiile și închiderile.

În Capitolul 20, vom completa un proiect în care vom implementa un server web de nivel scăzut cu fire de execuție multiple!

În final, unele anexe conțin informații utile despre limbaj într-un format mai apropiat de cel de referință. Anexa A este despre cuvintele cheie din Rust, Anexa B despre operatorii și simbolurile din Rust, Anexa C despre trăsăturile derivabile oferite de biblioteca standard, Anexa D despre unele instrumente de dezvoltare utile și Anexa E explică edițiile Rust. În Anexa F poți găsi traduceri ale cărții, iar în Anexa G vom explica cum este dezvoltat Rust și ce înseamnă Nightly Rust.

Nu e nicio metodă greșită de a citi această carte: dacă dorești să sari peste unele părți, fă-o! Poate fi nevoie să te întorci la capitolele anterioare dacă întâmpini greutăți. Dar fă ceea ce ți se potrivește.

<span id="ferris"></span>

O parte importantă a procesului de învățare Rust este să înveți cum să citești mesajele de eroare afișate de compilator: acestea te vor îndruma către un cod funcțional. Prin urmare, vom prezenta multe exemple care nu se compilează, alături de mesajul de eroare pe care compilatorul îl va afișa în fiecare situație. Înainte de a introduce și rula un exemplu la întâmplare, ține cont că s-ar putea să nu se compileze! Asigură-te că ai citit textul înconjurător pentru a vedea dacă exemplul pe care încerci să îl execuți este menit să genereze o eroare. Ferris te va ajuta să distingi și codul care nu este destinat să funcționeze:

| Ferris                                                                                                           | Semnificație                                     |
|------------------------------------------------------------------------------------------------------------------|--------------------------------------------------|
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris cu un semn de întrebare"/>         | Codul nu se compilează!                          |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris cu mâinile în aer"/>                         | Codul generează panică!                          |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris cu o labă în sus, mirat"/>     | Codul nu are comportamentul așteptat.            |

În majoritatea situațiilor, te vom îndruma către versiunea corectă a oricărui cod care nu compilează.

## Codul sursă

Fișierele sursă din care această carte a fost generată pot fi găsite pe [GitHub][book].

[book]: https://github.com/rust-lang/book/tree/main/src
