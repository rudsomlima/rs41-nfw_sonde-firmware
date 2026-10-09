# 🎯 Resumo: RS41-NFW + ESP32APRS_LoRa

## ⚠️ AVISO CRÍTICO

**O ESP32APRS_LoRa NÃO pode receber diretamente o sinal APRS AFSK da sonda!**

O módulo LoRa (SX1276/SX1278) não suporta modulação AFSK (1200 baud).

---

## 🔌 Solução: Adicionar Receptor FM

```
┌──────────────────────────────────────────────────────────┐
│                     ARQUITETURA                          │
└──────────────────────────────────────────────────────────┘

[SONDA RS41-NFW v65]
        │
        │ TX 433.55 MHz
        │ APRS AFSK 1200 baud
        │ Intervalo: 30s
        │ Potência: 100mW
        ▼
   ┌─────────┐
   │ ANTENA  │ 433 MHz (vertical, ganho 3-6 dBi)
   └────┬────┘
        │
        ▼
 ┌──────────────┐
 │ RECEPTOR FM  │ Baofeng UV-5R, SA828, ou similar
 │  433.55 MHz  │ Modo: FM Narrow (NFM 25kHz)
 │ Squelch: OFF │ Volume: 50%
 └──────┬───────┘
        │
        │ Cabo de áudio (P2/P3)
        │ Saída de áudio demodulado
        ▼
  ┌─────────────────┐
  │ ESP32APRS_LoRa  │ GPIO34 (ADC input)
  │  TNC Software   │ Decodifica AFSK → APRS
  │ 192.168.31.81   │ Modo: AFSK/FM TNC
  └────────┬────────┘
           │
           │ WiFi → Internet
           ▼
    ┌──────────────┐
    │   APRS-IS    │ rotate.aprs2.net:14580
    │  aprs.fi     │ Rastreamento online
    └──────────────┘
```

---

## 📋 Hardware Necessário

| Item | Descrição | Preço Aprox. |
|------|-----------|--------------|
| **✅ Você já tem:** |
| Sonda RS41-NFW | Com firmware v65 configurado | - |
| ESP32APRS_LoRa | Com módulo LoRa | - |
| Antena 433 MHz | Para recepção | - |
| **❌ Você precisa adicionar:** |
| **Receptor FM** | Baofeng UV-5R, UV-82, ou SA828 | R$ 150-300 |
| Cabo de áudio | P2/P3 para conectar receptor→ESP32 | R$ 10-20 |
| **Ou Alternativa:** |
| **RTL-SDR** | RTL-SDR Blog V3 + antena | R$ 150-250 |
| Software | Direwolf (grátis) em PC/Raspberry | R$ 0 |

---

## ⚙️ Configuração Mínima ESP32APRS_LoRa

### Interface Web: http://192.168.31.81

```yaml
System:
  Callsign: PU7IOL
  SSID: 1
  Position: [Sua LAT/LON]
  Symbol: /& (Gateway)

iGate:
  Enable: true
  RF2INET: true          # Enviar RF → Internet
  INET2RF: false         # NÃO transmitir
  Server: rotate.aprs2.net
  Port: 14580
  Filter: b/PU7IOL-1     # Apenas sua sonda

Radio:
  Mode: AFSK/FM (TNC)    # ← IMPORTANTE!
  RX Freq: 433.550 MHz
  Input: GPIO34 (ADC)
  RX Gain: 50            # Ajustar conforme necessário
  TX: Disabled

Display:
  Show RX: true
  Show TX: true
  Filter Duplicates: true
```

---

## 🔍 Configurações da Sonda (Atual)

### APRS ✅ (Compatível com ESP32APRS_LoRa + Receptor FM)
```cpp
Frequência: 433.55 MHz
Modulação: AFSK (Bell 202)
Taxa: 1200 baud
Intervalo: 30 segundos
Callsign: PU7IOL-1
Potência: 100mW (20dBm)
```

### Horus V3 ❌ (NÃO compatível)
```cpp
Frequência: 430.43 MHz
Modulação: 4FSK
Taxa: 100 baud
Intervalo: 15 segundos
Callsign: PU7IOL-1
Potência: 100mW (20dBm)
```
*Para Horus V3: Use RTL-SDR + Horus-GUI*

---

## ✅ Passo-a-Passo (30 minutos)

### 1️⃣ Conectar Hardware (5 min)
```
1. Receptor FM → sintonizar 433.550 MHz
2. Receptor FM → modo FM Narrow (NFM)
3. Receptor FM → Squelch ABERTO
4. Receptor FM → Volume ~50%
5. Cabo áudio: Receptor (saída) → ESP32 GPIO34
```

### 2️⃣ Configurar ESP32 Web (15 min)
```
1. Acessar: http://192.168.31.81
2. System → Configurar callsign PU7IOL-1
3. iGate → Habilitar RF2INET
4. Radio → Modo AFSK/FM, 433.55 MHz, GPIO34
5. Salvar tudo → Reiniciar
```

### 3️⃣ Testar (10 min)
```
1. Monitor Serial (115200 baud)
2. Aguardar transmissão sonda (30s)
3. Console: "RX: PU7IOL-1>APRNFW..."
4. Console: "TX APRS-IS: OK"
5. Verificar: https://aprs.fi → buscar PU7IOL-1
```

---

## 🧪 Teste Sem Sonda (Validar Setup)

Use celular com app APRS para testar:

1. **Android**: APRSdroid
2. **iOS**: PocketPacket

Configure app:
- Callsign: `TEST-1`
- Frequência: `433.55 MHz` (via receptor FM externo)
- Transmitir: pacote de teste

Se ESP32 receber o teste, está funcionando! ✅

---

## 📡 Alternativa Completa: RTL-SDR

Se não quiser usar receptor FM:

```
[RTL-SDR Dongle] → [PC/RaspberryPi] → [APRS-IS]
      433.55 MHz      Direwolf TNC      Internet
```

**Vantagens:**
- ✅ Mais barato que receptor FM
- ✅ Mais flexível (múltiplas frequências)
- ✅ Melhor decodificação (filtros digitais)

**Desvantagens:**
- ❌ Precisa de PC/Raspberry rodando 24/7
- ❌ Mais complexo de configurar

**Software:**
```bash
# Linux/Raspberry Pi
sudo apt install rtl-sdr direwolf
direwolf -c sonde.conf
```

**sonde.conf:**
```ini
ADEVICE plughw:1,0
CHANNEL 0
MYCALL PU7IOL-1
MODEM 1200
IGSERVER rotate.aprs2.net
IGLOGIN PU7IOL-1 12345
FILTER b/PU7IOL-1
```

---

## 🎯 Qual Solução Escolher?

### Opção A: ESP32 + Receptor FM
**Melhor para:** Quem já tem ESP32APRS_LoRa
```
Custo adicional: ~R$ 150 (Baofeng)
Complexidade: Média
Confiabilidade: Alta
Alcance: Bom (depende da antena)
```

### Opção B: RTL-SDR + Direwolf
**Melhor para:** Setup novo ou quem prefere SDR
```
Custo total: ~R$ 150-250
Complexidade: Média-Alta
Confiabilidade: Muito Alta
Alcance: Muito Bom (melhor sensibilidade)
```

### Opção C: RTL-SDR + Horus-GUI (Dual Protocol)
**Melhor para:** Rastreamento profissional
```
Setup 1: RTL-SDR @ 433.55 → APRS
Setup 2: RTL-SDR @ 430.43 → Horus V3
Custo: ~R$ 300 (2× RTL-SDR)
Complexidade: Alta
Dados: Máximo (APRS + Horus V3)
```

---

## 📊 Comparação de Protocolos

| Aspecto | APRS (433.55 MHz) | Horus V3 (430.43 MHz) |
|---------|-------------------|------------------------|
| **Modulação** | AFSK 1200 baud | 4FSK 100 baud |
| **Intervalo** | 30 segundos | 15 segundos |
| **FEC** | ❌ Não | ✅ Golay (23,12) |
| **Alcance** | Bom | Melhor (FEC) |
| **Receptor** | FM comum ou SDR | Apenas SDR |
| **Software** | TNC (Direwolf, etc) | Horus-GUI |
| **Upload** | APRS-IS (aprs.fi) | SondeHub.org |
| **ESP32 Compatível** | ⚠️ Com receptor FM | ❌ Não |

**Recomendação:** Use **APRS** para simplicidade, ou **ambos** para redundância.

---

## 🚨 Erros Comuns

### 1. "Console não mostra pacotes"
```
Causa: Receptor não sintonizado ou sem áudio
Solução:
  - Verificar frequência: 433.550 MHz (exata!)
  - Verificar modo: FM Narrow
  - Verificar squelch: ABERTO
  - Testar áudio: deve ouvir "bip-bip" da sonda
```

### 2. "Pacotes com erro de checksum"
```
Causa: Nível de áudio incorreto
Solução:
  - Diminuir volume do receptor
  - Ajustar RX Gain no ESP32 (30-70)
  - Verificar cabo: pode estar com mau contato
```

### 3. "Recebe mas não envia para APRS-IS"
```
Causa: Sem conexão com servidor APRS-IS
Solução:
  - Verificar WiFi conectado
  - Verificar servidor: rotate.aprs2.net:14580
  - Verificar firewall: porta 14580 aberta
```

### 4. "Alcance muito curto"
```
Causa: Antena ruim ou obstruções
Solução:
  - Usar antena com ganho (Yagi, collinear)
  - Instalar em local elevado
  - Linha de visada desobstruída
  - Adicionar LNA (+20dB ganho)
```

---

## 📈 Alcance Esperado

### Fatores:
- Altura da sonda: 0-35 km
- Potência sonda: 100mW (20dBm)
- Antena receptora: ganho 3-6 dBi
- Local: urbano vs. rural
- Frequência: 433 MHz

### Estimativas:
```
Sonda a 1 km altitude:  ≈ 10-30 km alcance
Sonda a 5 km altitude:  ≈ 50-100 km alcance
Sonda a 10 km altitude: ≈ 100-200 km alcance
Sonda a 30 km altitude: ≈ 300-500 km alcance
```

**Linha de visada é crítica!**

```
          ★ Sonda @ 10km
         /│\
        / │ \
       /  │  \ Visada direta
      /   │   \
     /    │    \
    /     │     \
[Montanha]│    [Você]
```

---

## 🔗 Links Rápidos

### Rastreamento
- **APRS.fi**: https://aprs.fi/#!call=a%2FPU7IOL-1
- **SondeHub**: https://sondehub.org

### Código/Documentação
- **ESP32APRS_LoRa**: https://github.com/nakhonthai/ESP32APRS_LoRa
- **RS41-NFW**: https://github.com/sp5fra/rs41-nfw
- **Direwolf**: https://github.com/wb2osz/direwolf
- **Horus-GUI**: https://github.com/projecthorus/horus-gui

### Calculadoras
- **Alcance RF**: https://www.everythingrf.com/rf-calculators/line-of-sight-calculator
- **APRS Passcode**: https://apps.magicbug.co.uk/passcode/

---

## 📞 Próximos Passos

### Agora (Urgente):
1. ✅ Ler esta documentação
2. ✅ Decidir: Receptor FM ou RTL-SDR?
3. ✅ Comprar hardware necessário
4. ✅ Configurar ESP32APRS_LoRa
5. ✅ Testar com transmissão da sonda

### Depois (Melhorias):
6. ⬜ Adicionar LNA para maior alcance
7. ⬜ Instalar antena em local elevado
8. ⬜ Configurar backup (RTL-SDR + Horus V3)
9. ⬜ Documentar voos e compartilhar dados

---

## 💡 Dica Final

**Setup Mínimo Recomendado:**
```
Hardware:
  - ESP32APRS_LoRa (já tem)
  - Baofeng UV-5R (R$ 150)
  - Cabo P2 (R$ 10)
  - Antena 433 MHz (já tem)

Configuração:
  - 30 minutos
  
Resultado:
  - iGate funcional
  - Rastreamento em aprs.fi
  - Alcance 50-200 km (dependendo altitude sonda)
```

**Ou se preferir SDR:**
```
Hardware:
  - RTL-SDR Blog V3 (R$ 150)
  - Raspberry Pi (R$ 300) ou PC existente
  - Antena 433 MHz (já tem)

Configuração:
  - 1-2 horas (mais complexo)
  
Resultado:
  - iGate + decodificador Horus V3
  - Melhor sensibilidade
  - Dual protocol (APRS + Horus)
```

---

**Documentação completa:** Veja arquivos:
- `CONFIGURACAO_ESP32APRS_LoRa.md` - Detalhes técnicos completos
- `GUIA_RAPIDO_ESP32APRS.md` - Interface web passo-a-passo

**Dúvidas?** Abra issue no GitHub ou entre em contato.

**73 e bons rastreamentos!** 📡🎈

---

*Criado para: RS41-NFW v65 + ESP32APRS_LoRa*  
*Callsign: PU7IOL-1*  
*Última atualização: 2024*
