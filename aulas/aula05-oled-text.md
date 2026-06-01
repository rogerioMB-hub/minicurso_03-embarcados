---
layout: default
title: "Aula 5 — OLED: primeiros pixels"
---

# Aula 5 — OLED SSD1306: primeiros pixels

⏱️ **Duração:** ~30 min · 🎯 **Nível:** iniciante

## Objetivos

- Conhecer o display **OLED SSD1306** (128×64 pixels) e como ele difere do LCD.
- Entender o conceito de **framebuffer** (a "tela na memória").
- Escrever texto com `text()` e atualizar a tela com `show()`.

## Conceito

O **OLED SSD1306** também é I2C (dois fios, igual ao LCD), mas é bem
diferente por dentro:

| | LCD 16x2 | OLED SSD1306 |
|---|----------|--------------|
| Mostra | só **caracteres** prontos | **pixels** individuais (128×64) |
| Posição | coluna/linha (16×2) | x/y em pixels |
| Desenho | texto e símbolos 5×8 | texto, linhas, formas, imagens |
| Endereço I2C | `0x27` (PCF8574) | `0x3C` |

A grande ideia do OLED é o **framebuffer**: existe uma cópia da tela na
**memória** do ESP32. Você desenha **nessa memória** (texto, linhas...) e
**nada aparece** até chamar `show()`, que envia tudo de uma vez para o
display. É como montar um quadro e só então pendurá-lo na parede.

Fluxo típico:

1. `oled.fill(0)` — apaga a memória (0 = preto).
2. `oled.text("...", x, y)` — desenha na memória.
3. `oled.show()` — joga a memória na tela.

## Circuito

Agora o componente é o **OLED**, não o LCD. Pinos:

- **SDA → GPIO 21**, **SCL → GPIO 22**, **VCC → 3V3**, **GND → GND**.

### diagram.json

> ⚠️ **Validar antes de publicar.**

```json
{
  "version": 1,
  "author": "Mini-curso I2C",
  "editor": "wokwi",
  "parts": [
    { "type": "board-esp32-devkit-c-v4", "id": "esp", "top": 0, "left": 0, "attrs": {} },
    { "type": "board-ssd1306", "id": "oled1", "top": -120, "left": 0, "attrs": {} }
  ],
  "connections": [
    [ "esp:TX0", "$serialMonitor:RX", "", [] ],
    [ "esp:RX0", "$serialMonitor:TX", "", [] ],
    [ "oled1:VCC", "esp:3V3", "red", [ "h-20", "v-30" ] ],
    [ "oled1:GND", "esp:GND.1", "black", [ "h-30", "v40" ] ],
    [ "oled1:DATA", "esp:21", "green", [ "h-40", "v60" ] ],
    [ "oled1:CLK", "esp:22", "blue", [ "h-50", "v70" ] ]
  ],
  "dependencies": {}
}
```

> 💡 No OLED do Wokwi os pinos I2C aparecem como **DATA** (= SDA) e
> **CLK** (= SCL). Confira os nomes ao ligar.

## Código

> 📁 **Antes de rodar:** adicione o arquivo `ssd1306.py` ao projeto
> (conteúdo em `assets/libs/`).

`main.py`:

```python
# Aula 5 - Primeiros textos no OLED SSD1306
# ESP32 + Wokwi - MicroPython

from machine import Pin, SoftI2C
from ssd1306 import SSD1306_I2C
from time import sleep

LARGURA = 128
ALTURA  = 64

i2c = SoftI2C(sda=Pin(21), scl=Pin(22), freq=400000)
# Pico: i2c = SoftI2C(sda=Pin(0), scl=Pin(1), freq=400000)

# Cria o objeto do OLED (o endereco 0x3C e o padrao da biblioteca).
oled = SSD1306_I2C(LARGURA, ALTURA, i2c)

# 1) Limpa a memoria (tela toda preta).
oled.fill(0)

# 2) Desenha textos na MEMORIA, em posicoes (x, y) de PIXELS.
oled.text("Automacao", 0, 0)      # canto superior esquerdo
oled.text("Industrial", 0, 16)    # 16 pixels abaixo
oled.text("OLED SSD1306", 0, 40)

# 3) So agora a tela e atualizada de verdade.
oled.show()
```

> Cada caractere ocupa cerca de **8×8 pixels**. Por isso descemos de 16
> em 16 para dar um respiro entre as linhas. Com 64 pixels de altura
> cabem ~8 linhas de texto.

## Experimento

1. Comente a linha `oled.show()`. O que aparece na tela? Por quê?
2. Mude o `y` do terceiro texto para `56`. Ele ainda cabe? E com `60`?
3. Escreva um texto começando em `x = 100`. O que acontece com as letras
   que passam de 128?

## Desafio

**Principal:** mostre seu nome no topo e, abaixo, um **contador** que
sobe a cada segundo (lembre de `fill(0)` + `show()` a cada atualização,
senão os números se sobrepõem).

**Bônus:** crie um efeito de "digitação": exiba a frase letra por letra,
acrescentando um caractere e chamando `show()` a cada 0,1 s.

## Resumo

- O OLED desenha **pixels** (128×64), não caracteres prontos.
- Endereço I2C costuma ser **`0x3C`**; pinos no Wokwi: **DATA** e **CLK**.
- Tudo é desenhado num **framebuffer** (memória) e só aparece com `show()`.
- `fill(0)` limpa, `text(txt, x, y)` escreve em coordenadas de pixel.

---

[⬅️ Aula 4](aula04-lcd-custom.md) · [➡️ Aula 6 — Formas e barra de progresso](aula06-oled-graficos.md)
