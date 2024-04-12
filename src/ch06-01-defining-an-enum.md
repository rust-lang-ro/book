## Definirea unei enumerări

Structurile ne oferă o modalitate de a grupa câmpuri și date conexe. De exemplu, un `Rectangle` cu `width` și `height`. În contrast, enumerările ne permit să expresăm ideea că o valoare poate fi una dintre diversele opțiuni posibile. De pildă, am putea dori să specificăm că `Rectangle` este doar una dintre posibilele forme, care mai cuprinde și `Circle` și `Triangle`. Rust ne permite să transpunem aceste posibilități în cod prin intermediul unei enumerări.

Pentru a ilustra, să luăm în considerare o situație care ar avea nevoie să fie exprimată în cod pentru a înțelege de ce enumerările sunt utile și mai potrivite decât structurile. Să presupunem că dorim să lucrăm cu adrese IP. În prezent, există două standarde principale pentru adrese IP: versiunea patru și versiunea șase. Acestea sunt singurele variante de adrese IP cu care programul nostru va interacționa, deci putem *enumera* toate aceste variante, lucru de unde provine termenul de enumerare.

Orice adresă IP poate fi fie de versiunea patru, fie de versiunea șase, dar nu ambele simultan. Această caracteristică a adreselor IP face ca structura de date de tip enumerare să fie adecvată cazului nostru, pentru că o valoare de tip enumerare poate fi doar una dintre variantele sale. Adresele de versiunea patru și șase sunt, în esență, adrese IP, deci ar trebui considerate ca fiind de același tip atunci când codul gestionează situații care se aplică la ambele tipuri de adrese IP.

Acest concept poate fi transpus în cod prin definirea unei enumerări `IpAddrKind` și enumerarea tipurilor potențiale pe care o adresă IP le poate avea, adică `V4` și `V6`. Acestea sunt variantele enumerării:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

Acum, `IpAddrKind` este un tip de date personalizat pe care îl putem utiliza în alte părți ale codului nostru.

### Valori enum

Putem genera instanțe pentru ambele variante ale `IpAddrKind` astfel:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

Observă că variantele enum-ului sunt îngrădite în spațiul de nume al identificatorului său, iar noi utilizăm două puncte pentru a le separa. Acest aspect este util fiindcă, acum, ambele valori `IpAddrKind::V4` și `IpAddrKind::V6` sunt de același tip: `IpAddrKind`. Astfel, putem, de exemplu, să definim o funcție care poate primi orice `IpAddrKind`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

Iar această funcție poate fi apelată cu oricare dintre variantele enumerate:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

Utilizarea enum-urilor aduce și mai multe beneficii. Dacă ne gândim mai atent la tipul adreselor noastre IP, momentan nu avem o modalitate de a stoca efectiv datele adreselor IP; cunoaștem doar *tipul* acestora. Dat fiind că tocmai ai aflat despre structuri în Capitolul 5, ai putea fi tentat să soluționezi această problemă utilizând structurile, așa cum este exemplificat în Listarea 6-1.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

<span class="caption">Listarea 6-1: Încapsularea datelor și a variantei `IpAddrKind`
ale unei adrese IP folosind o structură</span>

În aceste rânduri, am definit o structură numită `IpAddr`. Aceasta conține două câmpuri: un câmp `kind`, care este de tip `IpAddrKind` (enumerarea pe care am stabilit-o mai devreme) și un câmp `address` de tip `String`. Avem două instanțe ale acestei structuri. Prima se numește `home` și cuprinde valoarea `IpAddrKind::V4` atribuită câmpului `kind`, având asociată adresa `127.0.0.1`. Cea de-a doua instanță, `loopback`, conține cealaltă variantă a `IpAddrKind` pentru câmpul `kind`, respectiv `V6`, și este asociată cu adresa `::1`. Am utilizat o structură pentru a grupa valorile `kind` și `address`, astfel încât acum varianta este legată direct de valoarea ei.

Cu toate acestea, putem exprima același concept într-un mod mai concis doar cu ajutorul unei enumerări: în loc să încapsulăm o enumerare într-o structură, putem plasa datele direct în fiecare variantă a enumeraței. Această nouă definiție a enumerației `IpAddr` indică faptul că ambele variante `V4` și `V6` vor fi asociate cu valori de tip `String`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

Atașăm datele direct variantei respective a enum-ului, astfel eliminând necesitatea unei structuri suplimentare. Tot aici, este mult mai simplu să înțelegem un alt aspect legat de funcționarea enum-urilor: numele fiecărei variante de enum pe care o definim se transformă de asemenea într-o funcție ce construiește o instanță a acelui enum. Adică, `IpAddr::V4()` reprezintă un apel de funcție care primește un argument de tip `String` și returnează o instanță a tipului `IpAddr`. Obținem în mod automat această funcție constructor ca rezultat al definirii enum-ului.

Mai există un avantaj în utilizarea unui enum în locul unei structuri: fiecare variantă poate avea diferite tipuri și cantități de date asociate. Adresele IP de versiunea patru vor avea mereu patru componente numerice cu valori între 0 și 255. Dacă am dori să stocăm adresele `V4` sub forma a patru valori `u8`, dar să reprezentăm în același timp adresele `V6` sub forma unei valori `String`, aceasta nu ar fi posibilă cu o structură. Cu toate acestea, enum-urile gestionează cu ușurință acest caz:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

Am prezentat mai multe metode prin care se pot defini structurile de date pentru a stoca adresele IP de versiunea patru și versiunea șase. Cu toate acestea, se pare că necesitatea de a stoca adrese IP și de a indica tipul lor este atât de frecventă încât însăși [biblioteca standard include o definiție pe care o putem utiliza!][IpAddr]<!-- ignore --> Să ne uităm cum biblioteca standard definește `IpAddr`: aceasta conține aceeași enumerare și variante pe care noi le-am definit și utilizat, însă datele adresei sunt incluse în variante sub forma a două structuri diferite, definite în mod distinct pentru fiecare variantă:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Acest cod exemplifică faptul că într-o variantă de enumerare se pot integra variate tipuri de date, fie ele string-uri, numere sau structuri. Se poate chiar insera și o altă enumerare. În plus, tipurile din biblioteca standard nu sunt de regulă mult mai complexe decât cele pe care le-ai putea elabora tu.

Deși biblioteca standard posedă o definiție pentru `IpAddr`, noi avem capacitatea de a crea și utiliza propria noastră definiție pentru `IpAddr`, fără a întâlni conflicte. Aceasta deoarece nu am importat definiția provenită din biblioteca standard în domeniul nostru de vizibilitate. Profundăm discuția despre cum se importă tipuri în domeniul de vizibilitate în Capitolul 7.

Să analizăm un alt exemplu de enumerare, ce poate fi găsit în Listarea 6-2. Aici observăm o diversitate mare de tipuri incluse în variantele enumerării.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

<span class="caption">Listarea 6-2: Enum-ul `Message` are variante ce stochează cantități și tipuri diferite de valori</span>

Acest enum conține patru variante de tipuri diferite:

* `Quit` nu are asociat niciun fel de date.
* `Move` conține câmpuri numite, similar unei structuri.
* `Write` include un singur `String`.
* `ChangeColor` include trei valori de tip `i32`.

Crearea unui enum cu variante cum sunt cele din Listarea 6-2 se aseamănă cu definirea diferitelor tipuri de structuri, dar cu câteva diferențe: enum nu folosește cuvântul cheie `struct` și toate variantele sunt grupate sub tipul `Message`. Structurile enumerate mai jos ar putea reține aceleași date ca și variantele enum-ului menționate anterior:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

Dacă am utiliza structuri diferite, fiecare având propriul său tip, ar fi mai dificil să definim o funcție care poate accepta oricare dintre aceste tipuri de mesaje. În schimb, prin intermediul enum-ului `Message`, definit în Listarea 6-2, putem realiza acest lucru mult mai simplu deoarece `Message` reprezintă un singur tip.

Enum-urile și structurile mai au o similitudine: așa cum definim metode pe structuri utilizând `impl`, tot așa putem defini metode și pe enum-uri. Să luăm de exemplu metoda numită `call`, pe care am putea să o definim pentru enum-ul nostru `Message`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

În corpul metodei utilizăm `self` pentru a obține valoarea pe care am invocat metoda. În acest exemplu, am creat variabila `m` cu valoarea `Message::Write(String::from("hello"))`, iar aceasta va reprezenta `self` în corpul metodei `call` când se execută instrucțiunea `m.call()`.

Să examinăm și un alt enum foarte comun și util din biblioteca standard, numit `Option`.

### Enum-ul `Option` și superioritatea sa față de valorile null

Această secțiune urmărește un studiu de caz al `Option`, care reprezintă o altă enumerare definită înlăuntrul librăriei standard. Tipul `Option` reflectă scenariul întâlnit frecvent în care o valoare poate exista sau poate absenta.

De exemplu, dacă ai solicita primul element dintr-o listă ce conține elemente, ai obține o valoare. În schimb, dacă ai solicita primul element dintr-o listă goală, nu ai obține nimic. Transpunerea acestui concept în sistemul de tipuri permite compilatorului să verifice dacă ai tratat toate cazurile pe care ar trebui să le gestionezi; această funcționalitate poate contribui la prevenirea apariției unor erori extrem de comune în alte limbaje de programare.

Designul unui limbaj de programare este adesea gândit în contextul funcțiilor pe care le adaugi, dar funcțiile pe care le ignori sunt la fel de importante. Rust nu dispune de opțiunea null pe care multe alte limbaje o oferă. *Null* este o valoare care indică lipsa unui răspuns sau a unei valori valide. În limbajele care permit valori null, variabilele pot exista mereu într-una din cele două stări: null sau non-null.

În discursul său din 2009, intitulat "Referințe Nule: Eroarea de un miliard de dolari", Tony Hoare, părintele conceptului de null, exprima astfel:

> Aceasta este greșeala mea de un miliard de dolari. Atunci, în acel moment,
> eram la etapa de proiectare a primului sistem complet de tipuri pentru
> referințe în cadrul unui limbaj orientat pe obiecte. Misiunea pe care mi-o
> asumasem era să garantez că orice utilizare a referințelor va fi absolut
> sigură, cu verificări efectuate automat de către compilator. Cu toate
> acestea, tentația de a introduce o referință nulă, deoarece era atât de
> simplu de realizat, a fost irezistibilă. Aceasta a condus la un număr
> inestimabil de erori, vulnerabilități și sisteme care s-au prăbușit,
> provocând, probabil, prejudicii de un miliard de dolari în ultimele patru
> decenii.

Problema cu valorile nule constă în faptul că, atunci când încerci să folosești o valoare nulă ca și cum ar fi una nenulă, inevitabil vei întâlni o eroare. Această caracteristică de null sau non-null este atât de răspândită, încât este extrem de ușor să comiți acest tip de eroare.

Totuși, conceptul pe care valoarea null îl reprezintă este în continuare unul esențial: un null reprezintă o valoare care este, temporar, nevalidă sau menținută absentă, dintr-un anumit motiv.

Problema nu este cu conceptul în sine, ci mai degrabă cu modul său specific de implementare. Prin urmare, Rust nu conține valori null, dar dispune de o enumerare care poate exemplifica ideea de prezență sau absență a unei valori. Această enumerare este `Option<T>`, definită în [biblioteca standard][option] astfel:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

Enumerarea `Option<T>` este atât de folositoare încât este inclusă chiar și în preludiu, nu necesită a fi adusă explicit în domeniul de vizibilitate. Variantele sale sunt de asemenea incluse în preludiu: poți folosi `Some` și `None` direct, fără a avea nevoie de prefixul `Option::`. Enumerarea `Option<T>` rămâne doar o enumerare obișnuită, iar `Some(T)` și `None` sunt în continuare variantele sale, de tip `Option<T>`.

Sintaxa `<T>` este o caracteristică a Rust pe care nu am discutat-o încă. Este un parametru de tip generic, despre care vom vorbi în amănunt în Capitolul 10. Deocamdată, trebuie doar să știi că `<T>` semnifică faptul că varianta `Some` a enumerării `Option` poate conține o dată de orice tip, iar fiecare tip concret care este utilizat în locul lui `T` transformă tipul global `Option<T>` într-un tip diferit. Iată câteva exemple de utilizare a valorilor `Option` pentru a reține tipuri de numere și de șiruri de caractere:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

Tipul variabilei `some_number` este `Option<i32>`. În contrast, `some_char` este de tip `Option<char>`, deci avem de-a face cu un tip diferit. Rust este capabil să deducă aceste tipuri fără nicio intervenție din partea noastră datorită faptului că am specificat deja o valoare în varianta `Some`. În cazul lui `absent_number`, lucrurile sunt un pic diferite. Aici, Rust ne solicită să adnotăm explicit tipul `Option` principal: compilatorul nu poate deduce tipul pe care variantul `Some` îl va deține doar uitându-se la o valoare `None`. De aceea, în acest context, îi comunicăm noi explicit lui Rust că intenționăm ca `absent_number` să fie de tip `Option<i32>`.

Când lucrăm cu o valoare `Some`, știm că avem o valoare prezentă și că aceasta se află în interiorul `Some`. În contrast, când avem o valoare `None`, semnifică într-un fel același lucru ca și null: nu dispunem de o valoare validă. Dar de ce ar fi mai avantajos să folosim `Option<T>` în locul lui null?

Pe scurt, avantajul de a lucra cu `Option<T>` în loc de null ține de faptul că `Option<T>` și `T` (unde `T` poate fi orice tip) reprezintă tipuri diferite. De-oarece avem de a face cu tipuri diferite, compilatorul nu ne va permite să utilizăm o valoare `Option<T>` ca și cum am fi absolut siguri că ea conține o valoare validă. De exemplu, codul următor nu va compila, deoarece încearcă să adauge un `i8` la un `Option<i8>`:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

Dacă rulăm acest cod, vom primi un mesaj de eroare similar cu acesta:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

Intens! Acesta ne spune că Rust nu poate efectua operația de adunare între un `i8` și un `Option<i8>`, deoarece acestea sunt de tipuri diferite. În Rust, atunci când lucrăm cu o valoare de tip `i8`, compilatorul se asigură că valoarea respectivă este întotdeauna validă. Prin urmare, putem utiliza valoarea respectivă fără a fi nevoie să verificăm în prealabil dacă aceasta este null. Problemele apar doar atunci când avem de-a face cu un `Option<i8>` (sau orice alt tip de valoare). În acest caz, este posibil să nu avem o valoare validă, iar compilatorul se va asigura că vom trata acest caz corect înainte de a folosi valoarea respectivă.

Altfel spus, pentru a putea efectua operațiuni cu `Option<T>`, va trebui mai întâi să-l convertim la `T`. Această regulă ne ajută să depistăm una dintre problemele cele mai comune legate de valoarea null: presupunerea că o anumită valoare nu poate fi null, când de fapt aceasta poate fi.

Faptul că putem elimina riscul de a presupune incorect că o valoare nu este null ne conferă mai multă încredere în corectitudinea codului nostru. De asemenea, pentru a putea avea o valoare care să poată fi eventual null, trebuie să optăm explicit pentru tipul de valoare `Option<T>`. Ulterior, la utilizarea acestei valori, vom fi nevoiți să tratăm explicit cazul în care valoarea este null. Oriunde avem o valoare al cărei tip nu este `Option<T>`, putem presupune în siguranță că valoarea respectivă nu este null. Aceasta este o decizie deliberată de proiectare a limbajului Rust, menită să limiteze prevalența valorilor null și să crească siguranța codului scris în Rust.

Acum să vedem cum putem extrage valoarea `T` dintr-un `Option<T>` de tip `Some`, astfel încât să o putem folosi. Enumerația `Option<T>` dispune de o serie de metode utile în diverse contexte, pe care le poți vedea în [documentația acestora][docs]<!-- ignore -->. Cunoașterea metodelor variabilei `Option<T>` va fi extrem de importantă pe parcursul călătoriei tale cu Rust.

În general, pentru a folosi o valoare `Option<T>`, va trebui să scrii un cod care să poată trata fiecare variantă. Vei avea nevoie de un cod ce se va executa doar atunci când ai o valoare `Some(T)`, iar acesta va putea folosi valoarea `T` internă. În același timp, vei avea nevoie de un alt cod ce se va executa doar dacă ai o valoare `None`, iar acesta nu va avea acces la o valoare `T`. Expresia `match` este un construct al fluxului de control care, atunci când este folosită cu enumerări, ne permite să rulăm cod diferit în funcție de tipul variantelor, având posibilitatea de a utiliza datele din interiorul valorii potrivite.

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html
