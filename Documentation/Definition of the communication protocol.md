# Definition of the communication protocol

### Microcontroller -> Server

| Command  | Bit No. + Value | Byte 1<br />(Controlbyte) | Byte 2 | Byte 3 | Byte 4 | Byte 5 | Description |                              |
| -------- | ------ | ------ | ------ | ------ | ------ | ---------------------------- | ---------------------------- | ---------------------------- |
| Start    | Bit 8 = 1 | 00000001 | 0 / 1  |        |        |        | Byte 2 = 0 => PvP<br />Byte 2 = 1 => PvEngine | [Start](#MC_Start) |
| Reset    | Bit 7 = 1 | 00000010 |        |        |        |        |  |                              |
| New Turn | Bit 6 = 1 | 00000100 | Ax     | Ay     | Bx     | By     | Byte 2 + 3 => Startfield (A)<br />Byte 3 + 4 => Endfield (B) | [New Turn](#MC_NewTurn)                             |
| Promotion Ack | Bit 5 = 1 | 00001000 | [Figure](#Def_Figures) |  |  |  | Byte 2 = Figure to replace the pawn |  |
| Error  | Bit 1 = 1 | 10000000                |        |        |        |        |        |                              |



#### Examples

##### <a name="MC_Start">Start</a>

| Case              | Byte 1 | Byte 2 |
| ----------------- | ------ | ------ |
| Player vs. Player | 1      | 0      |
| Player vs. Engine | 1      | 1      |



##### <a name="MC_NewTurn">NewTurn</a>

| Case               | Byte 1 | Byte 2 | Byte 3 | Byte 4 | Byte 5 |
| ------------------ | ------ | ------ | ------ | ------ | ------ |
| Move from A1 to A2 | 4      | 0      | 0      | 0      | 1      |
| Move from B8 to C8 | 4      | 1      | 7      | 2      | 7      |



### Server -> Microcontroller

The server answers a command of the MC with a new command and an enclosed AI move (if the last player move was legal).

The first byte of each command is a combination of all command bits.

| Command    | Bit No. + Value | Byte 1 | Byte 2 | Byte 3 | Byte 4 | Byte 5 | AI moves enclosed | Example            |
| ---------- | ------ | ------ | ------ | ------ | ------ | ----------------- | ------------------ | ------------------ |
| OK         | Bit 2 = 0 | 00XXXXX0 |        |        |        |        | X                 |  |
| Castling   | Bit 7 = 1 | 00XX0010 | RAx    | RAy    | RBx    | RBy    | X                 | [Castling](#Srv_Castling)                   |
| En passant | Bit 6 = 1 | 00XX0100 | Px     | Py     |        |        | X                 | [En passant](#Srv_EnPassant)                   |
| Promotion  | Bit 5 = 1 | 00XX1000 | Px     | Py     |        |        | X                 | [Promotion](#Srv_Promotion)                   |
| Check    | Bit 4 = 1 | 0001XXX0 | Kx     | Ky     |        |        | X                 | [Check](#Srv_Check)                   |
| Checkmate  | Bit 3 = 1 | 0010XXX0 | Kx     | Ky     |        |        |                   | [Checkmate](#Srv_Checkmate)                   |
| Illegal    | Bit 2 = 1 | 01000000 | Bx     | By     | Ax     | Ay     |                   | [Illegal](#Srv_Illegal) |
| Error      | Bit 1 = 1 | 10000000 |        |        |        |        |                   |                |



#### Examples

##### <a name="Srv_Castling">Castling</a>

- Short castling white (King E1G1)

  - ![SCD_castle_kingside](../../../Embedded%20Software/Praktikum/Documentation/img/SCD_castle_kingside.png)

    - (Wikimedia) [^1]

  - ```
    2 7 0 5 0   + next AI move
    ```

- Long castling white (King E1C1)

  - ![SCD_castle_queenside](../../../Embedded%20Software/Praktikum/Documentation/img/SCD_castle_queenside.png)

    - (Wikimedia)[^2]

  - ```
    2 0 0 3 0   + next AI move
    ```



##### <a name="Srv_EnPassant">En passant</a>

- Pawn D5C6 (After Pawn C7C5)

  - ![SCD_en_passant](../../../Embedded%20Software/Praktikum/Documentation/img/SCD_en_passant.png)

    - (Wikimedia) [^3]

  - ```
    4 2 4   + next AI move
    ```



##### <a name="Srv_Promotion">Promotion</a>

- Pawn A7A8 (becomes Queen)

  - ```
    8 0 7   + next AI move
    ```

- Pawn H2H1 (becomes Knight)

  - ```
    8 7 0   + next AI move
    ```

The server has to wait for a Promotion Ack-command by the MC after sending this.



##### <a name="Srv_Check">Check</a>

- Rook A2C2 (checks King C6)

  - ```
    16 2 5   + next AI move
    ```



##### <a name="Srv_Checkmate">Checkmate</a>

- Queen D8H4 (checkmates King E1)

  - ```
    32 4 0
    ```



##### <a name="Srv_Illegal">Illegal</a>

- Pawn A2A1 (which is an illegal move)

  - ```
    64 0 0 0 1
    ```



#### AI moves

| Command           | Bit No + Value | Byte 1 | Byte 2 | Byte 3 | Byte 4 | Byte 5 | Byte 6 | Byte 7 | Byte 8 | Byte 9 | Example                 |
| ----------------- | ------ | -------- | -------- | -------- | -------- | ----------------------- | ----------------------- | ----------------------- | ----------------------- | ----------------------- | ----------------------- |
| OK (regular move) | Bit 2 = 1 | 64    | Ax       | Ay       | Bx       | By       |        |        |        |        | [OK](#AI_OK) |
| Castling          | Bit 7 = 1 | 2     | KAx    | KAy    | KBx    | KBy    | RAx   | RAy   | RBx   | RBy   | [Castling](#AI_Castling)  |
| En passant        | Bit 6 = 1 | 4     | Ax     | Ay     | Bx | By | Px | Py     |        |          | [En passant](#AI_EnPassant)  |
| Promotion         | Bit 5 = 1 | 8     | Ax     | Ay     | Bx     | By | [Figure](#Def_Figures) |  |        |          | [Promotion](#AI_Promotion) |
| Check             | Bit 4 = 1    | 16    | Ax     | Ay     | Bx | By | Kx | Ky     |        |          | [Check](#AI_Check)  |
| Checkmate         | Bit 3 = 1 | 32   | Ax     | Ay     | Bx | By | Kx | Ky     |        |          | [Checkmate](#AI_Checkmate) |



##### AI moves examples

###### <a name="AI_OK">OK (regular move)</a>

- A1A2

  - ```
    64 0 0 0 1
    ```

- B8C8

  - ```
    64 1 7 2 7
    ```



###### <a name="AI_Castling">Castling</a>

- Short castling white (King E1G1)

  - ![SCD_castle_kingside](/img/SCD_castle_kingside.png)

    - (Wikimedia) [^1]

  - ```
    2 4 0 6 0 7 0 5 0
    ```

- Long castling white (King E1C1)

  -   ![SCD_castle_queenside](/img/SCD_castle_queenside.png)

    -   (Wikimedia)[^2]

  - ```
    2 4 0 2 0 0 0 3 0
    ```



###### <a name="AI_EnPassant">En passant</a>

- Pawn D5C6 (After Pawn C7C5)

  -   ![SCD_en_passant](img/SCD_en_passant.png)

    - (Wikimedia) [^3]

  - ```
    4 3 5 2 6 2 4
    ```



###### <a name="AI_Promotion">Promotion</a>

- Pawn A7A8 (becomes Queen)

  - ```
    8 0 6 0 7 Q
    ```

- Pawn H2H1 (becomes Knight)

  - ```
    8 7 1 7 0 N
    ```



###### <a name="AI_Check">Check</a>

- Rook A2C2 (checks King C6)

  - ```
    16 0 1 2 1 2 5
    ```



###### <a name="AI_Checkmate">Checkmate</a>

- Queen D8H4 (checkmates King E1)

  - ```
    32 3 7 7 3 4 0
    ```




### Definitions

#### <a name="Def_Figures">Figures</a>

| Figure | Char | ASCII (Dec) | ASCII (Hex) |
| ------ | ---- | ----------- | ----------- |
| Pawn   | P    | 80          | 50          |
| Knight | N    | 78          | 4E          |
| Bishop | B    | 66          | 42          |
| Rook   | R    | 82          | 52          |
| Queen  | Q    | 81          | 51          |
| King   | K    | 75          | 4B          |



### Sources

[^1]: <a href="http://creativecommons.org/licenses/by-sa/3.0/" title="Creative Commons Attribution-Share Alike 3.0">CC BY-SA 3.0</a>, <a href="https://commons.wikimedia.org/w/index.php?curid=278291">Link</a>

[^2]: <a href="http://creativecommons.org/licenses/by-sa/3.0/" title="Creative Commons Attribution-Share Alike 3.0">CC BY-SA 3.0</a>, <a href="https://commons.wikimedia.org/w/index.php?curid=278294">Link</a>

[^3]: <a href="http://creativecommons.org/licenses/by-sa/3.0/" title="Creative Commons Attribution-Share Alike 3.0">CC BY-SA 3.0</a>, <a href="https://commons.wikimedia.org/w/index.php?curid=278302">Link</a> 
