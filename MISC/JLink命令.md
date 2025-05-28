
|   功能   |                   命令                    |                        解释                        |
| :----: | :-------------------------------------: | :----------------------------------------------: |
|  Mem   | Mem  [<Zone>:]<Addr>, <NumBytes> (hex)  | Read memory and show corresponding ASCII values. |
|  Mem8  | Mem8  [<Zone>:]<Addr>, <NumBytes> (hex) |                Read  8-bit items.                |
| Mem16  | Mem16 [<Zone>:]<Addr>, <NumItems> (hex) |                Read 16-bit items.                |
| Mem32  | Mem32 [<Zone>:]<Addr>, <NumItems> (hex) |                Read 32-bit items.                |
| Write1 |    W1 [<Zone>:]<Addr>, <Data> (hex)     |               Write  8-bit items.                |
| Write2 |    W2 [<Zone>:]<Addr>, <Data> (hex)     |               Write 16-bit items.                |
| Write4 |    W4 [<Zone>:]<Addr>, <Data> (hex)     |               Write 32-bit items.                |
| Write8 |    W8 [<Zone>:]<Addr>, <Data> (hex)     |               Write 64-bit items.                |
|  Halt  |                  Halt                   |                    Halt CPU.                     |
|   Go   |                   Go                    |               Start CPU if halted.               |
| Reset  |                  Reset                  |                    Reset CPU.                    |
| ResetX |        ResetX <DelayAfterReset>         |        Reset CPU with delay after reset.         |
