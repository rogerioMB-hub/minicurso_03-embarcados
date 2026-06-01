---
layout: default
title: "Aula 4 — LCD: caracteres customizados"
---

# Aula 4 — LCD 16x2: caracteres customizados (máscara de bits!)

⏱️ **Duração:** ~30 min · 🎯 **Nível:** iniciante

## Objetivos

- Entender que cada caractere do LCD é uma **matriz de 5×8 pixels**.
- Desenhar um símbolo próprio usando **bits** (retomando o Mini-curso 01!).
- Carregar o símbolo com `custom_char()` e exibi-lo na tela.

## Conceito

🔗 **Conexão com o Mini-curso 01 (Bitwise):** lá você aprendeu que um
número guarda **bits** (0s e 1s) e que podíamos montar "máscaras". Aqui
isso vira **desenho na tela**!

Cada caractere do LCD é uma grade de **5 colunas × 8 linhas** de pixels.
Para criar um símbolo, descrevemos **cada linha como um número de 5 bits**:
bit `1` = pixel aceso, bit `0` = pixel apagado.

Veja um coração desenhado na grade (só as 5 colunas da direita contam):

```
coluna:  4 3 2 1 0     binario   numero
linha 0: . # . # .     0b01010 =  10
linha 1: # # # # #     0b11111 =  31
linha 2: # # # # #     0b11111 =  31
linha 3: # # # # #     0b11111 =  31
linha 4: . # # # .     0b01110 =  14
linha 5: . . # . .     0b00100 =   4
linha 6: . . . . .     0b00000 =   0
linha 7: . . . . .     0b00000 =   0
```

Cada `#` é um bit `1`. Lendo cada linha como binário, obtemos a lista de
8 números que **é** o desenho. Note como escrever `0b01010` é literalmente
montar uma **máscara de bits** — exatamente a ideia do curso 1.

## Circuito

O mesmo das aulas anteriores (SDA=21, SCL=22).

### diagram.json

> ⚠️ **Validar antes de publicar.**

```json
{
  "version": 1,
  "author": "Mini-curso I2C",
  "editor": "wokwi",
  "parts": [
    { "type": "board-esp32-devkit-c-v4", "id": "esp", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-lcd1602", "id": "lcd1", "top": -130, "left": -20, "attrs": { "pins": "i2c" } }
  ],
  "connections": [
    [ "esp:TX0", "$serialMonitor:RX", "", [] ],
    [ "esp:RX0", "$serialMonitor:TX", "", [] ],
    [ "lcd1:VCC", "esp:5V", "red", [ "h-20", "v-30" ] ],
    [ "lcd1:GND", "esp:GND.1", "black", [ "h-30", "v40" ] ],
    [ "lcd1:SDA", "esp:21", "green", [ "h-40", "v60" ] ],
    [ "lcd1:SCL", "esp:22", "blue", [ "h-50", "v70" ] ]
  ],
  "dependencies": {}
}
```

## Código

> 📁 **Antes de rodar:** este código importa `i2c_lcd`, que por sua vez
> usa `lcd_api`. Garanta que os **dois** arquivos — `lcd_api.py` e
> `i2c_lcd.py` — estejam no projeto Wokwi (botão **+** → *New File*).
> Conteúdo em [`assets/libs/`](../assets/libs/). Veja o passo a passo na
> [Aula 1](aula01-i2c-scan.md#como-adicionar-uma-biblioteca-módulo-no-wokwi).

`main.py`:

```python
# Aula 4 - Caractere customizado (coracao) usando mascara de bits
# ESP32 + Wokwi - MicroPython

from machine import Pin, SoftI2C
from i2c_lcd import I2cLcd
from time import sleep

ENDERECO = 0x27
i2c = SoftI2C(sda=Pin(21), scl=Pin(22), freq=100000)
# Pico: i2c = SoftI2C(sda=Pin(0), scl=Pin(1), freq=100000)
lcd = I2cLcd(i2c, ENDERECO, 2, 16)

# Cada numero abaixo e uma LINHA de 5 bits do desenho 5x8.
# 1 = pixel aceso, 0 = pixel apagado. Compare com o desenho da aula!
coracao = bytearray([
    0b01010,   # . # . # .
    0b11111,   # # # # # #
    0b11111,   # # # # # #
    0b11111,   # # # # # #
    0b01110,   # . # # # .
    0b00100,   # . . # . .
    0b00000,   # . . . . .
    0b00000,   # . . . . .
])

# Carrega o desenho no "slot" 0 da memoria do LCD (slots vao de 0 a 7).
lcd.custom_char(0, coracao)

# Mostra um texto e o coracao. chr(0) chama o caractere do slot 0.
lcd.putstr("MicroPython ")
lcd.putchar(chr(0))
```

No display aparece `MicroPython ♥`.

> 💡 O LCD guarda até **8 caracteres customizados** (slots 0 a 7).
> Use `chr(n)` para exibir o do slot `n`.

## Experimento

1. Apague a primeira linha do coração (troque `0b01010` por `0b00000`).
   O que muda no desenho?
2. Desenhe na grade de papel uma **seta para cima** e converta cada linha
   em binário. Carregue-a e exiba.
3. Quantos caracteres customizados cabem ao mesmo tempo na memória do LCD?

## Desafio

**Principal:** crie **dois** símbolos (ex.: um "boneco" e um "sino"),
carregue nos slots 0 e 1, e exiba os dois lado a lado com um texto.

**Bônus:** faça um **barra de carregamento por caractere**: crie 5
símbolos, cada um com mais colunas acesas (1, 2, 3, 4 e 5 colunas), e
exiba-os em sequência na mesma posição para simular uma barra enchendo —
uma prévia do que faremos com pixels no OLED.

## Resumo

- Cada caractere do LCD é uma matriz **5×8**.
- Cada linha vira um **número de 5 bits** (máscara: `1`=aceso, `0`=apagado).
- `custom_char(slot, bytearray(...))` carrega o desenho (slots 0–7).
- `chr(slot)` (ou `putchar(chr(slot))`) exibe o símbolo.
- 🔗 É a mesma lógica de **bits e máscaras** do Mini-curso 01, agora visual.

---

[⬅️ Aula 3](aula03-lcd-dinamico.md) · [➡️ Aula 5 — OLED: primeiros pixels](aula05-oled-text.md)
