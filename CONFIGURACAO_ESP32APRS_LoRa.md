# Configuração ESP32APRS_LoRa para Receber Dados da Sonda RS41-NFW

## 📋 Índice
1. [Visão Geral](#visão-geral)
2. [Configurações da Sonda RS41-NFW](#configurações-da-sonda-rs41-nfw)
3. [Configuração do ESP32APRS_LoRa](#configuração-do-esp32aprs-lora)
4. [Protocolos Suportados](#protocolos-suportados)
5. [Limitações Importantes](#limitações-importantes)
6. [Configuração Passo a Passo](#configuração-passo-a-passo)
7. [Solução de Problemas](#solução-de-problemas)

---

## 🔍 Visão Geral

A sonda RS41-NFW v65 transmite telemetria usando múltiplos protocolos simultaneamente. O ESP32APRS_LoRa é um iGate (Internet Gateway) para APRS que pode ser configurado para receber e retransmitir esses dados para a Internet via APRS-IS.

**Equipamentos:**
- **Sonda**: Vaisala RS41 com firmware RS41-NFW v65 por SP5FRA
- **iGate**: ESP32APRS_LoRa por HS5TQA (nakhonthai)
- **Callsign**: PU7IOL-1

---

## 📡 Configurações da Sonda RS41-NFW

### Modo 1: APRS (✅ SUPORTADO pelo ESP32APRS_LoRa)

```cpp
// Configuração atual no firmware da sonda
bool aprsEnable = true;
constexpr float aprsFreqTable[] = {433.55};      // Frequência: 433.55 MHz
char aprsCall[] = "PU7IOL-1";                    // Callsign
constexpr char aprsSsid = 11;                    // SSID: 11
constexpr uint16_t aprsTimeSyncSeconds = 30;     // Intervalo: 30 segundos
constexpr int8_t aprsRadioPower = 7;            // Potência: 20dBm (100mW)
constexpr int8_t aprsOperationMode = 1;         // Modo: tracker telemetry
```

**Especificações Técnicas APRS:**
- **Modulação**: AFSK (Audio Frequency Shift Keying)
- **Taxa de dados**: 1200 baud (Bell 202)
- **Mark frequency**: 1200 Hz (indica bit "1")
- **Space frequency**: 2200 Hz (indica bit "0")
- **Desvio FM**: ±3 kHz (típico)
- **Potência de transmissão**: 100mW (20dBm)

**Formato do Pacote APRS:**
- Callsign: `PU7IOL-1`
- Destino: `APRNFW`
- Digipeater: `WIDE2-1`
- Símbolo: `/O` (balão)
- Comentário: Inclui dados de telemetria:
  - **F**: Frame counter
  - **S**: Número de satélites GPS
  - **V**: Tensão da bateria (V)
  - **C**: Taxa de subida (m/s)
  - **I**: Temperatura interna (°C)
  - **T**: Temperatura externa (°C)
  - **H**: Umidade externa (%)
  - **P**: Pressão atmosférica (hPa)

---

### Modo 2: Horus V3 (❌ NÃO SUPORTADO diretamente pelo ESP32APRS_LoRa)

```cpp
// Configuração atual no firmware da sonda
bool horusV3Enable = true;
constexpr float horusV3FreqTable[] = {430.43};   // Frequência: 430.43 MHz
#define HORUS_V3_CALLSIGN "PU7IOL-1"
constexpr uint16_t horusV3Bdr = 100;             // Baudrate: 100 baud
constexpr uint16_t horusV3TimeSyncSeconds = 15;  // Intervalo: 15 segundos
constexpr int8_t horusV3RadioPower = 7;          // Potência: 20dBm (100mW)
bool horusV3LongerPacket = true;                 // Pacotes estendidos
```

**Especificações Técnicas Horus V3:**
- **Modulação**: 4FSK (4-level Frequency Shift Keying)
- **Taxa de símbolos**: 100 baud
- **Codificação**: ASN.1 com Golay (23,12) FEC
- **Potência de transmissão**: 100mW (20dBm)
- **Características**: Correção de erros, callsign no pacote, dados estendidos

**⚠️ IMPORTANTE**: O ESP32APRS_LoRa **NÃO suporta modulação 4FSK**. Horus V3 requer decodificadores específicos como:
- Horus-GUI (Windows/Linux/Mac)
- dl-fldigi com plugin Horus
- SondeHub com receptor SDR

---

### Outros Modos Disponíveis (Desabilitados)

```cpp
// RTTY (desabilitado)
bool rttyEnable = false;
constexpr float rttyFrequencyMhz = 434.6;
constexpr uint16_t rttyBitDelay = 10000;  // ~100 baud

// Morse (desabilitado)
bool morseEnable = false;
constexpr float morseFrequencyMhz = 434.6;

// Horus V2 (obsoleto, desabilitado)
bool horusEnable = false;
```

---

## ⚙️ Configuração do ESP32APRS_LoRa

### Acesso à Interface Web

1. **Conecte-se ao ESP32APRS_LoRa**:
   - Via WiFi (se configurado como AP): `http://192.168.4.1`
   - Via rede local: `http://192.168.31.81` (seu caso)

2. **Login**: Use as credenciais configuradas

---

### Configuração Modo iGate (RF → Internet)

#### 1. Sistema → Configuração Básica

```
Callsign: PU7IOL
SSID: 1
Comment: RS41-NFW iGate
TX Power: 20 (dBm)
```

#### 2. iGate → Configuração Principal

```
☑ Enable iGate
☑ Enable RF to INET (RF2INET)
☐ Enable INET to RF (INET2RF) [não recomendado para sondes]

APRS-IS Server: rotate.aprs2.net
Port: 14580
Filter: r/[LAT]/[LON]/100  [substitua LAT/LON pela sua localização]

Exemplo de filtro:
r/-23.5489/-46.6388/100  [São Paulo, raio de 100km]
```

**Explicação do Filtro:**
- `r/LAT/LON/RAIO` - Recebe pacotes dentro do raio especificado (em km)
- Para rastrear a sonda especificamente: `b/PU7IOL-1`
- Combinação: `r/-23.5489/-46.6388/100 b/PU7IOL-1`

#### 3. Radio → Configuração LoRa/RF

**⚠️ IMPORTANTE**: Configure o rádio para receber APRS da sonda

```
Frequency: 433.550 MHz

Modulation: FM/AFSK (para APRS)
Bandwidth: 25 kHz (ou conforme capacidade do hardware)

☑ Enable RX
☑ Enable TX [apenas se quiser retransmitir localmente]
```

**Nota sobre Modulação:**
- O ESP32APRS_LoRa com módulo LoRa (SX127x) suporta **LoRa** e **FSK**, não AFSK diretamente
- Para receber APRS AFSK (1200 baud), você precisará de um **módulo FM tradicional** ou **SDR**
- Alternativamente, use um **rádio FM comum** conectado ao ESP32 via áudio (modo TNC)

#### 4. Position → Localização do iGate

```
Latitude: -23.5489  [exemplo: São Paulo]
Longitude: -46.6388
Altitude: 760 (metros)

☑ Enable Position Beacon
Beacon Interval: 600 (segundos, 10 minutos)

Symbol Table: /
Symbol Code: &  [símbolo de gateway)
```

#### 5. Display → Opções de Exibição

```
☑ Show RX Packet
☑ Show TX Packet
☑ Show Position
☑ Filter Duplicate Packets [recommended]

Display Timeout: 60 (seconds)
```

---

## 🔄 Protocolos Suportados

### ✅ Suportados pelo ESP32APRS_LoRa

| Protocolo | Frequência | Modulação | Status | Notas |
|-----------|------------|-----------|--------|-------|
| **APRS** | 433.55 MHz | AFSK 1200 baud | ✅ Suportado | Requer receptor FM ou SDR |
| **LoRa APRS** | Variável | LoRa | ✅ Nativo | Não usado pela sonda |

### ❌ NÃO Suportados pelo ESP32APRS_LoRa

| Protocolo | Frequência | Modulação | Status | Alternativa |
|-----------|------------|-----------|--------|-------------|
| **Horus V3** | 430.43 MHz | 4FSK 100 baud | ❌ Não suportado | Use Horus-GUI + SDR |
| **RTTY** | 434.6 MHz | FSK 100 baud | ❌ Não suportado | Use dl-fldigi + SDR |
| **Morse** | 434.6 MHz | CW | ❌ Não suportado | Decodificador Morse + SDR |

---

## ⚠️ Limitações Importantes

### 1. Compatibilidade de Hardware

**O ESP32APRS_LoRa com módulo LoRa (SX1276/SX1278) NÃO pode decodificar APRS AFSK diretamente.**

**Soluções:**

#### Opção A: Usar Receptor FM Externo (Recomendado)
```
[Receptor FM] → [Cabo de áudio] → [Entrada ADC do ESP32]
   433.55 MHz      Áudio AFSK         Demodulação BELL 202
```

Configuração:
1. Sintonize um receptor FM em 433.55 MHz
2. Conecte a saída de áudio ao pino ADC do ESP32 (ex: GPIO 34)
3. Configure o ESP32APRS_LoRa para modo TNC (Terminal Node Controller)
4. O ESP32 decodifica o AFSK em software

#### Opção B: Usar SDR (Software Defined Radio)
```
[Antena] → [RTL-SDR] → [PC com direwolf] → [APRS-IS]
                433.55 MHz     Demod AFSK      Internet
```

Software necessário:
- **RTL-SDR** ou similar (AirSpy, HackRF)
- **Direwolf** (TNC software para Linux/Windows/Mac)
- **SDR#** ou **GQRX** (receptor SDR)

#### Opção C: Usar Dual Setup (Melhor cobertura)
```
Sistema 1: [Receptor FM] → [ESP32APRS_LoRa] → APRS AFSK @ 433.55 MHz
Sistema 2: [RTL-SDR] → [Horus-GUI] → Horus V3 4FSK @ 430.43 MHz
                                  ↓
                            [SondeHub.org]
```

---

### 2. Frequências Diferentes

A sonda transmite em **DUAS frequências simultaneamente**:
- **433.55 MHz** - APRS (AFSK)
- **430.43 MHz** - Horus V3 (4FSK)

O ESP32 com um único receptor só pode monitorar **UMA frequência por vez**. Soluções:

1. **Priorizar APRS** (433.55 MHz) - Dados vão direto para APRS-IS
2. **Priorizar Horus V3** (430.43 MHz) - Requer decodificador específico
3. **Usar dois receptores** - Cobertura completa de ambos os protocolos

---

### 3. Upload para Internet

**APRS** → ESP32APRS_LoRa iGate → **APRS-IS** (aprs.fi, aprs2.net)
- ✅ Funcionamento automático após configuração
- ✅ Rastreamento em tempo real
- ✅ Mapas online instantâneos

**Horus V3** → Decodificador → **SondeHub.org**
- ⚠️ Requer setup adicional (RTL-SDR + Horus-GUI)
- ✅ Maior precisão de dados
- ✅ Correção de erros FEC
- ✅ Upload automático para SondeHub

---

## 📝 Configuração Passo a Passo

### Cenário 1: Receber Apenas APRS (433.55 MHz)

**Requisitos:**
- ESP32APRS_LoRa
- Receptor FM compatível (ex: Baofeng UV-5R, NiceRF SA828)
- Cabo de áudio

**Passos:**

1. **Hardware:**
   ```
   [Receptor FM] GPIO34 ← [Saída de áudio]
        433.55 MHz        Volume: 50%
   ```

2. **Configuração do Receptor:**
   - Frequência: `433.550 MHz`
   - Modo: `FM` (narrow band 25kHz)
   - Squelch: `Aberto` ou muito baixo
   - Volume: `50%` (ajustar para evitar distorção)

3. **Configuração ESP32APRS_LoRa** (http://192.168.31.81):

   **System:**
   ```
   Callsign: PU7IOL
   SSID: 1
   ```

   **iGate:**
   ```
   ☑ Enable iGate
   ☑ Enable RF2INET
   Server: rotate.aprs2.net
   Port: 14580
   Filter: b/PU7IOL-1
   ```

   **TNC:**
   ```
   Mode: AFSK1200
   Input: ADC (GPIO34)
   RX Gain: 50 (ajustar conforme necessário)
   TX Audio: Disabled (apenas RX)
   ```

4. **Verificação:**
   - Aguarde transmissão da sonda (a cada 30s)
   - Verifique console serial: `RX: PU7IOL-1>APRNFW,WIDE2-1:...`
   - Acesse [aprs.fi](https://aprs.fi) → Busque `PU7IOL-1`
   - Confirme que os pacotes estão chegando ao APRS-IS

---

### Cenário 2: Receber Horus V3 (430.43 MHz) com SDR

**Requisitos:**
- RTL-SDR ou compatível
- PC com Windows/Linux/Mac
- Software Horus-GUI

**Passos:**

1. **Instalar Software:**
   - **Windows**: Baixe [Horus-GUI](https://github.com/projecthorus/horus-gui/releases)
   - **Linux**: `pip3 install horusdemodlib`
   - Instale drivers RTL-SDR

2. **Configurar Horus-GUI:**
   ```
   Frequency: 430.430 MHz
   Mode: LoRa 4FSK
   Baud Rate: 100
   Bandwidth: 3000 Hz
   
   ☑ Upload to SondeHub
   Callsign: PU7IOL (seu callsign de operador)
   ```

3. **Iniciar Recepção:**
   - Conecte o RTL-SDR
   - Clique em "Start"
   - Aguarde transmissão Horus V3 (a cada 15s)
   - Verifique decodificação na janela de log

4. **Verificação:**
   - Acesse [SondeHub.org](https://sondehub.org)
   - Busque pelo callsign `PU7IOL-1`
   - Confirme recepção de telemetria

---

### Cenário 3: Setup Completo Dual Band (RECOMENDADO)

**Configuração Ideal:**

```
┌─────────────────────────────────────────────┐
│         SONDA RS41-NFW v65 (PU7IOL-1)       │
│                                             │
│  TX1: 433.55 MHz APRS    TX2: 430.43 MHz   │
│       AFSK 1200 baud           Horus V3     │
│       Intervalo: 30s           4FSK 100 baud│
│                                Intervalo: 15s│
└──────────┬──────────────────────┬─────────────┘
           │                      │
           ▼                      ▼
  ┌─────────────────┐    ┌──────────────────┐
  │ Sistema 1: APRS │    │Sistema 2: Horus  │
  │                 │    │                  │
  │ [RX FM 433.55]  │    │ [RTL-SDR 430.43] │
  │       ↓         │    │       ↓          │
  │ [ESP32APRS_LoRa]│    │ [Horus-GUI/PC]   │
  │       ↓         │    │       ↓          │
  │ [APRS-IS]       │    │ [SondeHub.org]   │
  └─────────────────┘    └──────────────────┘
```

**Vantagens:**
- ✅ Cobertura completa de ambos os protocolos
- ✅ Redundância (se um falhar, o outro continua)
- ✅ APRS: rastreamento ao vivo, baixa latência
- ✅ Horus V3: alta precisão, FEC, dados estendidos
- ✅ Contribui para duas redes (APRS-IS e SondeHub)

---

## 🔧 Solução de Problemas

### Problema 1: ESP32 não recebe pacotes APRS

**Causas possíveis:**

1. **Receptor não sintonizado corretamente:**
   - Verifique frequência: `433.550 MHz` (exata)
   - Modo: `FM narrow` (25 kHz)
   - Squelch: aberto

2. **Nível de áudio incorreto:**
   - Muito baixo: nenhum sinal detectado
   - Muito alto: distorção, erros de decodificação
   - **Solução**: Ajuste volume do receptor para 50%, ajuste RX Gain no ESP32

3. **Antena inadequada:**
   - Use antena com bom ganho para 433 MHz
   - Polarização vertical
   - Linha de visada desobstruída

4. **Interferência:**
   - 433 MHz é banda ISM (Industrial, Scientific, Medical)
   - Muitos dispositivos transmitem nesta faixa
   - **Solução**: Filtro passa-faixa, local elevado, antena direcional

---

### Problema 2: Pacotes decodificados mas não aparecem no APRS-IS

**Verificações:**

1. **Conexão com APRS-IS:**
   ```
   Status: Connected ✅
   Server: rotate.aprs2.net:14580
   ```

2. **Credenciais corretas:**
   - Callsign: `PU7IOL`
   - SSID: `1`
   - Passcode: calculado automaticamente (baseado no callsign)

3. **Verificar console serial:**
   ```
   TX APRS-IS: PU7IOL-1>APRNFW,WIDE2-1,qAR,PU7IOL-1:!LAT/LON...
   ```

4. **Firewall/Rede:**
   - Porta `14580` deve estar aberta (saída)
   - Verifique se o ESP32 tem acesso à Internet

5. **Filtro APRS-IS:**
   - Remova filtros muito restritivos
   - Teste com `b/PU7IOL-1` (específico para a sonda)

---

### Problema 3: Horus V3 não decodifica

**Causas:**

1. **SDR não detectado:**
   - Verifique drivers RTL-SDR
   - Teste com `rtl_test` (Linux) ou `SDR#` (Windows)

2. **Frequência incorreta:**
   - Confirme: `430.430 MHz`
   - Horus V3 tem **15 preamble bits** - facilita sincronização

3. **Taxa de símbolos errada:**
   - Deve ser `100 baud` (4FSK)
   - Não confundir com bitrate (400 bps para 4FSK 100 baud)

4. **Ganho do SDR:**
   - Muito baixo: SNR insuficiente
   - Muito alto: saturação, distorção
   - **Ideal**: 30-40 dB (ajustar com AGC off)

5. **Software desatualizado:**
   - Horus-GUI deve estar na versão mais recente
   - Suporte a Horus V3 foi adicionado recentemente
   - Verifique: [GitHub Horus-GUI](https://github.com/projecthorus/horus-gui)

---

### Problema 4: Distância de recepção muito curta

**Melhorias:**

1. **Antena:**
   - Substituir por antena com maior ganho (Yagi, quad, collinear)
   - Instalar em local elevado
   - Cabo coaxial de baixa perda (LMR-400, RG-213)

2. **LNA (Low Noise Amplifier):**
   - Adicionar pré-amplificador entre antena e receptor
   - Exemplo: LNA4ALL (0.1-2 GHz, 20 dB ganho)
   - Instalar próximo à antena (menor ruído)

3. **Filtro:**
   - Filtro passa-faixa 430-435 MHz
   - Reduz interferências de outras fontes

4. **Local de instalação:**
   - Telhado, torre, morro
   - Linha de visada desobstruída
   - Evitar prédios altos, montanhas

**Alcance esperado:**
- **Sonda a 5km altitude**: 50-100 km de alcance
- **Sonda a 10km altitude**: 100-200 km de alcance
- **Sonda a 30km altitude**: 300-500 km de alcance
- Valores dependem de potência (100mW), antenas, topografia

---

## 📊 Dados Transmitidos pela Sonda

### APRS - Telemetria Padrão

Formato do comentário APRS:
```
F123 S8 V2.75 C+12.5 I-25 T-48 H65 P1013 J0 R2
```

| Campo | Descrição | Unidade | Exemplo |
|-------|-----------|---------|---------|
| **F** | Frame counter | - | 123 |
| **S** | Satélites GPS | - | 8 |
| **V** | Tensão bateria | Volts | 2.75 |
| **C** | Taxa de subida | m/s | +12.5 |
| **I** | Temp. interna | °C | -25 |
| **T** | Temp. externa | °C | -48 |
| **H** | Umidade | % | 65 |
| **P** | Pressão | hPa | 1013 |
| **J** | Aviso de jamming | 0/1 | 0 |
| **R** | Revisão PCB | - | 2 (RSM4x2) |

---

### Horus V3 - Telemetria Estendida

**Pacote padrão (23 bytes):**
```
Callsign: PU7IOL-1
Timestamp: HH:MM:SS
Position: Lat, Lon, Alt
Speed: km/h
Satellites: 8
Temperature: -48°C
Battery: 2.75V
```

**Pacote estendido (horusV3LongerPacket = true):**
Adiciona:
- **Temperatura módulo de umidade**
- **Status GPS** (0=off, 1=max perf, 2=powersave, 3=efficient tracking)
- **HDOP** (Horizontal Dilution of Precision)

**Vantagens do Horus V3:**
- ✅ Codificação ASN.1 (flexível, não precisa de payload ID)
- ✅ Golay (23,12) FEC (corrige até 3 erros por codeword)
- ✅ Scrambler (evita longas sequências iguais)
- ✅ Interleaver (resistente a burst errors)
- ✅ CRC16 (detecção de erros)

---

## 🌐 Recursos Online

### Rastreamento

- **APRS.fi**: https://aprs.fi
  - Busque: `PU7IOL-1`
  - Visualize trilha, altitude, velocidade

- **SondeHub**: https://sondehub.org
  - Busque: `PU7IOL-1`
  - Dados Horus V3, telemetria estendida

### Software

- **ESP32APRS_LoRa**: https://github.com/nakhonthai/ESP32APRS_LoRa
- **Horus-GUI**: https://github.com/projecthorus/horus-gui
- **Direwolf**: https://github.com/wb2osz/direwolf
- **RS41-NFW Firmware**: https://github.com/sp5fra/rs41-nfw

### Documentação

- **APRS Protocol**: http://www.aprs.org/doc/APRS101.PDF
- **Horus V3**: https://github.com/projecthorus/horusdemodlib/wiki
- **ESP32APRS_LoRa Wiki**: https://github.com/nakhonthai/ESP32APRS_LoRa/wiki

---

## 📞 Suporte

### Comunidades

- **Grupo Telegram**: Sondas Brasil
- **Email**: PU7IOL (seu email)
- **GitHub Issues**: Repositórios dos projetos

### Frequências para Testes

**Brasil** (autorização necessária):
- **430-440 MHz**: Banda de radioamador 70cm
- **433.05-434.79 MHz**: Banda ISM (baixa potência)

**Important**: Verifique regulamentação ANATEL antes de transmitir.

---

## ✅ Checklist Pré-Lançamento

### Sonda RS41-NFW

- [ ] Firmware RS41-NFW v65 atualizado
- [ ] Callsign configurado: `PU7IOL-1`
- [ ] APRS habilitado: 433.55 MHz
- [ ] Horus V3 habilitado: 430.43 MHz
- [ ] GPS com fix (LED verde constante)
- [ ] Bateria cheia (> 2.5V)
- [ ] Sensor boom calibrado
- [ ] Antena instalada
- [ ] Envelope inflado (se lançamento balão)

### iGate ESP32APRS_LoRa

- [ ] Hardware montado e testado
- [ ] Receptor FM sintonizado em 433.55 MHz
- [ ] iGate configurado (RF2INET)
- [ ] Conexão APRS-IS ativa
- [ ] Filtro configurado: `b/PU7IOL-1`
- [ ] Antena instalada em local elevado
- [ ] Console mostrando pacotes recebidos
- [ ] Teste de upload: pacotes visíveis em aprs.fi

### Sistema Horus V3 (Opcional)

- [ ] RTL-SDR conectado e funcionando
- [ ] Horus-GUI instalado e atualizado
- [ ] Frequência: 430.43 MHz
- [ ] Upload SondeHub habilitado
- [ ] Teste de decodificação OK

---

## 📈 Exemplo de Setup Completo

### Equipamentos

```
SONDA:
└─ Vaisala RS41 (RSM4x2)
   ├─ Firmware: RS41-NFW v65
   ├─ GPS: uBlox M10 (RSM4x4) ou NEO-6/7/8 (RSM4x2)
   ├─ Rádio: Si4032 (26 MHz XTAL)
   ├─ Sensor: RPM411 (Temp/Hum/Press)
   └─ Bateria: 3x AA (4.5V) ou 2x AA (3.0V)

ESTAÇÃO TERRESTRE:
└─ Sistema 1: APRS
   ├─ Antena: Diamond X-50N (2m/70cm)
   ├─ Receptor: NiceRF SA828 (433 MHz FM)
   ├─ TNC: ESP32APRS_LoRa
   └─ Internet: WiFi/Ethernet → APRS-IS

└─ Sistema 2: Horus V3
   ├─ Antena: Diamond X-50N (2m/70cm)
   ├─ LNA: LNA4ALL (+20dB)
   ├─ SDR: RTL-SDR Blog V3
   ├─ Software: Horus-GUI
   └─ Upload: SondeHub.org
```

### Topologia de Rede

```
                    ┌─────────────┐
                    │  INTERNET   │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
         ┌────▼──────┐          ┌──────▼────┐
         │  APRS-IS  │          │ SondeHub  │
         │ Port 14580│          │  Tracker  │
         └────▲──────┘          └──────▲────┘
              │                        │
              │                        │
      ┌───────┴────────┐      ┌────────┴────────┐
      │ ESP32APRS_LoRa │      │   Horus-GUI     │
      │  192.168.31.81 │      │   via RTL-SDR   │
      └───────▲────────┘      └────────▲────────┘
              │                        │
              │ 433.55 MHz             │ 430.43 MHz
              │ APRS AFSK              │ Horus 4FSK
              │                        │
         ┌────┴────────────────────────┴────┐
         │   SONDA RS41-NFW v65 (PU7IOL-1)  │
         │      Altitude: 0-35 km            │
         │      Velocidade: 0-50 km/h        │
         └───────────────────────────────────┘
```

---

## 🎯 Resumo Executivo

### O que FUNCIONA:

✅ **ESP32APRS_LoRa recebe APRS** (433.55 MHz) com receptor FM externo  
✅ **Upload automático para APRS-IS** (aprs.fi, aprs2.net)  
✅ **Rastreamento em tempo real** via Internet  

### O que NÃO FUNCIONA:

❌ **ESP32APRS_LoRa NÃO recebe Horus V3** (430.43 MHz, 4FSK)  
❌ **Módulo LoRa SX127x NÃO decodifica AFSK** diretamente  

### Recomendação FINAL:

**Setup Híbrido:**
1. **APRS**: Receptor FM + ESP32APRS_LoRa → APRS-IS
2. **Horus V3**: RTL-SDR + Horus-GUI → SondeHub

Ou se preferir **APENAS APRS**:
- Configure receptor FM em 433.55 MHz
- Conecte ao ESP32APRS_LoRa (modo TNC)
- Os dados aparecerão automaticamente em aprs.fi

---

## 📝 Notas Finais

Esta documentação foi criada especificamente para a configuração:
- **Sonda**: RS41-NFW v65 (firmware por SP5FRA)
- **Callsign**: PU7IOL-1
- **iGate**: ESP32APRS_LoRa (por HS5TQA)
- **Localização**: Brasil

Para outras configurações ou dúvidas específicas, consulte a documentação oficial dos projetos ou entre em contato com as comunidades de radioamadores e sondas meteorológicas.

**73 e bons voos!** 🎈📡

---

*Documento gerado em: 2024*  
*Versão: 1.0*  
*Autor: Documentação assistida por IA com base em análise do código-fonte*
