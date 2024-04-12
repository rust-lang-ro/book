# Colecțiile comune în Rust

Biblioteca standard Rust ne pune la dispoziție o serie de structuri de date foarte utile, denumite generic *colecții*. Spre deosebire de celelalte tipuri de date care reprezintă o singură valoare, colecțiile pot stoca multiple valori. În plus, nu ca tipurile native array și tuplă, datele la care se referă aceste colecții sunt stocate în heap, deci volumul acestora nu trebuie să fie cunoscut în momentul compilării și poate varia pe parcursul execuției programului. Fiecare tip de colecție vine cu propriile sale capabilități și costuri asociate, așadar selectarea celei potrivite pentru situația curentă este o abilitate care se dezvoltă în timp. În acest capitol ne vom concentra pe trei colecții foarte utilizate în programele Rust:

* Vectorul ne permite stocarea unui număr variabil de valori alături.
* String-ul reprezintă o colecție de caractere. Deși am menționat anterior tipul `String`, în acest capitol îl vom aborda în profunzime.
* O hartă hash ne dă posibilitatea de a asocia o valoare cu o cheie specifică, fiind un caz particular de implementare a structurii de date denumite *map*.

Consultează [documentația][collections] pentru a afla despre celelalte tipuri de colecții disponibile în biblioteca standard.

Vom discuta despre cum putem crea și actualiza vectori, string-uri și hărți hash, precum și despre ce caracteristici unice are fiecare.

[collections]: ../std/collections/index.html
