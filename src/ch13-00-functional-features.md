# Elemente de programare funcțională în Rust: închideri și iteratori

Designul Rust a fost inspirat de numeroase limbaje și tehnici existente, o influență semnificativă fiind *programarea funcțională*. Stilul funcțional de programare implică frecvent utilizarea funcțiilor ca valori: acestea pot fi transmise ca argumente, returnate de alte funcții sau atribuite variabilelor pentru execuție ulterioară.

În acest capitol, nu vom intra în dezbaterea despre ce definește programarea funcțională, ci vom explora unele caracteristici ale limbajului Rust care sunt tipice pentru limbajele considerate funcționale.

În detaliu, vom aborda:

* *Închiderile* (closures), structuri similare funcțiilor pe care le poți salva într-o variabilă
* *Iteratorii*, o abordare pentru procesarea unei serii de elemente
* Modul în care închiderile și iteratorii pot fi folosiți pentru a îmbunătăți proiectul de I/O din Capitolul 12
* Performanța închiderilor și a iteratorilor (Spoiler: sunt mai performanți decât ai crede!)

Am discutat anterior despre alte aspecte ale lui Rust, cum ar fi corelarea șabloanelor și enumerațiile, care sunt de asemenea marcate de influența stilului funcțional. Deoarece stăpânirea închiderilor și a iteratorilor constituie un element cheie în redactarea codului Rust idiomatic și eficient, le vom aloca un întreg capitol.
