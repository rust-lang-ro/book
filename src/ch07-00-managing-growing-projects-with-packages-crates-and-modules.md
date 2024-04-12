# Administrarea proiectelor în expansiune cu ajutorul pachetelor, crate-urilor și modulelor

Compunerea programelor de amploare invocă o importanță crescută asupra organizării codului tău. Grupând funcționalitățile corelate și separând codul cu particularități distincte, vei desluși unde să identifici codul care susține o anumită caracteristică și unde să intervii pentru a modifică cum acea caracteristică funcționează.

Până în prezent, programele pe care le-am creat au fost cuprinse într-un singur modul, într-un singur fișier. Odată cu expansiunea unui proiect, este esențial să organizezi codul împărțindu-l în diverse module și ulterior într-o multitudine de fișiere. Un pachet poate cuprinde multiple crate-uri binare și eventual un crate de bibliotecă. Odată cu dezvoltarea unui pachet, poți desprinde diverse componente în crate-uri distincte care vor deveni dependențe externe. Acest capitol îți prezintă toate aceste procedee. Pentru proiectele de o amploare considerabilă, compuse dintr-un ansamblu de pachete interconectate care evoluează împreună, Cargo oferă *spațiile de lucru (workspaces)*, pe care le vom descrie în secțiunea [„Spații de lucru Cargo”][workspaces] din Capitolul 14.

De asemenea, vom aborda subiectul încapsulării detaliilor de implementare. Acesta este un instrument esențial care îți permite să reutilizezi codul la un nivel superior. Odată ce ai implementat o operațiune, alt cod poate interacționa cu implementarea ta prin intermediul interfeței publice, fără a trebui să înțeleagă complexitatea acesteia. Modul în care structurezi codul stabilește care părți sunt accesibile altor bucăți de cod (publice) și care părți sunt detalii private ale implementării, pe care ai dreptul de a le modifica. Aceasta este o altă cale de a restrânge cantitatea de detalii pe care trebuie să o reții.

Un concept strâns legat este cel de domeniu de vizibilitate. Acesta se referă la contextul în care codul este scris, unde un set de nume sunt definite ca fiind „în domeniul de vizibilitate”. Atunci când se citește, scrie sau compilează cod, atât programatorii, cât și compilatoarele trebuie să știe dacă un anumit nume, într-un anumit context, se referă la o variabilă, o funcție, o structură, o enumerare, un modul, o constantă, sau alt element și ce semnificație are acel element. Ai libertatea de a crea domenii de vizibilitate și de a schimba ce nume sunt sau nu în vigoare. În același domeniu de vizibilitate, nu poți folosi același nume pentru două entități distincte, dar există unelte care te pot ajuta să rezolvi astfel de conflicte de nume.

Rust dispune de o varietate de funcții care îți facilitează gestionarea structurii codului, incluzând nivelul de accesibilitate a detaliilor, gradul de protecție al acestora, precum și gestionarea numelor din fiecare domeniu de vizibilitate din programele tale. Aceste facilități, adesea denumite în ansamblu ca *sistemul de module*, cuprind:

* **Pachetele:** O caracteristică proprie Cargo care îți permite să creezi, testezi și să distribui crate-uri.
* **Crate-urile:** Reprezintă un arbore de module ce produce o librărie sau un executabil.
* **Modulele** și **utilizarea:** Permite controlul asupra organizării, domeniului de vizibilitate și privației căilor.
* **Căile:** O modalitate de a denumi un element, precum o structură, o funcție sau un modul.

În cadrul acestui capitol, vom trece în revistă toate aceste facilități, vom discuta interacțiunea dintre ele și vom explica modul în care pot fi utilizate în gestionarea domeniului de vizibilitate. La final, vei avea o înțelegere profundă a sistemului de module și vei putea lucra cu domeniile de vizibilitate precum un expert!

[workspaces]: ch14-03-cargo-workspaces.html
