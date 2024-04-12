## Comparând performanța buclelor și a iteratorilor

Pentru a decide folosirea buclelor sau a iteratorilor, trebuie să determinăm care implementare este mai eficientă: versiunea funcției `search` ce utilizează o buclă `for` explicită sau versiunea ce se bazează pe iteratori.

Am efectuat un benchmark, încărcând întregul text al cărții *The Adventures of Sherlock Holmes* de Sir Arthur Conan Doyle într-un `String` și căutând cuvântul *the* în conținut. Iată rezultatele benchmark-ului pentru versiunea `search` folosind ciclul `for` și versiunea cu iteratori:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

Versiunea cu utilizarea iteratorilor a fost ușor mai rapidă! Nu vom detalia aici codul folosit în benchmark, deoarece scopul nu este să demonstrăm că cele două versiuni sunt identice ca funcționalitate, ci să înțelegem cum se compară la general cele două implementări ca performanță.

Pentru un benchmark mai amplu, ar fi indicat să folosim texte de dimensiuni variate ca `contents`, cu cuvinte diferite și de lungimi variate ca `query`, împreună cu alte multiple variații. Ideea principală este următoarea: deși reprezintă o abstracție de nivel înalt, iteratorii din Rust sunt compilați în cod foarte similar cu cel scris manual la un nivel jos. Aceștia sunt exemple ale abstracțiilor *cu cost zero* în Rust, prin care înțelegem că utilizarea abstracției nu adaugă supra-costuri la timpul de rulare. Conceptul dat este asemănător cu modul în care Bjarne Stroustrup, inițiatorul limbajului C++, definește noțiunea *fără surplus de costuri* în lucrarea sa "Foundations of C++" (2012):

> În general, implementările C++ respectă principiul zero supra-cost: dacă nu
> folosești ceva, nu plătești pentru acesta. Și mai mult: ceea ce folosești nu
> ai cum să codezi manual într-un mod mai eficient.

Ca un alt exemplu, porțiunea de cod de mai jos provine dintr-un decodor audio. Algoritmul de decodare folosește operațiunea matematică de predicție liniară pentru a estima valorile viitoare bazându-se pe o funcție liniară a mostrelor anterioare. Acest cod utilizează o serie de iteratori pentru a efectua calcule asupra a trei variabile din domeniul de vizibilitatea: o secțiune `buffer` de date, un array cu 12 `coefficients` și o valoare care indică cu cât se deplasează datele în `qlp_shift`. Variabilele sunt declarate în acest exemplu, însă nu li s-au atribuit valori; chiar dacă acest cod nu are mult sens fără contextul său, reprezintă totuși un exemplu concret și realist al modului în care Rust transformă idei complexe în cod de nivel jos.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

Pentru a calcula valoarea `prediction`, codul iterează prin fiecare dintre cele 12 valori din `coefficients` și aplică metoda `zip` pentru a forma perechi între valorile coeficienților și cele 12 mostre anterioare din `buffer`. Apoi, pentru fiecare pereche, se înmulțesc valorile între ele, se sumează toate rezultatele și suma rezultată este deplasată `qlp_shift` biți spre dreapta.

În aplicațiile precum decodoarele audio, calculele pun adesea întâietate pe performanță. În exemplul nostru, construim un iterator, utilizăm două adaptoare și apoi consumăm valoarea. În ce cod assembly va transforma codul Rust? Până la acest moment, se compilează în assembly-ul pe care l-ai scrie și manual. Nu există o buclă care să corespundă iterației peste `coefficients`: Rust recunoaște cele 12 iterații, așa că efectuează o „desfășurare” (unroll) a buclei. *Desfășurarea* este o optimizare care elimină supra-costul codului de control al buclei și generează, în schimb, cod repetitiv pentru fiecare iterație.

Toți coeficienții sunt stocați în registre, ceea ce facilitează un acces rapid la valori. La executarea codului nu sunt efectuate verificări de limită pentru array. Toate optimizările aplicate de Rust fac codul extrem de eficient. Având această cunoaștere, poți folosi iteratori și închideri fără nicio reținere! Ele fac codul să pară de un nivel înalt și nu aduc un cost adițional de performanță la execuție.

## Sumar

Închiderile și iteratorii sunt caracteristici ale Rust inspirate din conceptele limbajelor de programare funcționale. Ele contribuie la abilitatea limbajului Rust de a exprima idei complexe într-un mod clar, menținând totodată performanțe specifice codului de nivel jos. Implementările închiderilor și a iteratorilor sunt concepute în așa fel încât să nu afecteze performanța la runtime. Aceasta este parte a scopului Rust de a furniza abstracții fără un extra-cost.

Acum că am îmbunătățit expresivitatea proiectului nostru I/O, să examinăm unele funcționalități suplimentare ale `cargo` care ne vor ajuta să distribuim proiectul cu întreaga lume.
