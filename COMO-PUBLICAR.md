# Como publicar no GitHub Pages

Guia rápido para hospedar este mini-curso, igual aos anteriores.

## 1. Criar o repositório

1. No GitHub, crie um repositório **público** chamado
   `minicurso_03-embarcados`.
2. **Não** marque "Add a README" (já temos os arquivos).

## 2. Enviar os arquivos (pelo navegador, arrastando)

Na página do repositório, use **Add file → Upload files** e arraste,
mantendo a estrutura de pastas:

```
minicurso_03-embarcados/
├── README.md
├── index.md
├── _config.yml
├── COMO-PUBLICAR.md
├── aulas/
│   ├── aula01-i2c-scan.md
│   ├── aula02-lcd-hello.md
│   ├── aula03-lcd-dinamico.md
│   ├── aula04-lcd-custom.md
│   ├── aula05-oled-text.md
│   └── aula06-oled-graficos.md
└── assets/
    ├── banner.png
    ├── wokwi-links.md
    └── libs/
        ├── lcd_api.py
        ├── i2c_lcd.py
        └── ssd1306.py
```

> 💡 O GitHub cria pastas automaticamente quando você arrasta arquivos
> com `/` no nome do caminho. Se preferir, suba pasta por pasta.

## 3. Ativar o GitHub Pages

1. No repositório: **Settings → Pages**.
2. Em **Source**, escolha **Deploy from a branch**.
3. Branch: **main** · pasta: **/ (root)** · **Save**.
4. Aguarde ~1 min. O site sai em:
   `https://rogeriomb-hub.github.io/minicurso_03-embarcados/`

## 4. Conferir

- Abra a URL acima: deve aparecer o tema **Cayman** com o banner e a
  tabela de aulas.
- Teste os links das aulas e a navegação ⬅️ ➡️ no rodapé de cada uma.

## 5. Distribuir no Google Sites (opcional)

- Use o **banner** (`assets/banner.png`, 1360×480) como cabeçalho.
- Incorpore o link do GitHub Pages ou cada aula como botão.
