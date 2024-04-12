# Un proiect de I/O: Construirea unei aplicații de linie de comandă

În acest capitol vom recapitula diversele abilități învățate până acum și vom explora noi funcționalități ale bibliotecii standard. Construim o aplicație de linie de comandă pentru a interacționa cu fișiere și intrarea/ieșirea de linie de comandă, astfel exersăm conceptele Rust cu care sunteți deja familiarizați.

Datorită vitezei, siguranței, output-ului binar unic și suportului multi-platformă, Rust este un limbaj excelent pentru crearea de aplicații de linie de comandă. În proiectul nostru vom realiza o versiune proprie a utilitarului clasic de căutare în linia de comandă `grep` (căutare **g**lobală pentru o **r**egulară **e**xpresie și **p**rintare). În cel mai elementar scenariu, `grep` scanează un string specific în cadrul unui fișier dat. Astfel, `grep` utilizează ca argumente o cale de fișier și un string, citește fișierul, identifică liniile care includ string-ul specificat și le afișează.

De-a lungul capitolului, vom detalia modul în care aplicația noastră de linie de comandă poate folosi caracteristicile terminalului comune altor astfel de aplicații. Vom citi valoarea unei variabile de mediu pentru a permite utilizatorului să modifice comportamentul instrumentului nostru. Mai mult, vom afișa mesajele de eroare în fluxul consolei standard de eroare (`stderr`) decât în cel de ieșire standard (`stdout`), permițând astfel utilizatorului să redirecționeze ieșirile reușite către un fișier, dar să păstreze mesajele de eroare vizibile pe ecran.

Un membru al comunității Rust, Andrew Gallant, a creat o versiune avansată și rapidă de `grep`, cunoscută sub numele de `ripgrep`. Versiunea noastră va fi simplificată, dar capitolul va oferi informațiile de bază necesare pentru a înțelege un proiect concret precum `ripgrep`.

Proiectul `grep` va sintetiza numeroase concepte stăpânite până în momentul de față:

* Organizarea codului (așa cum am văzut în [Capitolul 7][ch7]<!-- ignore -->)
* Folosirea vectorilor și string-urilor (discutat în [Capitolul 8][ch8]<!-- ignore -->)
* Gestionarea erorilor (abordată în [Capitolul 9][ch9]<!-- ignore -->)
* Implementarea trăsăturilor și gestionarea duratelor de viață (prezentate în [Capitolul 10][ch10]<!-- ignore -->)
* Crearea de teste (explicate în [Capitolul 11][ch11]<!-- ignore -->)

Vom face, de asemenea, o trecere rapidă prin conceptul de închideri, iteratori și obiecte-trăsătură, pe care le vom aprofunda în Capitolele [13][ch13]<!-- ignore --> și [17][ch17]<!-- ignore -->.

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch17]: ch17-00-oop.html
