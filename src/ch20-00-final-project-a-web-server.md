# Proiect final: Dezvoltarea unui server web multi-thread

Drumul parcurs a fost lung, dar în final, am ajuns la ultimul capitol al cărții. În acest capitol, ne vom uni forțele pentru a construi încă un proiect, prin care să demonstrăm câteva din conceptele pe care le-am abordat în capitolele anterioare și să recapitulăm lecțiile învățate.

Pentru proiectul nostru de final, vom dezvolta un server web care va afișa un simplu “hello” și va arăta precum Figura 20-1 în browser-ul web.

![hello from rust](img/trpl20-01.png)

<span class="caption">Figura 20-1: Proiectul nostru final comun</span>

Iată pașii pe care îi vom urma pentru a construi serverul web:

1. O scurtă introducere în TCP și HTTP. 
2. Interceptarea conexiunilor TCP printr-un socket. 
3. Analiza unui set redus de cereri HTTP. 
4. Generarea unui răspuns HTTP adecvat. 
5. Creșterea capacității de procesare a serverului nostru printr-un pool de thread-uri.

Înainte de a începe, este important să punctăm un aspect: calea pe care o vom urma în acest proiect nu reprezintă cea mai eficace metodă de a construi un server web cu Rust. Membri ai comunității au publicat o varietate de crate-uri gata pentru producție, disponibile pe [crates.io](https://crates.io/), care prezintă implementări mai avansate pentru servere web și thread pools decât varianta pe care o vom dezvolta noi. Totuși, intenția noastră în acest capitol este de a facilita procesul de învățare, și nu de a urma soluția cea mai simplă. Fiind un limbaj de programare de sistem, Rust ne oferă libertatea de a alege nivelul de abstractizare la care dorim să lucrăm, ceea ce ne permite să ne aplecăm către detalii de nivel mai jos decât ne-ar permite alte limbaje. Așadar, vom construi de la zero serverul HTTP de bază și pool-ul de thread-uri, pentru ca tu să asimilezi conceptele fundamentale și tehnicile din spatele crate-urilor pe care le poți eventual utiliza în viitor.