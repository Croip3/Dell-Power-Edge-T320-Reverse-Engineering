# Dell-Power-Edge-T320-Reverse-Engineering
The goal is to reuse a Dell Power Edge T320 Case with the original power supple and power distribution as homeserver.

# Power Distribution Board Data Cable (P6 Connector in the Manual)

| Comment | V/GND | Color(text) | Pin | Color || Color | Pin | Color(text) | VGND | Comment |
| --- | --- | ---: | ---: | ---: | - | :--- | :--- | :--- | --- | --- |
| PSU-2 (+12V) when pulled to GND | 3.3V | yellow | 1 | 🟨 || ⬜️ | 2 | white | 3.3V | |
|  | CONT-GND | black | 3 | ⬛️ || ⬜️ | 4 | white |  | |
|  | 0.17V | yellow | 5 | 🟨 || — | 6 | — |  | |
|  | 3.3V / CONT-GND | blue | 7 | 🟦 || 🟨 | 8 | yellow | 0.19V | |
|  |  | green | 9 | 🟩 || 🔲 | 10 | gray | 3.3V | |
| PSU-2 on when pulled to GND |  | red | 11 | 🟥 || 🟨 | 12 | yellow | 3.3V | |
| PSU1 ON signal? |  | purple | 13 | 🟪 || ⬜️ | 14 | white | 0.0–3.3V | Singal stuff going on, changing voltag periodically |
|  |  | purple | 15 | 🟪 || ⬜️ | 16 | white | 3.3V | |
|  | 3.3V / CONT-GND | gray | 17 | 🔲 || 🟦 | 18 | blue | 3.3V | |
|  |  | brown | 19 | 🟫 || 🟩 | 20 | green | 0V | |
| Signal → PSU5 off | GND | black | 21 | ⬛️ || 🟧 | 22 | orange | 0–3.3V / CONT-GND | Singal stuff going on, changing voltag periodically |
|  | CONT-GND | blue | 23 | 🟦 || 🟧 | 24 | orange |  | |
|  | 3.3V / CONT-GND | red | 25 | 🟥 || 🟧 | 26 | orange | CONT-GND | |
|  |  | red | 27 | 🟥 || ⬛️ | 28 | black | CONT-GND | |
|  | CONT-GND | red | 29 | 🟥 || 🟩 | 30 | green |  | |
|  |  | — | 31 | — || ⬜️ | 32 | white |  | |
|  |  | — | 33 | — || ⬜️ | 34 | white |  | |

PSU-1: middle position or left position when viewed from the rear

PSU-2: outer position or right position when viewed from the rear

CONT-GND: Continuity to GND
