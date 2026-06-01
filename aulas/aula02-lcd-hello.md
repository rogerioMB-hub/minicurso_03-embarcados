---
layout: default
title: "Aula 2 — LCD: primeiros caracteres"
---

# Aula 2 — LCD 16x2: primeiros caracteres

⏱️ **Duração:** ~30 min · 🎯 **Nível:** iniciante

## Objetivos

- Entender o que é o **backpack PCF8574** do LCD.
- Adicionar as bibliotecas `lcd_api.py` e `i2c_lcd.py` ao projeto.
- Inicializar o LCD e escrever seu primeiro texto na tela.

## Conceito

O **LCD 16x2** mostra **16 colunas × 2 linhas** de caracteres. Sozinho,
ele precisa de muitos pinos. Por isso usamos um pequeno módulo soldado
atrás dele: o **backpack PCF8574**, que conversa por **I2C** (os dois
fios da Aula 1) e cuida de acionar o display.

Para falar com esse conjunto em MicroPython, usamos duas bibliotecas:

- **`lcd_api.py`** — comandos genéricos de LCD (mover cursor, escrever, limpar).
- **`i2c_lcd.py`** — a "ponte" que envia esses comandos via I2C/PCF8574.

Você adiciona esses dois arquivos no projeto Wokwi (botão **+** → *New File*)
e cola o conteúdo fornecido em `assets/libs/`.

## Circuito

O mesmo da Aula 1 (SDA=21, SCL=22, VCC=5V, GND=GND).

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

> 📁 **Antes de rodar:** adicione no projeto os arquivos `lcd_api.py` e
> `i2c_lcd.py` (conteúdo em `assets/libs/` deste repositório).

`main.py`:

```python
# Aula 2 - "Hello World" no LCD 16x2 via I2C
# ESP32 + Wokwi - MicroPython

from machine import Pin, SoftI2C
from i2c_lcd import I2cLcd
from time import sleep

# --- Configuracao do barramento e do display ---
ENDERECO = 0x27   # descoberto com i2c.scan() na Aula 1
LINHAS   = 2      # o display tem 2 linhas
COLUNAS  = 16     # ... e 16 colunas

i2c = SoftI2C(sda=Pin(21), scl=Pin(22), freq=100000)
# Pico: i2c = SoftI2C(sda=Pin(0), scl=Pin(1), freq=100000)

# Cria o objeto do LCD passando barramento, endereco, linhas e colunas.
lcd = I2cLcd(i2c, ENDERECO, LINHAS, COLUNAS)

# --- Escrevendo na tela ---
lcd.putstr("Ola, mundo!")   # escreve a partir do canto superior esquerdo
sleep(2)

lcd.clear()                 # limpa a tela inteira
lcd.putstr("Automacao")     # primeira linha
lcd.move_to(0, 1)           # vai para coluna 0, linha 1 (segunda linha)
lcd.putstr("Industrial")
```

Resultado esperado no display:

```
+----------------+
|Automacao       |
|Industrial      |
+----------------+
```

> 💡 O LCD do Wokwi não mostra acentos. Por isso escrevemos "Ola" e
> "Automacao" sem acento — no hardware real isso depende da tabela de
> caracteres do display.

## Experimento

1. Troque o texto de `lcd.putstr("Ola, mundo!")` pelo seu nome. Funciona?
2. Escreva uma frase com **mais de 16 letras**. O que acontece com o
   excedente?
3. Comente a linha `lcd.clear()` e rode de novo. O que muda?

## Desafio

**Principal:** mostre seu nome na linha 1 e o nome do curso na linha 2,
ambos **centralizados** (dica: conte as letras e use `move_to` para
escolher a coluna inicial).

**Bônus:** faça o texto "piscar" — escreva, espere 0,5 s, limpe, espere
0,5 s, repita dentro de um `while True`.

## Resumo

- O LCD usa o **backpack PCF8574** para falar I2C.
- Precisamos das bibliotecas **`lcd_api.py`** e **`i2c_lcd.py`**.
- `I2cLcd(i2c, endereco, linhas, colunas)` cria o display.
- `putstr()` escreve, `clear()` limpa, `move_to(coluna, linha)` posiciona.

---

[⬅️ Aula 1](aula01-i2c-scan.md) · [➡️ Aula 3 — Valores dinâmicos](aula03-lcd-dinamico.md)
