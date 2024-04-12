## Transferul datelor între fire de execuție cu pasare de mesaje

*Pasarea de mesaje* (message passing) este o metodă din ce în ce mai adoptată pentru a garanta concurența sigură. Această abordare implică comunicarea între fire de execuție sau actori prin intermediul mesajelor care includ date. Un slogan din [documentația limbajului Go](https://golang.org/doc/effective_go.html#concurrency) rezumă această filosofie: "Nu comunicați prin partajarea memoriei; în schimb, partajați memoria comunicând."

Rust îmbrățișează acest concept prin *canale* (channels), facilități oferite de biblioteca standard pentru implementarea concurenței prin trimiterea de mesaje. Un canal este un concept comun în programare pentru a transmite date de la un fir de execuție la altul.

Putem asemăna un canal în programare cu un torent de apă curgătoare, de exemplu un râu, în care obiectele plasate în apă se deplasează într-o singură direcție. Dacă introduci o rațușcă de cauciuc într-un râu, ea va pluti până la destinația din aval.

Un canal este format dintr-un emițător și un receptor. Emițătorul, situat "amonte", este locul de unde "lansezi rața de cauciuc", iar receptorul sau "avalul" este punctul unde aceasta ajunge. În practică, o parte din cod va invoca metode pe emițător pentru a trimite date, iar o altă parte va prelucra mesajele care ajung la receptor. Dacă una dintre aceste componente este distrusă, canalul este considerat *închis*.

Vom exemplifica această noțiune printr-un program care include un fir de execuție ce generează valori și le trimite printr-un canal și un alt fir care le recepționează și le afișează. Vom demonstra trimiterea de valori simple prin canal pentru a evidenția funcționalitatea. După ce te familiarizezi cu această abordare, poți extinde utilizarea canalelor pentru orice fire de execuție care necesită comunicare, de exemplu într-un sistem de chat sau într-un sistem de calcul distribuit unde diferite fire procesează părți dintr-o sarcină și le trimit spre un fir central care le integrează.

În prima etapă, în Listarea 16-6, intenționăm să creăm un canal, dar nu vom interacționa cu acesta. Este important de notat că acest cod nu va compila încă, pentru că Rust nu poate identifica tipul valorilor pe care dorim să le transmitem prin canal.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

<span class="caption">Listarea 16-6: Crearea unui canal și împărțirea acestuia în două părți, `tx` și `rx`</span>

Inițiem un canal nou cu ajutorul funcției `mpsc::channel`; unde `mpsc` sugerează ideea de *producător multiplu, consumator unic* (multiple producer, single consumer). Pe scurt, implementarea canalelor în biblioteca standard Rust permite existența mai multor puncte de *transmitere* care produc valori, dar numai un singur punct de *recepție* pentru a le consuma. Să ne imaginăm mai multe pârâiașe care se unesc într-un singur fluviu: orice este trimis prin aceste pârâiașe va ajunge inevitabil în fluviul mare. Pentru început, vom folosi un singur producător, urmând să adăugăm alți producători odată ce acest exemplu va fi funcțional.

Funcția `mpsc::channel` returnează o tuplă, ale cărei elemente sunt: primul - punctul de *transmitere*--transmițătorul--și al doilea - punctul de *recepție*--receptorul. Abrevierile `tx` pentru transmițător și `rx` pentru receptor sunt uzate frecvent în diverse domenii, astfel le vom folosi și noi pentru a denumi variabilele, marcând fiecare capăt al canalului. Aplicăm o declarație `let` cu un șablon care destramă tupla; mai multe despre șabloane în declarațiile `let` și destructurarea acestora vom explora în Capitolul 18. Pe moment, este suficient să înțelegem că utilizarea declarației `let` în acest mod este eficientă pentru a desface componentele tuplei oferite de `mpsc::channel`.

Permutăm punctul de transmitere într-un fir separat și îi permitem să expedieze un string către firul secundar, facilitând astfel comunicarea dintre noul fir de execuție și firul principal, conform exemplului prezentat în Listarea 16-7. Este ca și cum am pune o rață de cauciuc în apa râului în amonte sau am trimite un mesaj text de la un fir la altul.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

<span class="caption">Listarea 16-7: Mutarea `tx` într-un fir de execuție creat și trimiterea mesajului “hi”</span>

Din nou, folosim `thread::spawn` pentru a iniția un nou fir de execuție și apoi `move` pentru a permuta `tx` în închidere astfel încât noul firul de execuție să dețină `tx`. Firul de execuție creat are nevoie să dețină emitentul pentru a putea transmite mesaje prin canal. Emitentul dispune de metoda `send`, ce acceptă valoarea pe care dorim să o expediem. Metoda `send` returnează o structură de tip `Result<T, E>`, astfel că, dacă receptorul a fost deja eliminat din memorie și nu mai există un destinatar pentru valoare, operațiunea de trimitere va resulta într-o eroare. În acest exemplu, folosim `unwrap` pentru a induce panică în caz de eroare. Totuși, într-o aplicație reală, am aborda această situație corespunzător: vezi Capitolul 9 pentru a reexamina strategiile adecvate de gestionare a erorilor.

În Listarea 16-8, vom prelua valoarea de la receptor în firul principal. Este ca și cum am scoate rața de cauciuc din apă în punctul final al râului sau cum am primi un mesaj de chat.

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

<span class="caption">Listarea 16-8: Primirea valorii “hi” în firul principal și afișarea acesteia</span>

Receptorul are două metode utile: `recv` și `try_recv`. Utilizăm `recv`, abreviere pentru *receive*, care va opri temporar execuția firului principal și va aștepta până când o valoare va fi expediată prin canal. După expedierea unei valori, `recv` o va returna ca un `Result<T, E>`. Când emitentul se deconectează, `recv` va semnala prin intermediul unei erori că nu vor mai sosi valori suplimentare.

Metoda `try_recv` nu înterupe execuția, dar returnează imediat un `Result<T, E>`: o valoare `Ok` care conține un mesaj dacă este disponibil unul sau o valoare `Err` dacă nu există mesaje în acel moment. Utilizarea `try_recv` este eficientă dacă firul de execuție are alte activități de îndeplinit în timp ce așteaptă mesaje; am putea implementa o buclă care apelează `try_recv` periodic și procesează mesajul, dacă este disponibil unul, sau continuă cu alte sarcini pentru o perioadă, înainte de a verifica din nou.

Am ales `recv` pentru acest exemplu datorită simplității; nu avem nicio altă sarcină pentru firul principal decât așteptarea mesajelor, așa că oprirea temporară a execuției firului principal este justificată.

Când executăm codul din Listarea 16-8, vom observa valoarea afișată de către firul principal:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

Perfect!

### Canale și transferul de posesiune

Regulile de posesiune sunt vitale în trimiterea mesajelor deoarece ajută la scrierea de cod sigur și concurent. Prevenirea erorilor în programarea concurentă este un beneficiu al gândirii în termeni de posesiune de-a lungul programelor tale în Rust. Să realizăm un experiment pentru a demonstra cum canalele și posesiunea colaborează pentru a evita problemele: vom încerca să utilizăm o valoare `val` în firul de execuție derivat *după* ce am trimis-o prin canal. Încearcă să compilezi codul din Listarea 16-9 pentru a vedea motivul pentru care acest cod nu este acceptat:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

<span class="caption">Listarea 16-9: Tentativa de utilizare a `val` după ce a fost trimisă prin canal</span>

Aici, încercăm să afișăm `val` după ce am trimis-o prin canal cu ajutorul `tx.send`. A permite acest lucru ar constitui o idee rea: odată ce valoarea a fost trimisă unui alt fir de execuție, acel fir ar putea să modifice sau să elibereze valoarea înainte ca noi să încercăm utilizarea acesteia. Modificările aduse de celălalt fir ar putea duce la erori sau rezultate neașteptate datorate datelor inconsistente sau inexistente. Iarăși, Rust ne returnează o eroare dacă încercăm să compilăm codul din Listarea 16-9:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Greșeala noastră de concurență a produs o eroare la timpul compilării. Funcția `send` preia posesiunea parametrului său, iar atunci când valoarea este permutată, receptorul devine noul posesor al acesteia. Aceasta ne previne să folosim din greșeală valoarea încă o dată după trimiterea ei; sistemul de posesiune se asigură că totul este corect.

### Trimiterea de multiple valori și observarea așteptării receptorului

Codul din Listarea 16-8 a fost compilat și a rulat, dar nu a demonstrat clar că două fire separate de execuție comunica între ele prin canal. În Listarea 16-10 am efectuat niște modificări care vor demonstra cum codul din Listarea 16-8 execută operațiuni în mod concurent: firul de execuție derivat va trimite acum mai multe mesaje și va însera o pauză de o secundă între fiecare.

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

<span class="caption">Listarea 16-10: Trimite mai multe mesaje și pauza între ele</span>

În acest exemplu, firul derivat are un vector de string-uri pe care vrea să le trimită către firul principal. Iterăm pe acest vector, trimițând fiecare string în parte și folosind o pauză, invocând `thread::sleep` cu o valoare `Duration` de o secundă pe iterație.

În firul principal, nu mai apelăm funcția `recv` în mod direct; acum tratăm `rx` ca pe un iterator. Imprimăm fiecare valoare primită și când canalul se închide, iterația se oprește.

Rulând codul din Listarea 16-10, ar trebui să vezi următoarele mesaje afișate cu o pauză de 1 secundă între fiecare:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

Faptul că bucla `for` din firul principal nu conține niciun cod care să producă întârzieri ne spune că firul principal așteaptă să primească valori de la firul derivat.

### Crearea mai multor producători prin clonarea transmițătorului

Anterior, am menționat că `mpsc` este acronimul pentru *producător multiplu, consumator unic*. Să utilizăm `mpsc` și să extindem codul din Listarea 16-10 pentru a crea multiple fire de execuție care trimit valori către același receptor. Putem realiza acest lucru prin clonarea transmițătorului, așa cum este ilustrat în Listarea 16-11:

<span class="filename">Numele fișierului: src/main.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

<span class="caption">Listarea 16-11: Transmiterea mai multor mesaje de la mai mulți producători</span>

De această dată, înainte de a genera primul fir, apelăm metoda `clone` pe transmițător. Aceasta ne va oferi un nou transmițător pe care îl putem folosi în primul fir derivat. Transmițătorul original îl oferim unui al doilea fir derivat. Astfel obținem două fire de execuție, fiecare transmițând mesaje diferite către același receptor.

Atunci când rulezi codul, afișajul ar trebui să arate într-un fel similar cu acesta:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Poți observa valorile într-o altă ordine, depinde de sistemul tău. Acest lucru face din concurență un subiect atât de interesant, cât și de dificil de abordat. Dacă joci cu valorile `thread::sleep` în diferitele fire, vei face ca fiecare execuție să fie mai puțin deterministă și să producă un afișaj diferit de fiecare dată.

După ce am analizat modul în care funcționează canalele, să trecem la o altă abordare a concurenței.
