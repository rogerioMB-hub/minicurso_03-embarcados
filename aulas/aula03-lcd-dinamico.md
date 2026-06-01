---
layout: default
title: "Aula 3 — LCD: valores dinâmicos"
---

# Aula 3 — LCD 16x2: posição e valores dinâmicos

⏱️ **Duração:** ~30 min · 🎯 **Nível:** iniciante

## Objetivos

- Posicionar o cursor com precisão usando `move_to`.
- Exibir **valores que mudam** (contadores, leituras).
- Atualizar só uma parte da tela sem piscar tudo.

## Conceito

Um display só é útil de verdade quando mostra **informação que muda**:
a temperatura de um sensor, um contador, o estado de uma máquina. Para
isso precisamos:

1. **Posicionar** o cursor onde queremos escrever (`move_to(coluna, linha)`).
2. **Converter** o número em texto com `str(...)`, porque o LCD só
   escreve **caracteres**.
3. **Limpar o rastro** do valor anterior — senão `9` vira `10` e sobra
   um `0` antigo na tela (`19`).

O truque do rastro: escrevemos o valor seguido de **espaços** para
"apagar" sobras, ou usamos `move_to` e reescrevemos o campo todo.

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

`main.py`:

```python
# Aula 3 - Contador ao vivo no LCD
# ESP32 + Wokwi - MicroPython

from machine import Pin, SoftI2C
from i2c_lcd import I2cLcd
from time import sleep

ENDERECO = 0x27
i2c = SoftI2C(sda=Pin(21), scl=Pin(22), freq=100000)
# Pico: i2c = SoftI2C(sda=Pin(0), scl=Pin(1), freq=100000)
lcd = I2cLcd(i2c, ENDERECO, 2, 16)

# Texto fixo (rotulo) que NAO muda: escrevemos so uma vez.
lcd.move_to(0, 0)
lcd.putstr("Contador:")

contador = 0
while True:
    # Posiciona no inicio da segunda linha.
    lcd.move_to(0, 1)
    # Escreve o numero seguido de espacos para apagar o valor anterior.
    lcd.putstr(str(contador) + "   ")
    contador = contador + 1
    sleep(1)
```

No display você verá `Contador:` fixo em cima e o número subindo embaixo,
sem deixar rastro graças aos três espaços.

## Experimento

1. Remova os `"   "` (espaços) do `putstr`. Conte de 9 para 10 e de 99
   para 100. O que aparece de errado?
2. Troque `sleep(1)` por `sleep(0.2)`. O contador fica mais rápido?
3. Adicione um segundo rótulo `"Total"` em outra posição. Cabe?

## Desafio

**Principal:** simule a leitura de um "sensor" gerando um número aleatório
de 20 a 30 (`import random; random.randint(20, 30)`) e mostre como
`Temp: 25 C`, atualizando a cada 2 segundos. Cuide do rastro!

**Bônus:** mostre **duas informações** ao mesmo tempo — um contador na
linha 1 e a "temperatura" na linha 2 — cada um atualizando no seu ritmo.

## Resumo

- `move_to(coluna, linha)` escolhe onde escrever.
- Números viram texto com `str(...)`.
- Escreva **espaços extras** para apagar o valor antigo e evitar rastro.
- Escreva rótulos fixos **uma vez só**; atualize apenas o que muda.

---

[⬅️ Aula 2](aula02-lcd-hello.md) · [➡️ Aula 4 — Caracteres customizados](aula04-lcd-custom.md)
