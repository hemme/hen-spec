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
A HEN string is composed of parts A through H. All parts are optional and may appear in any order. Whitespace, newlines, and tabs are completely ignored: a HEN string can exist on a single line or span multiple lines by breaking between parts.

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

H. Indicate the **player order** (turn sequence):
`~{stone}{stone}…` - e.g.: `~bw` (default, 2-player), `~bwr` (3-player), `~bwrg` (4-player)
The player order defines how the color of a numbered stone (`~{number}` in goban rows) is determined: move *k* gets the color at position `(k − 1) mod N`, where *N* is the number of stones in the order. When part H is omitted, `~bw` is assumed.

## Goban content

The content of the goban is constructed by observing the board from Black's side and proceeding one row at a time, from top to bottom.
If a row contains stones, write:

`_{number}` - e.g., in a 19x19 goban, the topmost row will be: `_19`

Indicate the column letter (uppercase) of the first stone in that row, followed by its color. For each subsequent adjacent stone, only the color is needed — the column is inferred from the sequence:

`_{number}{letter}{stone...}` - e.g.: `_19Abwbbwww`

The column letter `A` may be omitted when the first stone is in column A; e.g. `_19Ab` is equivalent to `_19b`.

The `{stone}{number}` form can be used to encode consecutive runs of the same color; e.g. `_19bwbbwww` is equivalent to `_19bwb2w3`.

To indicate a numbered move on an intersection, use the `~{number}` syntax (preceded by an optional column letter); e.g. `_19~1` or `_19K~12`. The color of a numbered stone is not explicitly written — it is inferred from the move number using the player order (see part H). In standard 2-player Go (the default), odd-numbered moves are Black and even-numbered moves are White.

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
_19 bwb2w3
_10 Kb
_08 Kbw
_07 JbwMw
_06 Kbw
_01 Cw3bNwb2
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

The following HEN (241 characters) represents the goban position of the famous **Ear-reddening Game**, played in 1846 by Shusaku against Gennan Inseki, immediately after the eponymous move. The marks are as shown in the Sensei's Library [article about the game](https://senseis.xmp.net/?EarReddeningMove#toc2):

`_19Kbw2_18DbKbwNwPw2b_17Cw2FbJwb2w2Pwb_16Mb3Rb_15LbQb2_14Qbw2_13Ow3b3_12Pbw3b_11KbNbw2b3_10Nw2bRbw_9CwPwb2w_8Pwbwb_7NwPwbw2_6CwKbMbwPwb_5GbJwMbwbwbw_4CbEbHbMbw2bRw_3FbwbwLw2b4w2_2GbwKw2Nwb2Rbw_1JwMwObQbSb.K11b.w.M6-SQ.M5-SQ.M4-SQ.K6-SQ.J5-CR`

Multiple lines (for clarity / during manual entry):

```hen
_19 Kbw2
_18 DbKbwNwPw2b
_17 Cw2FbJwb2w2Pwb
_16 Mb3Rb_15LbQb2
_14 Qbw2_13Ow3b3
_12 Pbw3b
_11 KbNbw2b3
_10 Nw2bRbw
_09 CwPwb2w
_08 Pwbwb
_07 NwPwbw2
_06 CwKbMbwPwb
_05 GbJwMbwbwbw
_04 CbEbHbMbw2bRw
_03 FbwbwLw2b4w2
_02 GbwKw2Nwb2Rbw
_01 JwMwObQbSb
.K11b.w
.M6-SQ.M5-SQ.M4-SQ.K6-SQ.J5-CR
```

Equivalent SGF (521 characters):

```sgf
(;GM[1]FF[4]CA[UTF-8]SZ[19]
;AB[ja][db][jb][qb][fc][jc][kc][pc][ld][md][nd][qd][ke][pe][qe][pf][qg][rg][sg][oh][sh][mi][pi][qi][ri][oj][qj][pk][qk][pl][rl][pm][jn][ln][pn][go][lo][no][po][cp][ep][hp][lp][op][fq][hq][mq][nq][oq][pq][gr][nr][or][qr][ns][ps][rs]AW[ka][la][kb][mb][ob][pb][cc][dc][ic][lc][mc][oc][qf][rf][ng][og][pg][ph][qh][rh][ni][oi][mj][nj][rj][ck][ok][rk][ol][ql][mm][om][qm][rm][cn][mn][on][io][mo][oo][qo][mp][np][qp][gq][iq][kq][lq][qq][rq][hr][jr][kr][mr][rr][is][ls]
;B[ji]SQ[ln][lo][lp][jn]CR[io])
```


Equivalent URL-encoded SGF (1031 characters):

```
%28%3BGM%5B1%5DFF%5B4%5DCA%5BUTF-8%5DSZ%5B19%5D%0A%3BAB%5Bja%5D%5Bdb%5D%5Bjb%5D%5Bqb%5D%5Bfc%5D%5Bjc%5D%5Bkc%5D%5Bpc%5D%5Bld%5D%5Bmd%5D%5Bnd%5D%5Bqd%5D%5Bke%5D%5Bpe%5D%5Bqe%5D%5Bpf%5D%5Bqg%5D%5Brg%5D%5Bsg%5D%5Boh%5D%5Bsh%5D%5Bmi%5D%5Bpi%5D%5Bqi%5D%5Bri%5D%5Boj%5D%5Bqj%5D%5Bpk%5D%5Bqk%5D%5Bpl%5D%5Brl%5D%5Bpm%5D%5Bjn%5D%5Bln%5D%5Bpn%5D%5Bgo%5D%5Blo%5D%5Bno%5D%5Bpo%5D%5Bcp%5D%5Bep%5D%5Bhp%5D%5Blp%5D%5Bop%5D%5Bfq%5D%5Bhq%5D%5Bmq%5D%5Bnq%5D%5Boq%5D%5Bpq%5D%5Bgr%5D%5Bnr%5D%5Bor%5D%5Bqr%5D%5Bns%5D%5Bps%5D%5Brs%5DAW%5Bka%5D%5Bla%5D%5Bkb%5D%5Bmb%5D%5Bob%5D%5Bpb%5D%5Bcc%5D%5Bdc%5D%5Bic%5D%5Blc%5D%5Bmc%5D%5Boc%5D%5Bqf%5D%5Brf%5D%5Bng%5D%5Bog%5D%5Bpg%5D%5Bph%5D%5Bqh%5D%5Brh%5D%5Bni%5D%5Boi%5D%5Bmj%5D%5Bnj%5D%5Brj%5D%5Bck%5D%5Bok%5D%5Brk%5D%5Bol%5D%5Bql%5D%5Bmm%5D%5Bom%5D%5Bqm%5D%5Brm%5D%5Bcn%5D%5Bmn%5D%5Bon%5D%5Bio%5D%5Bmo%5D%5Boo%5D%5Bqo%5D%5Bmp%5D%5Bnp%5D%5Bqp%5D%5Bgq%5D%5Biq%5D%5Bkq%5D%5Blq%5D%5Bqq%5D%5Brq%5D%5Bhr%5D%5Bjr%5D%5Bkr%5D%5Bmr%5D%5Brr%5D%5Bis%5D%5Bls%5D%0A%3BB%5Bji%5DSQ%5Bln%5D%5Blo%5D%5Blp%5D%5Bjn%5DCR%5Bio%5D%29
```