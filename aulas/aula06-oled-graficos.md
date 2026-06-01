---
layout: default
title: "Aula 6 — OLED: formas e barra de progresso"
---

# Aula 6 — OLED SSD1306: formas e barra de progresso

⏱️ **Duração:** ~30 min · 🎯 **Nível:** iniciante

## Objetivos

- Desenhar **pontos, linhas e retângulos** no OLED.
- Diferenciar contorno (`rect`) de preenchido (`fill_rect`).
- Montar uma **barra de progresso** combinando as primitivas.

## Conceito

Como o OLED controla cada pixel, podemos desenhar **formas geométricas**.
A biblioteca `ssd1306` (que herda do `framebuf` do MicroPython) já traz
as funções de desenho:

| Função | O que faz |
|--------|-----------|
| `pixel(x, y, cor)` | acende/apaga um único ponto |
| `hline(x, y, comp, cor)` | linha **horizontal** |
| `vline(x, y, comp, cor)` | linha **vertical** |
| `line(x1, y1, x2, y2, cor)` | linha entre dois pontos |
| `rect(x, y, larg, alt, cor)` | retângulo **só contorno** |
| `fill_rect(x, y, larg, alt, cor)` | retângulo **preenchido** |

A **cor** é simples: `1` = pixel aceso (branco), `0` = apagado (preto).
Lembre: nada aparece sem `show()`.

Com `rect` (a "moldura") e `fill_rect` (o "enchimento") já dá para fazer
uma **barra de progresso**: desenhamos a moldura fixa e preenchemos por
dentro uma largura proporcional ao percentual.

## Circuito

O mesmo da Aula 5 (OLED: DATA=21, CLK=22, VCC=3V3, GND=GND).

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

## Código

> 📁 Use o mesmo `ssd1306.py` da Aula 5.

`main.py`:

```python
# Aula 6 - Formas e barra de progresso no OLED
# ESP32 + Wokwi - MicroPython

from machine import Pin, SoftI2C
from ssd1306 import SSD1306_I2C
from time import sleep

LARGURA = 128
ALTURA  = 64

i2c = SoftI2C(sda=Pin(21), scl=Pin(22), freq=400000)
# Pico: i2c = SoftI2C(sda=Pin(0), scl=Pin(1), freq=400000)
oled = SSD1306_I2C(LARGURA, ALTURA, i2c)

# --- Parte 1: um pouco de cada forma ---
oled.fill(0)
oled.text("Formas:", 0, 0)
oled.pixel(10, 20, 1)                 # um ponto
oled.line(20, 16, 60, 28, 1)          # uma linha inclinada
oled.rect(70, 16, 40, 16, 1)          # retangulo so contorno
oled.fill_rect(70, 36, 40, 16, 1)     # retangulo preenchido
oled.show()
sleep(2)

# --- Parte 2: barra de progresso ---
# A moldura: comeca em x=14, y=40, com 100 de largura e 12 de altura.
BARRA_X = 14
BARRA_Y = 40
BARRA_L = 100
BARRA_A = 12

for porcento in range(0, 101, 5):     # 0, 5, 10 ... 100
    oled.fill(0)
    # Titulo e numero
    oled.text("Carregando", 20, 8)
    oled.text(str(porcento) + "%", 50, 24)
    # Moldura fixa da barra (contorno).
    oled.rect(BARRA_X, BARRA_Y, BARRA_L, BARRA_A, 1)
    # Enchimento proporcional: largura = (porcento/100) * largura interna.
    largura_cheia = (BARRA_L - 2) * porcento // 100
    oled.fill_rect(BARRA_X + 1, BARRA_Y + 1, largura_cheia, BARRA_A - 2, 1)
    oled.show()
    sleep(0.2)

oled.fill(0)
oled.text("Concluido!", 24, 28)
oled.show()
```

A tela mostra primeiro um mostruário de formas e depois uma barra
enchendo de 0 a 100%, terminando em "Concluido!".

## Experimento

1. Troque `rect` por `fill_rect` na moldura da barra. O que acontece com
   o enchimento por cima?
2. Use `//` (divisão inteira) vs `/` (divisão normal) em `largura_cheia`.
   Por que `//` é melhor aqui? (Dica: `fill_rect` espera número inteiro.)
3. Mude o passo do `range` para `1` em vez de `5`. A barra fica mais suave?

## Desafio

**Principal:** desenhe uma **moldura ao redor da tela inteira**
(`rect(0, 0, 128, 64, 1)`) e escreva um texto centralizado dentro dela.

**Bônus:** faça a barra de progresso **descer** depois de cheia (de 100%
a 0%), como um indicador que esvazia — útil para mostrar tempo restante
em projetos futuros.

## Resumo

- O OLED desenha **pixels, linhas e retângulos**; cor `1`=aceso, `0`=apagado.
- `rect` faz **contorno**; `fill_rect` faz **preenchido**.
- Uma **barra de progresso** = moldura `rect` + enchimento `fill_rect`
  proporcional ao percentual.
- Sempre `fill(0)` para limpar e `show()` para atualizar.

---

[⬅️ Aula 5](aula05-oled-text.md) · [🏠 Início](../index.md)

> 🔜 **Próximos passos (mini-curso futuro):** integrar o display com
> **comunicação UART** e fazer **dois ESP32 conversando** — o display
> mostrará as mensagens recebidas.
