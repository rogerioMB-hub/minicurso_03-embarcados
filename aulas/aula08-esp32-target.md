---
layout: default
title: "Aula 08 — ESP32 como target I2C"
---

# Aula 08 — ESP32 como target I2C

> **Seção 2 — O ESP32 como periférico.** Na Aula 7 você descobriu que um periférico é **um endereço + um mapa de registradores**, e projetou no papel o mapa de um periférico com LED RGB. Agora vamos **construir esse periférico de verdade**, usando um segundo ESP32.

⏱️ Tempo estimado: 30 minutos · 🔧 **Bancada** (duas placas ESP32-C3 SuperMini)

> ⚠️ **Por que não no Wokwi?** O simulador ainda não implementa o ESP32 no papel de target I2C. O código abaixo funciona no hardware real (MicroPython 1.25 ou mais novo).

---

## 🎯 Objetivos

Ao final desta aula, você será capaz de:

1. **Configurar** um ESP32 como target I2C com `I2CTarget`, no endereço `0x42`.
2. **Implementar** um mapa de registradores usando o `mem` como memória compartilhada.
3. **Enviar** comandos e dados do controller com `writeto_mem()`.
4. **Ler** o estado do target com `readfrom_mem()`, fazendo a comunicação nos dois sentidos.
5. **Explicar** por que a ordem das escritas importa e por que o target "consome" o comando.

---

## 💡 Conceito

### Do papel para o código

Este é o mapa "oficial" do nosso periférico. **Compare com o que você projetou no desafio da Aula 7:** as ideias são as mesmas, mesmo que os números sejam outros.

![Mapa de registradores do periférico 0x42](../assets/fig09-mapa-registradores.png)

### O `mem` é o banco de registradores

No target, criamos um `bytearray` e o entregamos ao `I2CTarget`:

```python
mem = bytearray(8)
t = I2CTarget(0, addr=0x42, mem=mem, scl=Pin(7), sda=Pin(6))
```

A partir daí, **o hardware I2C escreve no `mem` sozinho**, em segundo plano. Quando o controller executa `writeto_mem(0x42, 1, bytes([255, 0, 128]))`, o target recebe os bytes e os guarda em `mem[1]`, `mem[2]` e `mem[3]`, sem que o programa do target precise fazer nada. O laço principal só precisa **olhar** o `mem` de tempos em tempos.

> 📏 **Regra do tamanho:** o `mem` deve ser **maior** que o maior bloco que o controller vai enviar, contando o byte do registrador. Com `bytearray(8)` temos folga para os registradores 0 a 7.

### Comando consumido

Depois de executar um comando, o target escreve `mem[0] = 0`. Isso diz: *"já fiz, pode mandar o próximo"*. Sem essa linha, o comando continua parado no registrador e o target não consegue distinguir um comando novo de um antigo.

### A ordem importa

Para pedir uma cor fixa, o controller envia **duas** mensagens: os dados e o comando. Se o comando chegasse primeiro, o target poderia agir **antes** de a cor nova chegar, e acenderia a cor antiga. Por isso: **primeiro os dados, depois o comando.**

![Sequência de mensagens para a cor fixa](../assets/fig10-sequencia-cor-fixa.png)

### Ida e volta

O registrador 4 (STATUS) é escrito pelo **target**. O controller o lê com:

```python
estado = i2c.readfrom_mem(0x42, 4, 1)[0]   # lê 1 byte a partir do registrador 4
```

Assim o controller pode **conferir** se o periférico fez o que foi pedido.

---

## 🔌 Circuito (bancada)

| Controller (C3) | Target (C3) | Observação |
|---|---|---|
| GPIO6 (SDA) | GPIO6 (SDA) | + resistor de 4,7 kΩ para 3V3 |
| GPIO7 (SCL) | GPIO7 (SCL) | + resistor de 4,7 kΩ para 3V3 |
| GND | GND | **obrigatório** |

No **target**, o LED RGB de catodo comum:

| Target | LED RGB |
|---|---|
| GPIO0 | R (via resistor de 220 Ω) |
| GPIO1 | G (via resistor de 220 Ω) |
| GPIO3 | B (via resistor de 220 Ω) |
| GND | catodo (perna mais longa) |

> 📌 **Pinos evitados:** GPIO2, GPIO8 e GPIO9 são pinos de *strapping* do C3 (definem o modo de boot). O GPIO8 também é o LED da placa.

<!-- FOTO DA MONTAGEM: assets/fig11-montagem-bancada.jpg (a fornecer) -->

### ✅ Checklist de montagem

- [ ] GND das duas placas ligados entre si
- [ ] SDA com SDA, SCL com SCL (não cruzar!)
- [ ] Um resistor de 4,7 kΩ do SDA para 3V3 e outro do SCL para 3V3 (só de um lado)
- [ ] LED com a perna longa (catodo) no GND e um resistor em cada cor
- [ ] **Target** ligado e rodando **antes** do controller

---

## 💻 Código

### Target (`main.py` da placa com o LED)

```python
# Aula 08 — TARGET: periférico I2C com LED RGB no endereço 0x42
from machine import I2CTarget, Pin, PWM
import time

INVERTIDO = False   # catodo comum (anodo comum: True)

# --- Mapa de registradores ---
REG_CMD    = 0      # controller escreve: 1 arco-íris, 2 desligar, 3 cor fixa
REG_R      = 1      # controller escreve: vermelho 0..255
REG_G      = 2      # controller escreve: verde 0..255
REG_B      = 3      # controller escreve: azul 0..255
REG_STATUS = 4      # TARGET escreve: 0 desligado, 1 arco-íris, 2 cor fixa

pr = PWM(Pin(0), freq=1000, duty_u16=0)
pg = PWM(Pin(1), freq=1000, duty_u16=0)
pb = PWM(Pin(3), freq=1000, duty_u16=0)

def cor(r, g, b):          # valores de 0.0 a 1.0
    for p, v in ((pr, r), (pg, g), (pb, b)):
        d = int(v * 65535)
        p.duty_u16(65535 - d if INVERTIDO else d)

def matiz(h):              # h de 0.0 a 1.0 -> cor do arco-íris
    i = int(h * 6)
    f = h * 6 - i
    q = 1 - f
    return [(1, f, 0), (q, 1, 0), (0, 1, f),
            (0, q, 1), (f, 0, 1), (1, 0, q)][i % 6]

mem = bytearray(8)         # 8 registradores (0 a 7)
t = I2CTarget(0, addr=0x42, mem=mem, scl=Pin(7), sda=Pin(6))
print("target pronto, aguardando")

cor(0, 0, 0)
modo = 0                   # 0 desligado, 1 arco-íris, 2 cor fixa
h = 0.0

while True:
    cmd = mem[REG_CMD]

    if cmd == 1:                        # ligar arco-íris
        print("recebido: arco-íris")
        modo = 1
        h = 0.0
        mem[REG_CMD] = 0                # comando consumido

    elif cmd == 2:                      # desligar
        print("recebido: desligar")
        for _ in range(3):
            cor(1, 0, 0)
            time.sleep_ms(300)
            cor(0, 0, 0)
            time.sleep_ms(300)
        modo = 0
        mem[REG_CMD] = 0

    elif cmd == 3:                      # cor fixa
        r, g, b = mem[REG_R], mem[REG_G], mem[REG_B]
        print("recebido: cor fixa", r, g, b)
        cor(r / 255, g / 255, b / 255)  # 0..255 -> 0.0..1.0
        modo = 2
        mem[REG_CMD] = 0

    mem[REG_STATUS] = modo              # o controller pode ler este registrador

    if modo == 1:
        cor(*matiz(h))
        h = (h + 0.003) % 1.0           # volta completa de cor em ~6,7 s
    time.sleep_ms(20)
```

### Controller (`main.py` da outra placa)

```python
# Aula 08 — CONTROLLER: comanda o periférico 0x42
from machine import I2C, Pin
import time

ENDERECO = 0x42
REG_CMD, REG_R, REG_STATUS = 0, 1, 4
NOMES = {0: "desligado", 1: "arco-íris", 2: "cor fixa"}

i2c = I2C(0, scl=Pin(7), sda=Pin(6), freq=100000)

achados = i2c.scan()
print("scan:", achados)                 # deve aparecer [66], que é 0x42
if ENDERECO not in achados:
    raise SystemExit("target 0x42 não encontrado: confira fios, GND e pull-ups")

def contagem(texto, segundos):
    for s in range(segundos, 0, -1):
        print(texto, s)
        time.sleep(1)

def status():
    estado = i2c.readfrom_mem(ENDERECO, REG_STATUS, 1)[0]
    print("  status do target:", estado, "=", NOMES.get(estado, "?"))

contagem("arco-íris em", 3)
i2c.writeto_mem(ENDERECO, REG_CMD, b'\x01')
print("sinal de arco-íris enviado")
time.sleep_ms(100)
status()

contagem("cor fixa em", 10)
i2c.writeto_mem(ENDERECO, REG_R, bytes([255, 0, 128]))   # 1º: os dados (R, G, B)
i2c.writeto_mem(ENDERECO, REG_CMD, b'\x03')              # 2º: o comando
print("sinal de cor fixa enviado")
time.sleep_ms(100)
status()

contagem("desligando em", 5)
i2c.writeto_mem(ENDERECO, REG_CMD, b'\x02')
print("sinal de desligar enviado")
time.sleep(2)                           # espera o pisca terminar
status()
```

### Como ler este código

- **`mem[REG_CMD] = 0`** aparece em todo comando: é o "comando consumido".
- **`mem[REG_STATUS] = modo`** roda em toda volta do laço: o registrador 4 sempre mostra o estado atual.
- No controller, **`writeto_mem(ENDERECO, REG_R, bytes([255, 0, 128]))`** escreve 3 bytes de uma vez, a partir do registrador 1: eles caem em `mem[1]`, `mem[2]` e `mem[3]`.
- O **`time.sleep_ms(100)`** antes de cada `status()` dá tempo ao laço do target (que dorme 20 ms por volta) de perceber o comando e atualizar o STATUS.

### O que esperar no Shell do controller

```
scan: [66]
arco-íris em 3 ...
sinal de arco-íris enviado
  status do target: 1 = arco-íris
cor fixa em 10 ...
sinal de cor fixa enviado
  status do target: 2 = cor fixa
desligando em 5 ...
sinal de desligar enviado
  status do target: 0 = desligado
```

E no LED: arco-íris por 10 s, depois um rosa-magenta fixo (`255, 0, 128`), depois 3 piscadas vermelhas e apaga.

---

## 🔬 Experimento

1. Com tudo rodando, desconecte o fio de GND entre as placas e reinicie o controller. O que o `scan()` mostra?
2. No controller, envie o comando `1` duas vezes seguidas, com 3 s de intervalo. O arco-íris **recomeça** do vermelho? Por quê? *Dica:* procure o `mem[REG_CMD] = 0`.
3. Comente a linha `mem[REG_CMD] = 0` do comando 1 no target e repita o teste 2. O que muda?
4. No controller, inverta a ordem: envie o comando `3` **antes** do RGB, mude a cor para `0, 255, 0` e coloque um `time.sleep_ms(500)` entre as duas escritas. Qual cor aparece primeiro no LED? Agora tire a pausa: o erro some? *(Ele fica raro, mas não impossível: é o tipo de defeito que aparece "às vezes" e dá muito trabalho para achar.)*
5. Troque o `time.sleep(2)` antes do último `status()` por `time.sleep_ms(100)`. Que estado aparece? Por quê? *Dica:* o que o target está fazendo durante as piscadas?

---

## 🏆 Desafio

### Principal: registrador de brilho

Use o registrador 5 como **BRILHO**, de 0 a 100 (%). No target, multiplique as cores pelo brilho antes de chamar `cor()`, tanto no arco-íris quanto na cor fixa. No controller, envie `writeto_mem(0x42, 5, bytes([30]))` e veja o LED ficar mais fraco.

*Pergunta para pensar:* o que deve acontecer se o controller nunca escrever no registrador 5? Que valor inicial faz sentido?

### Bônus: sem ficar olhando o `mem`

Hoje o target confere `mem[0]` a cada 20 ms (isso se chama *polling*). O `I2CTarget` também pode **avisar** o programa quando uma escrita termina, chamando uma função sua (*callback* de interrupção). Pesquise o método `irq()` do `I2CTarget` na documentação do MicroPython e tente usá-lo.

---

## 🧯 Se não funcionar

| Sintoma | Causa provável |
|---|---|
| `scan()` mostra `[]` | GND não ligado, faltam pull-ups, ou SDA/SCL trocados |
| `scan()` mostra `[66]` mas o LED não muda | O target não está rodando o `main.py`, ou o LED está invertido (teste `INVERTIDO = True`) |
| `OSError: [Errno 116] ETIMEDOUT` | O target travou ou foi desligado durante a comunicação |
| Funciona às vezes | Fios longos ou mal encaixados; pull-ups internos sozinhos não bastam |
| `ImportError: I2CTarget` | Firmware antigo no target: atualize para MicroPython 1.25 ou mais novo |

---

## 📌 Resumo

| Conceito | No código |
|---|---|
| Criar o periférico | `I2CTarget(0, addr=0x42, mem=mem, ...)` |
| Registradores | posições do `mem` (`mem[0]` a `mem[7]`) |
| Controller escreve | `writeto_mem(0x42, registrador, dados)` |
| Controller lê | `readfrom_mem(0x42, registrador, quantidade)` |
| Comando consumido | o target faz `mem[0] = 0` depois de agir |
| Ordem das escritas | **dados primeiro, comando depois** |

➡️ **Próxima aula:** vamos juntar tudo em um **projeto integrador**: o controller comanda o LED RGB e mostra o status do periférico no **OLED**.

---

[← Aula 07](aula07-anatomia-periferico-i2c.md) · [Início](../index.md) · Aula 09 *(em breve)*
