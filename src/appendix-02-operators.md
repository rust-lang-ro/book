## Anexa B: Operatori si simboluri

Această anexă conține un glosar al sintaxei Rust, inclusiv operatori și alte simboluri care apar singure sau în contextul căilor, genericelor, limitelor de trăsături, macrocomenzilor, atributelor, comentariilor, tuplelor și parantezelor.

### Operatori

Tabelul B-1 conține operatorii din Rust, un exemplu de cum ar apărea operatorul în context, o scurtă explicație și dacă acel operator este supraîncarcabil. Dacă un operator este supraîncarcabil, trăsătura relevantă de utilizat pentru a supraîncarca acel operator este listată.

<span class="caption">Tabelul B-1: Operatori</span>

| Operator | Exemplu | Explicație | Supraîncarcabil? |
|----------|---------|-------------|---------------|
| `!` | `ident!(...)`, `ident!{...}`, `ident![...]` | Expansiune de macrocomandă | |
| `!` | `!expr` | Complement logic sau pe biți | `Not` |
| `!=` | `expr != expr` | Comparare de non-egalitate | `PartialEq` |
| `%` | `expr % expr` | Rest aritmetic | `Rem` |
| `%=` | `var %= expr` | Rest aritmetic și atribuire | `RemAssign` |
| `&` | `&expr`, `&mut expr` | Referință | |
| `&` | `&type`, `&mut type`, `&'a type`, `&'a mut type` | Tip pointer împrumutat | |
| `&` | `expr & expr` | AND pe biți | `BitAnd` |
| `&=` | `var &= expr` | AND pe biți și atribuire | `BitAndAssign` |
| `&&` | `expr && expr` | AND logic scurtcircuitat (care abandonează evaluarea la prima expresie evaluată ca fals) | |
| `*` | `expr * expr` | Înmulțire aritmetică | `Mul` |
| `*=` | `var *= expr` | Înmulțire aritmetică și atribuire | `MulAssign` |
| `*` | `*expr` | Dereferențiere | `Deref` |
| `*` | `*const type`, `*mut type` | Pointer brut | |
| `+` | `trait + trait`, `'a + trait` | Constrângere de tip compus | |
| `+` | `expr + expr` | Adunare aritmetică | `Add` |
| `+=` | `var += expr` | Adunare aritmetică și atribuire | `AddAssign` |
| `,` | `expr, expr` | Separator de argumente și elemente | |
| `-` | `- expr` | Negare aritmetică| `Neg` |
| `-` | `expr - expr` | Scădere aritmetică | `Sub` |
| `-=` | `var -= expr` | Scădere aritmetică și atribuire | `SubAssign` |
| `->` | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Tipul de returnare pentru funcție și închidere | |
| `.` | `expr.ident` | Acces la membru | |
| `..` | `..`, `expr..`, `..expr`, `expr..expr` | Literal pentru rang exclus la dreapta | `PartialOrd` |
| `..=` | `..=expr`, `expr..=expr` | Literal pentru rang inclusiv la dreapta | `PartialOrd` |
| `..` | `..expr` | Sintaxă pentru actualizare literală structură | |
| `..` | `variant(x, ..)`, `struct_type { x, .. }` | Legare de pattern „Și restul” | |
| `...` | `expr...expr` | (Depreciat, în schimb folosiți `..=`) Într-un pattern: pattern pentru diapazon inclusiv| |
| `/` | `expr / expr` | Diviziune aritmetică| `Div` |
| `/=` | `var /= expr` | Diviziune aritmetică și atribuire | `DivAssign` |
| `:` | `pat: type`, `ident: type` | Constrângeri | |
| `:` | `ident: expr` | Inițializator de câmp de structură | |
| `:` | `'a: loop {...}` | Etichetă de buclă | |
| `;` | `expr;` | Terminare de instrucțiune și element | |
| `;` | `[...; len]` | Parte din sintaxă de array cu dimensiune fixă | |
| `<<` | `expr << expr` | Deplasare la stânga | `Shl` |
| `<<=` | `var <<= expr` | Deplasare la stânga și atribuire | `ShlAssign` |
| `<` | `expr < expr` | Comparare mai mică decât | `PartialOrd` |
| `<=` | `expr <= expr` | Comparare mai mic sau egal | `PartialOrd` |
| `=` | `var = expr`, `ident = type` | Atribuire/echivalență | |
| `==` | `expr == expr` | Comparare de egalitate | `PartialEq` |
| `=>` | `pat => expr` | Parte din sintaxa pentru ramura match | |
| `>` | `expr > expr` | Comparare mai mare decât | `PartialOrd` |
| `>=` | `expr >= expr` | Comparare mai mare sau egal | `PartialOrd` |
| `>>` | `expr >> expr` | Deplasare la dreapta | `Shr` |
| `>>=` | `var >>= expr` | Deplasare la dreapta și atribuire | `ShrAssign` |
| `@` | `ident @ pat` | Legare de pattern | |
| `^` | `expr ^ expr` | OR exclusiv pe biți | `BitXor` |
| `^=` | `var ^= expr` | OR exclusiv pe biți și atribuire | `BitXorAssign` |
| <code>&vert;</code> | <code>pat &vert; pat</code> | Alternative de pattern | |
| <code>&vert;</code> | <code>expr &vert; expr</code> | OR pe biți | `BitOr` |
| <code>&vert;=</code> | <code>var &vert;= expr</code> | OR pe biți și atribuire | `BitOrAssign` |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code> | OR logic care abandonează execuția la prima expresie evaluată ca adevărat | |
| `?` | `expr?` | Propagarea erorilor | |

### Simboluri non-operator

Următoarea listă conține toate simbolurile care nu funcționează ca operatori; adică, ele nu se comportă ca o funcție sau o apelare de metodă.

Tabelul B-2 arată simbolurile care apar singure și sunt valabile într-o varietate de locuri.

<span class="caption">Tabelul B-2: Sintaxa Stand-Alone</span>

| Simbol | Explicație |
|--------|-------------|
| `'ident` | Durata de viață numită sau eticheta loop-ului |
| `...u8`, `...i32`, `...f64`, `...usize`, etc. | Literal numeric de un anumit tip |
| `"..."` | Literal de șir de caractere |
| `r"..."`, `r#"..."#`, `r##"..."##`, etc. |  Literal de șir de caractere brut, caracterele de evadare nu sunt procesate |
| `b"..."` |  Literal de șir de byte-uri; construiește un array de byte-uri în loc de un șir |
| `br"..."`, `br#"..."#`, `br##"..."##`, etc. | Literal de șir de byte-uri brut, combinația dintre șirul brut de caractere și șirul de byte-uri |
| `'...'` | Literal de caracter |
| `b'...'` | Literal de byte ASCII |
| <code>&vert;...&vert; expr</code> | Închidere (Closure) |
| `!` | Tipul de fundal întotdeauna gol pentru funcțiile divergente |
| `_` |  Legare a modelului "Ignorat"; de asemenea, este utilizat pentru a face literalele întregi lizibile |

Tabelul B-3 arată simbolurile care apar în contextul unei cale prin modulul
ierarhia către un element.

<span class="caption">Tabelul B-3: Sintaxa legată de Path</span>

| Simbol | Explicație |
|--------|-------------|
| `ident::ident` | Calea Namespace-ului |
| `::path` | Cale relativă la rădăcina crate-ului (adică, o cale absolută explicită) |
| `self::path` | Cale relativă la modulul curent (adică, o cale relativă explicită) |
| `super::path` | Cale relativă la părintele modulului curent |
| `type::ident`, `<type as trait>::ident` |  Constante asociate, funcții, și tipuri |
| `<type>::...` | Element asociat pentru un tip care nu poate fi numit direct (de exemplu, `<&T>::...`, `<[T]>::...`, etc.) |
| `trait::method(...)` | Dezambiguizarea unui apel de metoda prin numirea tipului care o definește |
| `type::method(...)` | Dezambiguizarea unui apel de metoda prin numirea tipului pentru care este definit |
| `<type as trait>::method(...)` | Dezambiguizarea unui apel de metoda prin numirea trăsăturii și tipului |

Tabelul B-4 arată simboluri care apar în contextul utilizării parametrilor de tip generic.

<span class="caption">Tabelul B-4: Generics</span>

| Simbol | Explicație |
|--------|-------------|
| `path<...>` | Specifică parametrii către tipul generic într-un tip (de exemplu, `Vec<u8>`) |
| `path::<...>`, `method::<...>` | Specifică parametrii către tipul generic, funcție, sau metodă într-o expresie; de multe ori se referă la acesta ca la turbofish (de exemplu, `"42".parse::<i32>()`) |
| `fn ident<...> ...` | Definește funcție generică |
| `struct ident<...> ...` | Definește structura generică |
| `enum ident<...> ...` | Definește enumerația generică |
| `impl<...> ...` | Definește implementarea generică |
| `for<...> type` | Limitele de durată de viață cu rang superior (Higher-ranked lifetime bounds) |
| `type<ident=type>` | Un tip generic în care unul sau mai multe tipuri asociate au atribuiri specifice (de exemplu, `Iterator<Item=T>`) |

Tabelul B-5 arată simboluri care apar în contextul constrângerii parametrilor de tip generic
cu limite de trăsături (Trait bounds).

<span class="caption">Tabelul B-5: Limitările Trăsăturilor</span>

| Simbol | Explicație |
|--------|-------------|
| `T: U` | Parametrul generic `T` limitat la tipurile care implementează `U` |
| `T: 'a` | Tipul generic `T` trebuie să supraviețuiască duratei de viață `'a` (asta înseamnă că tipul nu poate să conțină în mod tranzitiv orice referințe cu duratele de viață mai scurte decât `'a`) |
| `T: 'static` | Tipul generic `T` nu conține alte referințe împrumutate decât cele `'static` |
| `'b: 'a` | Durata de viață generică `'b` trebuie să supraviețuiască duratei de viață `'a` |
| `T: ?Sized` | Permite parametrului de tip generic să fie un tip dinamic de dimensiune |
| `'a + trait`, `trait + trait` | Constrângere de tip compusă |

Tabelul B-6 arată simbolurile care apar în contextul apelării sau definirii de macrouri și specificarea atributelor pe un element.

<span class="caption">Tabelul B-6: Macrouri și Atribute</span>

| Simbol | Explicatie |
|--------|-------------|
| `#[meta]` | Atribut outer |
| `#![meta]` | Atribut inner |
| `$ident` | Substituție macro |
| `$ident:kind` | Captură macro |
| `$(…)…` | Repetitie macro |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Invocare macro |

Tabelul B-7 arată simbolurile care creează comentarii.

<span class="caption">Tabelul B-7: Comentarii</span>

| Simbol | Explicatie |
|--------|-------------|
| `//` | Comentariu de linie |
| `//!` | Comentariu de linie doc intern |
| `///` | Comentariu de linie doc extern |
| `/*...*/` | Comentariu de bloc |
| `/*!...*/` | Comentariu de bloc doc intern |
| `/**...*/` | Comentariu de bloc doc extern |

Tabelul B-6 arată simboluri care apar în contextul apelării sau definirii macrourilor și specificarea atributelor unui element.

<span class="caption">Tabelul B-6: Macrouri și Atribute</span>

| Simbol | Explicație |
|--------|-------------|
| `#[meta]` | Atribut extern |
| `#![meta]` | Atribut intern |
| `$ident` | Substituție macro |
| `$ident:kind` | Captură macro |
| `$(…)…` | Repetare macro |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Invocare macro |

Tabelul B-7 arată simbolurile care creează comentarii.

<span class="caption">Tabelul B-7: Comentarii</span>

| Simbol | Explicație |
|--------|-------------|
| `//` | Comentariu linie |
| `//!` | Comentariu intern documentație linie |
| `///` | Comentariu extern documentație linie |
| `/*...*/` | Comentariu bloc |
| `/*!...*/` | Comentariu intern documentație bloc |
| `/**...*/` | Comentariu extern documentație bloc |

Tabelul B-8 arată simbolurile care apar în contextul folosirii tuplurilor.

<span class="caption">Tabelul B-8: Tuple</span>

| Simbol | Explicație |
|--------|-------------|
| `()` | Tuplă goală (cunoscută ca unit), atât literal cât și tip |
| `(expr)` | Expresie între paranteze |
| `(expr,)` | Expresie tuplă cu un singur element |
| `(type,)` | Tip de tuplă cu un singur element |
| `(expr, ...)` | Expresie tuplă |
| `(type, ...)` | Tip de tuplă |
| `expr(expr, ...)` | Expresie de apel de funcție; de asemenea utilizată pentru a inițializa structuri de tuplă și variante enum de tuplă |
| `expr.0`, `expr.1`, etc. | Indexare de tuplă |

Tabelul B-9 arată contextele în care sunt utilizate acoladele.

<span class="caption">Tabelul B-9: Acolade</span>

| Context | Explicație |
|---------|-------------|
| `{...}` | Expresie bloc |
| `Type {...}` | Literal `struct` |

Tabelul B-10 arată contextele în care sunt utilizate parantezele pătrate.

<span class="caption">Tabelul B-10: Paranteze pătrate</span>

| Context | Explicație |
|---------|-------------|
| `[...]` | Literal de array |
| `[expr; len]` | Literal de array conținând `len` copii ale `expr` |
| `[type; len]` | Tip de array conținând `len` instanțe ale `type` |
| `expr[expr]` | Indexare în colecție. Poate fi supraîncărcată (`Index`, `IndexMut`) |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Indexare în colecție care pretinde a fi secțiune de colecție, folosind `Range`, `RangeFrom`, `RangeTo`, sau `RangeFull` ca „index” |
