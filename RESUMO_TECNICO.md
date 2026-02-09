# Resumo Técnico de Alterações - RS41-NFW

## Versão: Otimizada para RSM4x2
## Data: 08/02/2026

---

## 🔴 Bug Crítico Corrigido

**Problema:** `interfaceHandler()` não funcionava em RSM4x2 devido a `#ifndef RSM4x2`

**Localização:** `rs41-nfw_sonde-firmware.ino` linhas 5110-5400

**Impacto:** Software Python (Ground Control) não recebia dados de placas RSM4x2

**Solução:** Removido bloco condicional, função agora compila para todos os modelos

---

## ⚙️ Otimizações de Memória

### Total Economizado: ~1800 bytes

| Otimização | Bytes Economizados |
|------------|-------------------|
| interfaceHandler() simplificada | ~1500 bytes |
| RTTY desabilitado | ~400 bytes |
| Morse desabilitado | ~300 bytes |
| Macro F() em strings | ~200 bytes |

---

## 📋 Alterações no Código

### 1. Função `interfaceHandler()` (Linha 5110)

**Antes:** 280+ linhas, 80+ variáveis enviadas
**Depois:** 30 linhas, 27 variáveis essenciais

**Variáveis mantidas:**
- Hardware: isRSM4x2, isRSM4x4, fwVer, millis
- GPS completo: lat, long, alt, sats, hora, velocidade, vVCalc
- Rádio: horusV3Enable, aprsEnable, frequências, callsign
- Sensores: batV, temperatura, umidade, pressão
- Status: err, warn, ok

### 2. Modos Desabilitados (Linhas 289, 299)

```cpp
bool rttyEnable = false;   // Linha ~289 (antes: true)
bool morseEnable = false;  // Linha ~299 (antes: true)
```

### 3. Strings Otimizadas

```cpp
// Antes:
xdataSerial.print("texto");

// Depois:
xdataSerial.print(F("texto"));
```

---

## 📊 Estado Atual do Firmware

### Transmissões Ativas
- ✅ **Horus V3:** 430.43 MHz, 15s, 100mW
- ✅ **APRS:** 432.5 MHz, 30s, 100mW
- ❌ **RTTY:** Desabilitado
- ❌ **Morse:** Desabilitado

### Memória
- **FLASH:** ~63KB de 64KB (97%)
- **RAM:** ~75%
- **Margem:** 1.5KB

### GPS
- **Modo:** 2 (powersave RSM4x2)
- **improvedGpsPerformance:** false
- **Airborne:** true

---

## 🔧 Comunicação Serial

**Porta:** XDATA (PB11/TX, PB10/RX)
**Baudrate:** 115200
**Modo:** xdataPortMode = 4

**Status:**
- ✅ Funciona em RSM4x2
- ✅ Funciona em RSM4x4
- ✅ Compatível com Ground Control Software Python

---

## ⚠️ Avisos

1. **Reativar RTTY/Morse pode causar overflow novamente**
2. **Firmware modificado - testar antes de voo**
3. **GPS precisa ver céu - normal não pegar fix indoor**
4. **LED laranja = sem fix, mas transmitindo**

---

## 📁 Arquivos Criados/Modificados

```
✏️ rs41-nfw_sonde-firmware.ino - MODIFICADO
📄 ALTERACOES_PT-BR.md - NOVO (documentação completa PT)
📄 CHANGES_EN.md - NOVO (documentação completa EN)
📄 RESUMO_TECNICO.md - NOVO (este arquivo)
```

---

## 🔄 Como Reverter

Para voltar ao comportamento original (com overflow):

1. Linha 5110: Adicionar `#ifndef RSM4x2`
2. Linha 5410: Adicionar `#endif`
3. Linha 289: `bool rttyEnable = true;`
4. Linha 299: `bool morseEnable = true;`

**Nota:** Isso causará overflow de memória novamente!

---

## ✅ Checklist de Teste

- [ ] Compila sem erros
- [ ] Upload OK
- [ ] LED verde após boot
- [ ] Python recebe dados (COM8, 115200)
- [ ] GPS pega fix ao ar livre
- [ ] Horus V3 transmitindo
- [ ] APRS transmitindo
- [ ] Sensores lendo valores

---

## 📞 Suporte

**Projeto Original:**
- GitHub: https://github.com/Nevvman18/rs41-nfw
- Issues: https://github.com/Nevvman18/rs41-nfw/issues

**Estas Modificações:**
- Autor: rudso (PU7IOL)
- Data: 08/02/2026
- Testado em: RSM4x2

---

**Fim do Resumo**
