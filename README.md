

---

# Merlot

Sistema de **monitoramento de adega** com **Arduino**, que acompanha **temperatura**, **luminosidade** e **umidade** do ambiente, exibe os dados em **LCD 16x2**, e aciona **LEDs** e **buzzer** conforme faixas ideais para armazenamento de vinhos. O projeto foi desenvolvido no contexto do caso **Vinheria Agnello** (Checkpoint 01 – Engenharia de Software / Edge Computing & Computer Systems).

---

## Índice
- Biblioteca
- Contexto & Motivação
- Requisitos do Desafio
- Funcionalidades
- Arquitetura e Componentes
- Pinagem (Arduino Uno)
- Instalação e Execução
- Parâmetros Ajustáveis
- Lógica de Operação
- Cálculos e Conversões
- Como Reproduzir no Tinkercad
- Estrutura do Repositório
- Critérios de Avaliação
- Autores
- Licença

---
## Biblioteca
- include <LiquidCrystal.h> // Controle do display LCD
- include <math.h>          // Funções matemáticas (pow, isfinite)
- include <stdio.h>         // Funções de I/O (sprintf)
- include <string.h>        // Manipulação de strings (strlen, strcat)

---


## Contexto & Motivação
A qualidade do vinho é influenciada diretamente por **luminosidade**, **temperatura** e **umidade**. Em linhas gerais:
- **Luminosidade**: manter **baixa** (penumbra), evitando UV.
- **Temperatura**: ideal próxima de **13 °C** e **sem variações > 3 °C**.
- **Umidade**: **~70%** (tolerância ~**60%–80%**), evitando ressecamento da rolha e mofo.

Este projeto busca **monitorar** essas grandezas e **alertar** quando saem das faixas ideais, contribuindo para a **conservação** do acervo da adega.

---

## Requisitos do Desafio
1. Capturar **luminosidade** com **LDR** via ADC do Arduino e normalizar com `map()` para **0–100%**.
2. **LEDs**: **verde** (OK), **amarelo** (alerta) e **vermelho** (alarme/problema).
3. Em **alerta de luminosidade**, o **buzzer** deve soar **3 s** e silenciar **3 s**, repetindo enquanto o alerta persistir.
4. **LCD 16x2** com **logo/mensagem de boas‑vindas** (obrigatório).
5. Entregas: **projeto no Tinkercad**, **código comentado**, **README**, **vídeo público (≤ 3 min)**.

---

## Funcionalidades
- Leitura contínua de **temperatura** (TMP36), **luminosidade** (LDR) e **umidade** (simulada com potenciômetro).
- **LCD 16x2** com **animação/logo** e **mensagem de boas‑vindas**; telas rotativas a cada **5 s**:
  - **TEMPERATURA** (°C + status: BAIXA/IDEAL/ALTA)
  - **LUMINOSIDADE** (% + status: OK/ALT/ALM)
  - **UMIDADE** (% + status: OK/ALT/ALM)
- **LEDs** indicativos e **buzzer** em estado de **alerta de luminosidade** (3 s ON / 3 s OFF).

---

## Arquitetura e Componentes
**Hardware**
- Arduino Uno
- LCD 16x2 (modo 4 bits)
- LDR + resistor **10 kΩ** (divisor de tensão)
- Sensor de temperatura **TMP36**
- **Potenciômetro** (simulação de umidade)
- LEDs: **verde**, **amarelo**, **vermelho**
- **Buzzer**
- Protoboard e jumpers

**Software**
- Arduino IDE 1.8.x ou 2.x

---

## Pinagem (Arduino Uno)
> Ajuste conforme sua montagem. Estes são os pinos utilizados no sketch do projeto.

**LCD (modo 4 bits – RS,E,D4,D5,D6,D7):** `10, 9, 8, 7, 6, 5, 4`

**Sensores:**
- **LDR:** `A0` (com **10 kΩ** de referência)
- **TMP36 (temperatura):** `A1`
- **Umidade (potenciômetro):** `A2`

**Atuadores:**
- **LED verde:** `D13`
- **LED amarelo:** `D12`
- **LED vermelho:** `D11`
- **Buzzer:** `D3`

---

## Instalação e Execução
1. **Clone** este repositório:
   ```bash
   git clone https://github.com/<seu-usuario>/merlot.git
   cd merlot
   ```
2. Abra o projeto na **Arduino IDE** e confirme a biblioteca `LiquidCrystal`.
3. Conecte os componentes conforme a **pinagem** e **carregue** o sketch no Arduino.
4. Ao iniciar, o LCD exibirá a **animação/logo** e a **mensagem de boas‑vindas**. Em seguida, as telas **alternam a cada 5 s**.

---

## Parâmetros Ajustáveis
Os principais limites e margens estão definidos no código como `const` e podem ser ajustados conforme sua calibração/ambiente:

```cpp
// Faixas de luminosidade (Lux)
const float RECOMENDADO_MIN = 100.0;   // mínimo ideal
const float RECOMENDADO_MAX = 200.0;   // máximo ideal
const float ALERTA_MARGIN   = 20.0;    // margem p/ alerta (±)

// Umidade (%)
const int UMIDADE_IDEAL_MIN    = 60;   // mínimo ideal
const int UMIDADE_IDEAL_MAX    = 80;   // máximo ideal
const int UMIDADE_ALERTA_MARGIN = 10;  // margem p/ alerta (±)

// Intervalo de troca de tela (ms)
const unsigned long intervaloDisplay = 5000; // 5 s
```

---

## Lógica de Operação
**LCD (rotação 5 s):**
- **Temperatura**: mostra `°C` + status:
  - **BAIXA** se `< 12 °C`
  - **IDEAL** se `12–14 °C`
  - **ALTA** se `> 14 °C`
- **Luminosidade**: mostra `%` + status:
  - **OK** dentro da faixa ideal (100–200 Lux)
  - **ALT (alerta)** dentro da margem (80–100 Lux ou 200–220 Lux)
  - **ALM (alarme)** abaixo de 80 Lux ou acima de 220 Lux
- **Umidade**: mostra `%` + status com base em `60–80%` e margem de `±10%`.

**LEDs & Buzzer (baseados em luminosidade):**
- **OK** → LED **verde** ligado; **buzzer OFF**
- **ALT** → LED **amarelo**; **buzzer ON 3 s / OFF 3 s** (enquanto persistir)
- **ALM** → LED **vermelho**; **buzzer OFF** (conforme a lógica atual do sketch)

> Observação: a mesma lógica pode ser estendida para **temperatura** e **umidade**.

---

## Cálculos e Conversões

### Luminosidade (LDR → Lux → %)
```cpp
// Conversão ADC -> tensão
float Vout = adc * (VCC / 1023.0);  // VCC = 5.0 V

// Resistência do LDR (divisor com R_FIXED = 10kΩ)
float R_LDR = R_FIXED * (VCC / Vout - 1.0);

// Curva empírica para Lux
const float A = 600000.0;
const float B = -1.25;
float lux = A * pow(R_LDR, B);

// Exibição em % (0–100) a partir de 0–1000 Lux
int luminosidade_pct = map((int)lux, 0, 1000, 0, 100);
```

### Temperatura (TMP36)
```cpp
float tensao = analogRead(A1) * (5.0 / 1023.0);
float temperaturaC = (tensao - 0.5) * 100.0; // °C
```

### Umidade (simulada com potenciômetro)
```cpp
int umidadePct = map(analogRead(A2), 0, 1023, 0, 100); // %
```

---

## Como Reproduzir no Tinkercad
1. Monte o circuito: Arduino Uno, LCD 16x2 (4 bits), LDR+10 kΩ (divisor), TMP36 em `A1`, potenciômetro em `A2`, LEDs (D13/D12/D11) e buzzer (D3).
2. Cole o **mesmo sketch** do diretório `src/`.
3. Varie a luz no LDR (lanterna/cobrir com a mão) e observe **OK/ALT/ALM** no LCD e LEDs. O **buzzer** deve alternar **3 s ON / 3 s OFF** em **ALT**.

> Dica: na simulação, ajuste o nível de iluminação do LDR para testar as transições.

---

## Estrutura do Repositório
```
merlot/
├─ README.md
├─ /src
│  └─ merlot.ino
├─ /hardware
│  ├─ esquema-fritzing.fzz
│  └─ fotos-montagem/
├─ /docs
│  ├─ slides.pdf
│  └─ requisitos.pdf
└─ /media
   ├─ demo.mp4
   └─ thumbnails/
```

---

## Autoras
- **Giovanna Oliveira Ferreira Dias** — RM **566647**
- **Maria Laura Pereira Druzeic** — RM **566634**
- **Marianne Mukai Nishikawa** — RM **568001**

---

## Licença
Este projeto está licenciado sob os termos da Licença **MIT.**

---

