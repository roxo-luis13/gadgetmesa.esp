# Gadget mesa — Relógios inteligentes ESP32 + TFT

Projeto pessoal de gadgets de mesa baseados em **ESP32 DevKit** com **display TFT SPI 240×240**, programados em **C++ (Arduino)**. Existem duas builds relacionadas:

1. **Relógio WiFi** (mais recente, principal) — multi-tela, saudações, lembrete de água, clima, controle pelo celular.
2. **Relógio despertador/timer** (anterior) — botões físicos, 3 alarmes persistentes, timer regressivo, alerta piscante.

---

## Hardware

| Item | Valor |
|---|---|
| MCU | ESP32 DevKit |
| Display | TFT SPI **240×240**, orientação retrato |
| Biblioteca do display | TFT_eSPI |

### Ligação TFT → ESP32

| TFT | ESP32 |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SCL (SCLK) | GPIO18 |
| SDA (MOSI) | GPIO23 |
| RST | GPIO4 |
| DC | GPIO2 |
| CS | GPIO15 |
| BL | GPIO21 |

Configuração equivalente no `User_Setup.h` do TFT_eSPI:

```cpp
#define TFT_MOSI 23
#define TFT_SCLK 18
#define TFT_CS   15
#define TFT_DC    2
#define TFT_RST   4
#define TFT_BL   21
#define TFT_WIDTH  240
#define TFT_HEIGHT 240
```

> ⚠️ A tela é **240×240**. No início foi assumido 320px e todo o layout teve que ser refeito. Todo desenho deve caber em 240×240.

### Botões (somente na build despertador/timer)

| Botão | GPIO | Função |
|---|---|---|
| A | GPIO34 | SELECT / OK |
| B | GPIO35 | NEXT / UP |

> ⚠️ **Verificar:** GPIO34 e GPIO35 são *input-only* no ESP32 e **não têm pull-up/pull-down internos**. O código original usava `INPUT_PULLDOWN` com botões ligados ao GND, o que não funciona de forma confiável. Corrigir com resistor externo (ex.: pull-up 10k para 3.3V + botão para GND, lendo LOW como pressionado) ou mover os botões para GPIOs com pull-up interno (ex.: 32/33 com `INPUT_PULLUP`).

---

## Rede, hora e clima

- **WiFi:** SSID `Roxo` (senha fica fora do repositório — ver "Segredos" abaixo)
- **NTP:** `pool.ntp.org`, fuso **UTC-3**, sem horário de verão (`gmtOffset = -10800`, `daylightOffset = 0`)
- **Clima:** API **Open-Meteo** (gratuita, sem chave)
  - Campinas/SP: `latitude=-22.9056`, `longitude=-47.0608`
  - Atualização a cada **10 minutos**

### Segredos

Não versionar a senha do WiFi. Sugestão: arquivo `secrets.h` no `.gitignore`:

```cpp
// secrets.h
#define WIFI_SSID "Roxo"
#define WIFI_PASS "********"
```

---

## Build 1 — Relógio WiFi (principal)

Arquivo gerado anteriormente: `relogio_esp32.ino`

### Tela 1 — Relógio
- Saudação no topo conforme o horário:
  - **"Bom dia!"** 05:00–11:59
  - **"Boa tarde!"** 12:00–17:59
  - **"Boa noite!"** 18:00–04:59
- Hora, dia da semana e data em linhas separadas, tudo **centralizado**
- Lembrete **"Beba Água!"** piscando:
  - dispara **a cada 5 minutos**
  - dura **7 segundos**
  - ciclo de pisca de **250 ms**
  - fonte tamanho **4**
- (O ícone de gota d'água foi **removido**.)

### Tela 2 — Teste
- Texto "Teste de tela 2" (placeholder para conteúdo futuro)

### Tela 3 — Clima
- Dados do Open-Meteo para Campinas, atualizados a cada 10 min
- Layout ajustado para caber em 240×240

### Navegação
- Indicadores de página (pontinhos) mostrando a tela ativa
- **Sem botões físicos** — foram removidos
- Troca de tela via **WebServer na porta 80**: página HTML responsiva para celular, acessada pelo IP do ESP32

---

## Build 2 — Relógio despertador/timer (anterior)

- Máquina de estados com **4 telas**: relógio, menu, configurar alarme, configurar timer
- **3 alarmes** configuráveis, salvos em flash com a biblioteca **Preferences** (NVS) — persistem após desligar
- **Timer regressivo**: iniciar/pausar com o Botão B a partir da tela do relógio
- **Alerta piscante** vermelho/preto quando alarme ou timer dispara; só para ao pressionar qualquer botão
- Esquema elétrico em SVG foi feito na época (TFT + 2 botões)

---

## Bibliotecas

| Biblioteca | Uso | Observação |
|---|---|---|
| TFT_eSPI | Display | Configurar `User_Setup.h` com os pinos acima |
| WiFi | Conexão | Core ESP32 |
| WebServer | Controle de telas pelo celular | Core ESP32 |
| HTTPClient | Chamadas ao Open-Meteo | Core ESP32 |
| ArduinoJson | Parse da resposta de clima | **Instalar manualmente** pelo Library Manager |
| Preferences | Persistir alarmes (NVS) | Core ESP32 |
| time.h | NTP | Core ESP32 |

---

## Próximos passos possíveis

- Buzzer/alerta sonoro (oferecido para a build despertador)
- Dar conteúdo real à Tela 2
- Unificar as duas builds (alarmes + timer controlados pela página web, sem botões)
- Refinamentos de layout e novas telas

---

## Painel web (simulador)

- Fonte: `docs/index.html`: só o preview da tela 240×240 e as configurações (telas, Pomodoro, Picture-in-Picture).
- Telas no preview: Relógio, Bolsa B3, Clima e Pomodoro (foco 25 / pausa curta 5 / pausa longa 15 / 4 ciclos, configuráveis; no fim pisca vermelho/preto até tocar, como a build despertador; ao confirmar o fim do foco, a pausa começa sozinha; só entra na troca automática de telas quando está em uso). A hora aparece em todas as telas.
- Publicado no GitHub Pages a partir da branch `gh-pages` (arquivo `index.html` na raiz): https://roxo-luis13.github.io/gadgetmesa.esp/
- Ao mudar `docs/index.html`, copiar para `index.html` na `gh-pages` e dar push para atualizar o site.

---

## Como trabalhar neste projeto

- Comunicação em **português**, pedidos curtos e diretos.
- **Antes de mudar layout:** mostrar um preview visual (HTML/CSS simulando a tela 240×240) e aguardar aprovação; só então gerar o código.
- Preferência por **arquivo `.ino` completo** em vez de trechos/diffs.
- Valores numéricos de tempo/tamanho informados pelo Luis devem ser usados **exatamente** como pedidos.
- Iterações pequenas e frequentes.
- Sempre confirmar que tudo cabe em 240×240 antes de entregar.

### Sugestão de estrutura no repositório

```
gadget-mesa/
├── CLAUDE.md
├── .gitignore            # inclui secrets.h
├── relogio_wifi/
│   ├── relogio_wifi.ino
│   └── secrets.h         # não versionado
├── relogio_alarme/
│   └── relogio_alarme.ino
├── docs/
│   └── esquema.svg
└── tft_setup/
    └── User_Setup.h
```

Para compilar fora da IDE: `arduino-cli` com a placa `esp32:esp32:esp32`, ou migrar para PlatformIO (`platformio.ini` com `framework = arduino`, `board = esp32dev`, `lib_deps` para TFT_eSPI e ArduinoJson, e `build_flags` com os `-D` dos pinos do TFT).
