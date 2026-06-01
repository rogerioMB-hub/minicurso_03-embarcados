---
layout: default
title: "Aula 1 — O que é I2C"
---

# Aula 1 — O que é I2C e como varrer o barramento

⏱️ **Duração:** ~30 min · 🎯 **Nível:** iniciante

## Objetivos

Ao final desta aula você será capaz de:

- Explicar o que é o barramento **I2C** e para que serve.
- Identificar as linhas **SDA** e **SCL**.
- Entender o conceito de **endereço** de um dispositivo I2C.
- Usar `i2c.scan()` para descobrir quem está conectado ao barramento.

## Conceito

Até agora, cada dispositivo que ligamos ao ESP32 usava um ou mais pinos
dedicados. Imagine ligar um display, um sensor de temperatura e um relógio
ao mesmo tempo — faltariam pinos rapidamente.

O **I2C** (lê-se "i-dois-cê" ou "ai-squared-cee") resolve isso. É um
**barramento serial** em que **vários dispositivos compartilham apenas
dois fios**:

| Linha | Nome | Função |
|-------|------|--------|
| **SDA** | *Serial Data* | leva os dados (vai e volta) |
| **SCL** | *Serial Clock* | leva o pulso de relógio (sincroniza) |

Como todos compartilham os mesmos dois fios, cada dispositivo precisa de um
**endereço** único — um número (geralmente em hexadecimal, como `0x27`) que
funciona como o "RG" dele no barramento. O ESP32 (o **mestre**) chama o
dispositivo pelo endereço; só aquele dispositivo responde.

> 💡 **Pull-ups:** as linhas SDA e SCL precisam de resistores de
> *pull-up* (puxam o sinal para o nível alto). No ESP32 e no Wokwi isso
> costuma já estar resolvido internamente, então **não vamos nos preocupar
> com isso na simulação** — mas saiba que no hardware real eles existem.

## Circuito

Nesta primeira aula usaremos um LCD 16x2 só para **ter algo no barramento**
e enxergá-lo no scan. Pinos no ESP32:

- **SDA → GPIO 21**
- **SCL → GPIO 22**
- **VCC → 5V** e **GND → GND**

### diagram.json

> ⚠️ **Validar antes de publicar** (conferir se as 4 conexões aparecem no Wokwi).

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
# Aula 1 - Varredura (scan) do barramento I2C
# ESP32 + Wokwi - MicroPython
# Pico: troque os pinos abaixo por sda=Pin(0), scl=Pin(1)

from machine import Pin, SoftI2C
from time import sleep

# Cria o barramento I2C por software (funciona bem no Wokwi).
# SDA no GPIO 21, SCL no GPIO 22, velocidade de 100 kHz.
i2c = SoftI2C(sda=Pin(21), scl=Pin(22), freq=100000)
# Pico: i2c = SoftI2C(sda=Pin(0), scl=Pin(1), freq=100000)

print("Procurando dispositivos no barramento I2C...")
sleep(1)

# scan() devolve uma LISTA com os endereços encontrados.
dispositivos = i2c.scan()

if len(dispositivos) == 0:
    print("Nenhum dispositivo encontrado. Confira as ligacoes SDA/SCL.")
else:
    print("Encontrei", len(dispositivos), "dispositivo(s):")
    for endereco in dispositivos:
        # hex() mostra o endereco em hexadecimal, como 0x27.
        print("  ->", hex(endereco), "(decimal:", endereco, ")")
```

Ao rodar, o terminal deve mostrar algo como:

```
Procurando dispositivos no barramento I2C...
Encontrei 1 dispositivo(s):
  -> 0x27 (decimal: 39)
```

Esse `0x27` é o endereço do **PCF8574**, o chip que fica nas costas do
LCD (o tal "backpack") e converte I2C nos sinais do display. Guarde esse
número — vamos usá-lo na próxima aula.

## Experimento

1. Rode o código. Qual endereço apareceu?
2. No `diagram.json`, troque o pino do SDA de `esp:21` para `esp:19`
   **sem** mudar o código. O que acontece no scan? Por quê?
3. Quantos fios o I2C usa para os dados, independentemente de termos
   1 ou 5 dispositivos ligados?

## Desafio

**Principal:** transforme a saída do scan em uma frase amigável, por
exemplo: `"Dispositivo no endereco 0x27 esta pronto!"` para cada item
encontrado.

**Bônus:** faça o programa varrer o barramento **a cada 3 segundos**
dentro de um laço `while True`, imprimindo a hora simulada com
`time.ticks_ms()`. Assim você teria um "monitor" de barramento.

## Resumo

- I2C usa **2 fios**: **SDA** (dados) e **SCL** (clock).
- Cada dispositivo tem um **endereço** único (ex.: `0x27`).
- `i2c.scan()` devolve a **lista** de endereços presentes.
- O LCD 16x2 aparece pelo seu backpack **PCF8574**.

---

[⬅️ Início](../index.md) · [➡️ Aula 2 — LCD: primeiros caracteres](aula02-lcd-hello.md)
