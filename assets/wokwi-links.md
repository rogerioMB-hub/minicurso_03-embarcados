# Links e templates de circuito — Wokwi

> Coleção dos `diagram.json` usados no curso, para copiar rapidamente.
> ⚠️ **Validar antes de publicar:** confira no Wokwi se todas as conexões
> aparecem desenhadas (a importação às vezes omite fios).

## Como montar um projeto

1. Acesse <https://wokwi.com/projects/new/micropython-esp32>.
2. Cole o `main.py` da aula.
3. Clique na aba **diagram.json** e cole o circuito correspondente.
4. Para bibliotecas: botão **+** → *New File* → cole `lcd_api.py`,
   `i2c_lcd.py` ou `ssd1306.py` (de `assets/libs/`).
5. ▶ para simular.

---

## Template A — LCD 16x2 (Aulas 1 a 4)

Pinos: SDA=21, SCL=22, VCC=5V, GND=GND. Endereço I2C: `0x27`.

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

---

## Template B — OLED SSD1306 (Aulas 5 e 6)

Pinos: DATA(SDA)=21, CLK(SCL)=22, VCC=3V3, GND=GND. Endereço I2C: `0x3C`.

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

---

## Bibliotecas necessárias

| Display | Arquivos a adicionar no Wokwi |
|---------|-------------------------------|
| LCD 16x2 | `lcd_api.py` + `i2c_lcd.py` |
| OLED SSD1306 | `ssd1306.py` |

Todos estão em `assets/libs/` deste repositório.
