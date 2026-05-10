# HEN Formal Grammar

The grammar below defines the set of valid HEN strings using **EBNF** (Extended Backus-Naur Form, ISO/IEC 14977).

Conventions:
- Terminal symbols are quoted (`"."`, `"x"`, etc.).
- Square brackets `[ … ]` denote optional elements (0 or 1 occurrence).
- Braces `{ … }` denote repetition (0 or more occurrences).
- A vertical bar `|` separates alternatives.
- Comments follow the BNF convention or are noted after a `(* … *)`.

---

## Productions

```ebnf
<hen>                ::= { <part> } ;

<part>               ::= <goban-size>
                       | <goban-row>
                       | <ko>
                       | <last-move>
                       | <turn>
                       | <label-annotation>
                       | <symbol-annotation> ;

(* A. Goban size *)
<goban-size>         ::= "." <number> "x" <number> ;

(* B. Goban content — one row at a time *)
<goban-row>          ::= "_" <number> { <stone-seq-item> }+ ;

(* C. Ko *)
<ko>                 ::= "." <column> <number> ;

(* D. Last move *)
<last-move>          ::= "." <column> <number> <stone>
                       | "." "p" <stone> ;

(* E. Turn *)
<turn>               ::= "." <stone> ;

(* F. Label *)
<label-annotation>   ::= "." <column> <number> "-" <label> ;

(* G. Symbol *)
<symbol-annotation>  ::= "." <column> <number> "-" <mark> ;

(* Goban row internals *)
<stone-seq-item>     ::= [ <column> ] <stone> [ <number> ] ;

(* Primitives *)
<column>             ::= "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H"
                       | "J" | "K" | "L" | "M" | "N" | "O" | "P" | "Q"
                       | "R" | "S" | "T" | "U" | "V" | "W" | "X" | "Y"
                       | "Z" ;
                       (* Uppercase letter, excluding "I" *)

<stone>              ::= "b" | "w" | "r" | "g" | "l" | "y" | "p" ;
                       (* b=black  w=white  r=red  g=green  l=blue  y=yellow  p=purple *)

<mark>               ::= "CR" | "SQ" | "TR" | "MA" ;
                       (* CR=circle  SQ=square  TR=triangle  MA=X *)

<number>             ::= <digit> { <digit> } ;

<digit>              ::= "0" | "1" | "2" | "3" | "4"
                       | "5" | "6" | "7" | "8" | "9" ;

<label>              ::= <label-char> { <label-char> } ;

<label-char>         ::= ? any printable character except whitespace, ".", "_", and "-" ? ;
```

---

## Semantic constraints

The following constraints cannot be expressed in BNF and must be enforced separately:

1. **`<number>`** must represent a positive integer (value > 0). Leading zeros are allowed (e.g. `01` = `1`).
2. In `<goban-row>`, the initial `<number>` is the **row number** and must be within the board height. The first `<column>` (if present) indicates the starting column; if omitted, it defaults to `"A"`.
3. Each `<stone-seq-item>` without a `<column>` refers to the next adjacent column. A `<column>` indicates a skip (jump) to that column.
4. `<stone> <number>` encodes a consecutive run of `<number>` stones of that color. The run must not extend beyond the board width.
5. Parts **A–G** are all optional and may appear in any order. Whitespace characters (spaces, tabs, newlines) may appear freely between parts, as well as before the first part or after the last part, and are ignored by the parser.
6. When the text after `"-"` in a `<label-annotation>` or `<symbol-annotation>` matches a `<mark>` value (`CR`, `SQ`, `TR`, `MA`), it is interpreted as a **symbol annotation** (part G), not a label.

---

## Disambiguation of `.`-prefixed parts

Since multiple part types start with `"."`, a parser reads the characters after `"."` to determine the part type:

| Lookahead pattern                     | Part |
|---------------------------------------|------|
| `<digit>+` `"x"` `<digit>+`           | A — goban size |
| `"p"` `<stone>`                       | D — last move (pass) |
| `<column>` `<number>` `"-"` `<mark>`  | G — symbol |
| `<column>` `<number>` `"-"` `<label>` | F — label |
| `<column>` `<number>` `<stone>`       | D — last move (placement) |
| `<column>` `<number>`                 | C — ko |
| `<stone>`                             | E — turn |

`<column>` (uppercase, no `I`) and `<stone>` (lowercase) never overlap, so the first character after `"."` unambiguously selects the branch.
