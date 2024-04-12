# Pattern-urile și potrivirea

*Pattern-urile* (a nu se confunda cu pattern-urile de design) reprezintă o sintaxă specială în Rust dedicată potrivirii cu structura tipurilor, fie ele complexe sau simple. Folosirea pattern-urilor în conjuncție cu expresiile `match` și alte construcții îți oferă mai mult control asupra fluxului de execuție al programului. Un pattern este alcătuit dintr-o combinație a următoarelor elemente:

* Literali
* Array-uri, enumerări, struct-uri sau tuple-uri destructurate
* Variabile
* Wildcard-uri
* Placeholder-uri

Printre exemple de pattern-uri se numără `x`, `(a, 3)` și `Some(Color::Red)`. În contextele unde pattern-urile sunt aplicabile, aceste componente descriu structura datelor. Programul nostru corelează apoi valorile împotriva pattern-urilor pentru a determina dacă acestea corespund cu structura de date necesară pentru a executa o porțiune de cod.

Pentru a folosi un pattern, îl comparăm cu o valoare. Dacă pattern-ul se potrivește cu valoarea, folosim acele părți ale valorii în codul nostru. Amintește-ți de expresiile `match` din Capitolul 6 care au utilizat pattern-uri, precum exemplul cu mașina de sortat monede. Dacă valoarea se încadrează în structura pattern-ului, putem folosi piesele cu denumiri specifice. În caz contrar, secțiunea de cod asociată cu acel pattern nu va fi executată.

Acest capitol este un ghid complet despre tot ce este legat de pattern-uri. Vom explora contextele adecvate pentru utilizarea pattern-urilor, diferența între pattern-urile contestabile și incontestabile, precum și diferitele tipuri de sintaxă pattern pe care le vei întâlni. Până la finalul capitolului, vei învăța cum să folosești pattern-urile pentru exprimarea clară a multor concepte.
