# HEN (Hemme Notation)

**HEN (Hemme Notation)** is a lightweight, text-based format designed for efficiently encoding and sharing Go board positions.

Inspired by the **Forsyth-Edwards Notation (FEN)** used in chess, HEN allows developers and players to represent the *goban* state — including board dimensions, stone placement, the *Ko* situation, and whose turn it is — in a reasonably compressed yet human-readable string.

## Why HEN?

While SGF (Smart Game Format) is the absolute standard for recording complete Go games and complex variations (move trees), it is not ideal for representing a single, static board position. SGF files can be verbose and are not easily embeddable.

HEN fills this gap by providing a compact, snapshot-based format representing a single state of the game. This makes it perfect for:
- Sharing specific board positions via URLs, query parameters, or chat messages.
- Indexing databases of *Tsumego* (Go problems) or specific board states.
- Writing concise, readable unit tests for Go applications and bots.
- Embedding board states in documentation without the overhead of full SGF trees.

## Structure
A HEN string is composed of parts A through G. All parts are optional and may appear in any order. Whitespace, newlines, and tabs are completely ignored: a HEN string can exist on a single line or span multiple lines by breaking between parts.

A. Indicate the goban size:
`.{number}x{number}` - e.g.: `.19x19`

B. Indicate the **goban content**:
`_{goban}` - e.g. `_19bwb2w3_10Kb_1Cw3bNwbb`

C. Indicate if there is a ko:
`.{letter}{number}` - e.g. `.L7`

D. Indicate the last move:
- Placement: `.{letter}{number}{stone}` - e.g. `.K7w`
- Pass: `.p{stone}` - e.g. `.pw`

E. Indicate whose turn it is:
`.{stone}` - e.g.: `.b`

F. Indicate a label:
`.{letter}{number}-{label}` - e.g.: `.K10-a` 

G. Indicate a symbol:
`.{letter}{number}-{mark}` - e.g.: `.K10-X` 

## Goban content

The content of the goban is constructed by observing the board from Black's side and proceeding one row at a time, from top to bottom.
If a row contains stones, write:

`_{number}` - e.g., in a 19x19 goban, the topmost row will be: `_19`

Indicate the column letter (uppercase) of the first stone in that row, followed by its color. For each subsequent adjacent stone, only the color is needed — the column is inferred from the sequence:

`_{number}{letter}{stone...}` - e.g.: `_19Abwbbwww`

The column letter `A` may be omitted when the first stone is in column A; e.g. `_19Ab` is equivalent to `_19b`.

The `{stone}{number}` form can be used to encode consecutive runs of the same color; e.g. `_19bwbbwww` is equivalent to `_19bwb2w3`.

Repeat for each row containing at least 1 stone.

## Formats
`{number}`: integer > 0. Leading zero is allowed.

`{letter}`: an uppercase alphabetic letter except I.

`{stone}`: a lowercase letter indicating the color:
- `b`: Black
- `w`: White

For multi-color Go, you can also use:
- `r`: Red
- `g`: Green
- `l`: Blue
- `y`: Yellow
- `p`: Purple

`{label}`: a simple text (typically a letter or a number).

`{mark}`: can be `CR`, `SQ`, `TR`, `MA` (respectively for: circle, square, triangle, X) 

## Examples

### 1) An arbitrary position
Single line:
`.19x19_19bwb2w3_10Kb_8Kbw_7JbwMw_6Kbw_1Cw3bNwb2.L7.K7w.b`

Multiple lines:
```hen
.19x19
_19bwb2w3
_10Kb
_8Kbw
_7JbwMw
_6Kbw
_1Cw3bNwb2
.L7
.K7w
.b
```

Equivalent SGF:

```sgf
(;GM[1]FF[4]SZ[19]PB[Black]PW[White]
;AB[aa][ca][da][jj][jl][im][km][jn][fs][ns][os]AW[ba][ea][fa][ga][kl][lm][kn][cs][ds][es][ms]
;W[jm])
```

### 2) The famous Ear-reddening move

The following HEN represents the goban position of the famous **Ear-reddening Game**, played in 1846 by Shusaku against Gennan Inseki, immediately after the eponymous move. The marks are as shown in the Sensei's Library [article about the game](https://senseis.xmp.net/?EarReddeningMove#toc2):

`_19Kbw2_18DbKbwNwPw2b_17Cw2FbJwb2w2Pwb_16Mb3Rb_15LbQb2_14Qbw2_13Ow3b3_12Pbw3b_11KbNbw2b3_10Nw2bRbw_9CwPwb2w_8Pwbwb_7NwPwbw2_6CwKbMbwPwb_5GbJwMbwbwbw_4CbEbHbMbw2bRw_3FbwbwLw2b4w2_2GbwKw2Nwb2Rbw_1JwMwObQbSb.K11b.w.M6-SQ.M5-SQ.M4-SQ.K6-SQ.J5-CR`

Equivalent SGF:

```sgf
(;GM[1]FF[4]CA[UTF-8]SZ[19]
;AB[ja][db][jb][qb][fc][jc][kc][pc][ld][md][nd][qd][ke][pe][qe][pf][qg][rg][sg][oh][sh][mi][pi][qi][ri][oj][qj][pk][qk][pl][rl][pm][jn][ln][pn][go][lo][no][po][cp][ep][hp][lp][op][fq][hq][mq][nq][oq][pq][gr][nr][or][qr][ns][ps][rs]AW[ka][la][kb][mb][ob][pb][cc][dc][ic][lc][mc][oc][qf][rf][ng][og][pg][ph][qh][rh][ni][oi][mj][nj][rj][ck][ok][rk][ol][ql][mm][om][qm][rm][cn][mn][on][io][mo][oo][qo][mp][np][qp][gq][iq][kq][lq][qq][rq][hr][jr][kr][mr][rr][is][ls]
;B[ji]SQ[ln][lo][lp][jn]CR[io])
```