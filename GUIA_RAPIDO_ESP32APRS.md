# Guia Rápido: Configuração ESP32APRS_LoRa para RS41-NFW

## 🎯 Objetivo
Configurar o ESP32APRS_LoRa em http://192.168.31.81 para receber telemetria APRS da sonda RS41-NFW v65 (callsign PU7IOL-1) e retransmitir para APRS-IS.

---

## ⚠️ AVISO IMPORTANTE

**O ESP32APRS_LoRa com módulo LoRa (SX127x) NÃO pode receber APRS AFSK diretamente!**

### Soluções:

**Opção 1 (Recomendada):** Adicionar receptor FM externo
```
[Receptor FM 433.55 MHz] → [Cabo áudio] → [GPIO ADC do ESP32]
```

**Opção 2:** Usar SDR ao invés do ESP32APRS_LoRa
```
[RTL-SDR] → [Direwolf em PC] → [APRS-IS]
```

**Opção 3:** Trocar módulo LoRa por módulo FM (SA828, DRA818)

---

## 📋 Configuração Interface Web (192.168.31.81)

### 1. System Settings

**Acesse:** `System` → `Config`

```
┌─────────────────────────────────────┐
│ SYSTEM CONFIGURATION                │
├─────────────────────────────────────┤
│ Callsign: PU7IOL                    │
│ SSID: 1                             │
│ Comment: RS41-NFW iGate            │
│ Latitude: -23.5489                  │ [Sua localização]
│ Longitude: -46.6388                 │ [Sua localização]
│ Altitude: 760                       │ [metros]
│                                     │
│ [x] Enable Position Beacon          │
│ Beacon Interval: 600                │ [segundos]
│                                     │
│ Symbol Table: /                     │
│ Symbol Code: &                      │ [Gateway]
└─────────────────────────────────────┘
   [Save]  [Reboot]
```

---

### 2. iGate Settings

**Acesse:** `iGate` → `Config`

```
┌─────────────────────────────────────┐
│ IGATE CONFIGURATION                 │
├─────────────────────────────────────┤
│ [x] Enable iGate                    │
│ [x] Enable RF to INET (RF2INET)     │
│ [ ] Enable INET to RF (INET2RF)     │ [Deixar DESMARCADO]
│                                     │
│ ─────── APRS-IS Server ──────────   │
│ Server: rotate.aprs2.net            │
│ Port: 14580                         │
│ Passcode: [auto]                    │ [Gerado automaticamente]
│                                     │
│ ─────── Filter ──────────────────   │
│ Filter: b/PU7IOL-1                  │ [Específico para a sonda]
│                                     │
│ Alternate Filter Options:           │
│ r/-23.5489/-46.6388/100             │ [Raio 100km da posição]
│ b/PU7IOL-1 r/-23.5489/-46.6388/100  │ [Combinação]
└─────────────────────────────────────┘
   [Save]  [Test Connection]
```

**Explicação dos Filtros:**
- `b/PU7IOL-1` - Recebe apenas callsign PU7IOL-1
- `r/LAT/LON/RAIO` - Recebe tudo num raio (km) da posição
- Combinação: Use espaço entre filtros

---

### 3. Radio Settings (COM RECEPTOR FM EXTERNO)

**Acesse:** `Radio` → `Config`

```
┌─────────────────────────────────────┐
│ RADIO CONFIGURATION                 │
├─────────────────────────────────────┤
│ ─────── Mode ─────────────────────  │
│ ( ) LoRa Mode                       │
│ (x) AFSK/FM Mode (TNC)              │ [Selecionar este]
│                                     │
│ ─────── Frequency ────────────────  │
│ RX Frequency: 433.550               │ [MHz]
│ TX Frequency: 433.550               │ [MHz]
│                                     │
│ ─────── AFSK Settings ────────────  │
│ Modulation: Bell 202                │
│ Baud Rate: 1200                     │
│ Mark: 1200 Hz                       │
│ Space: 2200 Hz                      │
│                                     │
│ ─────── TNC Settings ──────────────│
│ Input: ADC (GPIO34)                 │ [Conectar receptor FM aqui]
│ RX Gain: 50                         │ [Ajustar conforme necessário]
│ TX Enable: [ ]                      │ [Desabilitar se não quiser TX]
└─────────────────────────────────────┘
   [Save]  [Test Audio]
```

**Conexão Hardware:**
```
Receptor FM (Baofeng, SA828, etc)
├─ Frequência: 433.550 MHz
├─ Modo: FM Narrow (25 kHz)
├─ Squelch: Aberto (ou muito baixo)
└─ Saída de áudio → ESP32 GPIO34 (ou outro ADC)
      │
      └─ Volume: ~50% (ajustar para evitar distorção)
```

---

### 4. Display Settings

**Acesse:** `Display` → `Config`

```
┌─────────────────────────────────────┐
│ DISPLAY CONFIGURATION               │
├─────────────────────────────────────┤
│ [x] Show RX Packets                 │
│ [x] Show TX Packets                 │
│ [x] Show Position                   │
│ [x] Filter Duplicates               │
│                                     │
│ Display Timeout: 60                 │ [segundos]
│ Max Packets: 50                     │
└─────────────────────────────────────┘
   [Save]
```

---

### 5. WiFi Settings (Opcional)

**Acesse:** `WiFi` → `Config`

```
┌─────────────────────────────────────┐
│ WIFI CONFIGURATION                  │
├─────────────────────────────────────┤
│ ─────── WiFi Mode ─────────────────│
│ (x) WiFi Client (STA)               │
│ ( ) Access Point (AP)               │
│                                     │
│ SSID: [Seu_WiFi]                    │
│ Password: [Sua_Senha]               │
│                                     │
│ IP Configuration:                   │
│ (x) DHCP                            │
│ ( ) Static IP                       │
│                                     │
│ Current IP: 192.168.31.81           │
└─────────────────────────────────────┘
   [Save]  [Reconnect]
```

---

## 🔍 Monitoramento e Diagnóstico

### Console Serial (115200 baud)

**Pacote Recebido com Sucesso:**
```
[INFO] RX: PU7IOL-1>APRNFW,WIDE2-1,qAR,PU7IOL-1:!2317.52S/04638.33W>000/000/A=012345 F123 S8 V2.75 C+12 NFWv65
[INFO] Decoded: LAT=-23.292, LON=-46.639, ALT=3765m
[INFO] TX APRS-IS: OK
[INFO] Status: https://aprs.fi/#!call=a%2FPU7IOL-1
```

**Erros Comuns:**
```
[ERROR] RX: Invalid checksum
[ERROR] RX: Incomplete packet
[ERROR] APRS-IS: Connection failed
[WARN] APRS-IS: Duplicate packet, not forwarding
```

---

### Status LED

| LED | Estado | Significado |
|-----|--------|-------------|
| Verde | Sólido | Conectado ao WiFi e APRS-IS |
| Verde | Piscando | Recebendo pacotes |
| Azul | Piscando | Transmitindo para APRS-IS |
| Amarelo | Sólido | Conectado WiFi, mas sem APRS-IS |
| Vermelho | Sólido | Erro crítico (sem WiFi) |

---

### Página de Status

**Acesse:** `Status` → `Info`

```
┌─────────────────────────────────────┐
│ SYSTEM STATUS                       │
├─────────────────────────────────────┤
│ Uptime: 2h 34m 12s                  │
│ Free Memory: 234 KB / 512 KB        │
│ WiFi RSSI: -52 dBm                  │
│                                     │
│ ─────── APRS-IS Status ──────────   │
│ Connected: YES                      │
│ Server: rotate.aprs2.net:14580      │
│ Callsign: PU7IOL-1                  │
│ Session: 1h 23m                     │
│                                     │
│ ─────── Packet Stats ────────────   │
│ RX Total: 45                        │
│ RX Valid: 42                        │
│ RX Errors: 3                        │
│ TX APRS-IS: 42                      │
│ Duplicates: 12                      │
│                                     │
│ ─────── Last Packet ──────────────  │
│ From: PU7IOL-1                      │
│ Time: 12:34:56 (15s ago)            │
│ Position: -23.292, -46.639          │
│ Altitude: 3765 m                    │
│ Speed: 12 km/h                      │
│ Course: 245°                        │
│ Comment: F123 S8 V2.75 NFWv65       │
└─────────────────────────────────────┘
   [Refresh]  [Clear Stats]
```

---

## ✅ Checklist de Configuração

### Pré-requisitos
- [ ] ESP32APRS_LoRa montado e funcionando
- [ ] Receptor FM em 433.55 MHz (ex: Baofeng, SA828)
- [ ] Cabo de áudio conectado (receptor → GPIO34 ESP32)
- [ ] Antena 433 MHz instalada
- [ ] WiFi configurado e conectado
- [ ] Acesso à interface web (192.168.31.81)

### Configuração (30-45 minutos)
- [ ] **System**: Callsign PU7IOL-1, posição, símbolo gateway
- [ ] **iGate**: RF2INET habilitado, servidor rotate.aprs2.net:14580
- [ ] **Radio**: Modo AFSK/FM 433.55 MHz, entrada ADC GPIO34
- [ ] **Display**: Opções de visualização habilitadas
- [ ] Salvar todas as configurações
- [ ] Reiniciar ESP32

### Testes (15-20 minutos)
- [ ] Console serial mostra conexão WiFi OK
- [ ] Console mostra conexão APRS-IS OK
- [ ] Receptor FM sintonizado em 433.55 MHz
- [ ] Volume do receptor ajustado (~50%)
- [ ] Aguardar transmissão da sonda (intervalo 30s)
- [ ] Console mostra: `RX: PU7IOL-1>APRNFW...`
- [ ] Console mostra: `TX APRS-IS: OK`
- [ ] Verificar no navegador: https://aprs.fi → buscar PU7IOL-1
- [ ] Confirmar que pacotes estão aparecendo no mapa

### Troubleshooting (se necessário)
- [ ] Verificar frequência receptor: exatamente 433.550 MHz
- [ ] Verificar modo: FM Narrow (25 kHz)
- [ ] Ajustar squelch: aberto ou muito baixo
- [ ] Ajustar volume: 30-70% (testar diferentes valores)
- [ ] Ajustar RX Gain no ESP32: 30-70 (testar diferentes valores)
- [ ] Verificar conexão física: cabo de áudio bem conectado
- [ ] Verificar antena: SWR baixo, conexões OK
- [ ] Testar com outro receptor FM
- [ ] Verificar log de erros no console serial

---

## 📊 Parâmetros da Sonda RS41-NFW v65

### APRS (Suportado)
```yaml
Protocolo: APRS
Frequência: 433.550 MHz
Modulação: AFSK (Bell 202)
Taxa: 1200 baud
Mark/Space: 1200 Hz / 2200 Hz
Desvio FM: ±3 kHz
Intervalo TX: 30 segundos
Potência: 100 mW (20 dBm)
Callsign: PU7IOL-1
SSID: 11
Destino: APRNFW
Digi: WIDE2-1
```

### Horus V3 (NÃO Suportado pelo ESP32APRS_LoRa)
```yaml
Protocolo: Horus V3
Frequência: 430.430 MHz
Modulação: 4FSK
Taxa símbolos: 100 baud
Taxa bits: 400 bps (4FSK)
Codificação: ASN.1 + Golay (23,12)
Intervalo TX: 15 segundos
Potência: 100 mW (20 dBm)
Callsign: PU7IOL-1 (no payload)
```

**Para receber Horus V3:** Use RTL-SDR + Horus-GUI

---

## 🌐 Verificação Online

Após configuração e primeiro pacote recebido:

### APRS.fi
1. Acesse: https://aprs.fi
2. Busque: `PU7IOL-1`
3. Verifique:
   - Posição no mapa
   - Trilha (trail)
   - Altitude
   - Último pacote recebido
   - Estação que fez forward (sua: PU7IOL-1)

### APRS Direct
1. Acesse: https://www.aprsdirect.com
2. Busque: `PU7IOL-1`
3. Verifique linha do tempo de pacotes

### APRS2.net
1. Acesse: https://aprs2.net
2. Busque: `PU7IOL-1`
3. Verifique estatísticas detalhadas

---

## 🔗 Links Úteis

### Documentação Oficial
- **ESP32APRS_LoRa GitHub**: https://github.com/nakhonthai/ESP32APRS_LoRa
- **ESP32APRS_LoRa Wiki**: https://github.com/nakhonthai/ESP32APRS_LoRa/wiki
- **RS41-NFW Firmware**: https://github.com/sp5fra/rs41-nfw

### Rastreamento
- **APRS.fi**: https://aprs.fi
- **APRS Direct**: https://www.aprsdirect.com
- **APRS2**: https://aprs2.net

### Software Adicional
- **Direwolf TNC**: https://github.com/wb2osz/direwolf
- **Horus-GUI**: https://github.com/projecthorus/horus-gui
- **SondeHub**: https://sondehub.org

### Comunidades
- **Telegram**: Grupo "Sondas Brasil"
- **Reddit**: r/amateurradio
- **Facebook**: Grupos de radioamadores locais

---

## 📝 Exemplo Completo: Primeiro Uso

### 1. Preparação (10 min)
```bash
# Conectar hardware
1. Instalar antena 433 MHz no receptor FM
2. Ligar receptor FM, sintonizar 433.550 MHz
3. Conectar cabo áudio: Receptor (saída) → ESP32 (GPIO34)
4. Conectar ESP32 ao computador via USB (para debug serial)
5. Ligar ESP32
```

### 2. Configuração Web (20 min)
```bash
1. Conectar PC ao mesmo WiFi que ESP32
2. Abrir navegador: http://192.168.31.81
3. Login com credenciais
4. System: definir PU7IOL-1, posição GPS, símbolo
5. iGate: habilitar RF2INET, servidor rotate.aprs2.net:14580
6. Radio: modo AFSK/FM, 433.55 MHz, entrada GPIO34
7. Display: habilitar visualizações
8. Salvar tudo e reiniciar
```

### 3. Teste Console Serial (5 min)
```bash
# Abrir Serial Monitor (115200 baud)
[INFO] Booting ESP32APRS_LoRa...
[INFO] WiFi: Connecting to "Seu_WiFi"...
[INFO] WiFi: Connected! IP=192.168.31.81
[INFO] APRS-IS: Connecting to rotate.aprs2.net:14580...
[INFO] APRS-IS: Connected! Callsign=PU7IOL-1
[INFO] Radio: AFSK Mode, RX=433.55 MHz
[INFO] Ready!
```

### 4. Aguardar Transmissão Sonda (30s)
```bash
# No console serial, quando sonda transmitir:
[INFO] RX Audio Level: ████████░░ (80%)
[INFO] RX AFSK Decode: ••••••••
[INFO] RX: PU7IOL-1>APRNFW,WIDE2-1:!2317.52S/04638.33W>...
[INFO] Parsed: LAT=-23.292 LON=-46.639 ALT=3765m
[INFO] TX → APRS-IS: OK
[INFO] Visible at: https://aprs.fi/#!call=a%2FPU7IOL-1
```

### 5. Verificação Web (2 min)
```bash
1. Abrir: https://aprs.fi
2. Buscar: PU7IOL-1
3. Ver posição da sonda no mapa
4. Clicar na sonda → Ver detalhes:
   - Altitude: 3765 m
   - Velocidade: 12 km/h
   - Temperatura: -48°C
   - Bateria: 2.75V
   - Via: PU7IOL-1 (sua estação!)
```

✅ **Sucesso!** Seu iGate está funcionando e rastreando a sonda!

---

## ⚡ Dicas Rápidas

### Melhorar Recepção
- ✅ Use antena com ganho (Yagi, quad, collinear)
- ✅ Instale antena em local elevado (telhado, torre)
- ✅ Use cabo coaxial de baixa perda (LMR-400)
- ✅ Adicione LNA (Low Noise Amplifier) próximo à antena
- ✅ Use filtro passa-faixa 430-435 MHz

### Economizar Banda APRS-IS
- ✅ Use filtro específico: `b/PU7IOL-1` (apenas sua sonda)
- ✅ Habilite filtro de duplicatas
- ❌ Não use filtro muito amplo (ex: `p/P` = todos os brasileiros)

### Debug
- ✅ Console serial: informações detalhadas em tempo real
- ✅ Página Status: estatísticas e último pacote
- ✅ APRS.fi: verificar se pacotes chegaram ao servidor
- ✅ Ajustar RX Gain: 30-70 (ideal ~50)
- ✅ Ajustar volume receptor: 30-70% (ideal ~50%)

### Segurança
- ⚠️ Não habilite INET2RF sem autorização (transmissão)
- ⚠️ Verifique regulamentos ANATEL para sua classe
- ⚠️ Use potência mínima necessária se transmitir
- ⚠️ Nunca transmita em frequências não autorizadas

---

## 📞 Suporte

**Problemas comuns:**
1. **Não recebe pacotes**: Verificar frequência receptor, volume, conexões
2. **Recebe mas não envia APRS-IS**: Verificar conexão Internet, credenciais
3. **Pacotes com erros**: Ajustar RX Gain e volume do receptor
4. **Alcance curto**: Melhorar antena, adicionar LNA

**Contato:**
- Email: [seu_email]
- Callsign: PU7IOL
- GitHub: [abrir issue nos repositórios]

---

*Guia criado especificamente para:*
- *Sonda: RS41-NFW v65 (PU7IOL-1)*
- *iGate: ESP32APRS_LoRa (192.168.31.81)*
- *Protocolo: APRS 433.55 MHz*

**73!** 📡🎈
