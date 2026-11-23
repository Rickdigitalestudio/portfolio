# Diagnóstico e Solução para Adaptador Wi-Fi GotWire que não conecta no Linux Manjaro

## Resumo do Problema

O seu adaptador Wi-Fi (provavelmente o de Vendor ID 0a5c:bd27, identificado como "Remote Download Wireless Adapter") está sendo detectado pelo kernel, mas está **preso no modo de download de firmware**. Isso impede que o driver brcmfmac inicialize o dispositivo como adaptador wireless normal, por isso não aparece nenhuma interface `wlan` e não é possível conectar-se a redes.

## O que foi encontrado no seu sistema

Dois adaptadores USB Wi-Fi estão presentes:

### 1. Adaptador MT7601U (Ralink/MediaTek)
- **Bus**: 3, Device: 18
- **Driver**: `mt7601u` (carregado)
- **Capacidade**: Apenas 2.4GHz (802.n b/g/n), velocidade máxima 150 Mbps
- **Status**: Conectado à rede "EveRick" na banda 2.4GHz (canal 10)
- **Interface**: `wlp9s0f3u1u1u1`
- **Nota**: Esse adaptador **não suporta 5GHz**. Se o seu adaptador é de 5GHz dual-band 1300Mbps, este não é o dispositivo correto.

### 2. Adaptador Broadcom 0a5c:bd27 (Provavelmente o seu adaptador GotWire)
- **Bus**: 1, Device: 22
- **Produto**: "Remote Download Wireless Adapter"
- **Driver esperado**: `brcmfmac` (driver Broadcom fullmac)
- **Problema**: O dispositivo está **preso no modo de download de firmware** (DFU mode). O kernel identifica como classe "Vendor Specific" e não como interface de rede wireless.
- **Consequência**: Nenhuma interface `wlan` é criada, o NetworkManager não consegue ver redes Wi-Fi desse adaptador.

## Causa Provável

O modo "Remote Download" é um estado de recuperação do firmware. Isso normalmente acontece quando:

1. **Atualização de firmware interrompida**: O Windows pode ter iniciado uma atualização de firmware do adaptador e o processo foi interrompido (desconexão acidental, energia instável, reinicialização).
2. **Corrupção de firmware**: O firmware do adaptador ficou danificado, fazendo o dispositivo entrar em modo de download persistente.
3. **Incompatibilidade de driver**: Após atualizações do kernel do Manjaro, o dispositivo não voltou ao modo operacional normal.

## Soluções Possíveis

### Solução 1: Tentar recuperar o adaptador Broadcom (recomendado)

Esse é o método para tentar restituir o adaptador ao modo normal sem precisar comprar novo hardware.

**Passo 1: Reset do dispositivo USB**

Tente reiniciar o adaptador removendo e reconectando:

```bash
# Verificar a quais portas o adaptador está conectado
lsusb | grep -i broadcom

# Desconectar fisicamente o adaptador, aguardar 5 segundos e reconectar
```

**Passo 2: Forçar a carga do driver brcmfmac com firmware correto**

O driver `brcmfmac` já tem suporte ao seu dispositivo (VID:PID 0a5c:bd27 está na lista de aliases), mas precisa do firmware correto.

1. Verifique se há firmware adequado:
   ```bash
   ls /lib/firmware/brcm/
   ```
   
2. O arquivo `BCM-0a5c-6410.hcd.zst` existente é para um produto diferente (0a5c:6410). Pode ser necessário obter o firmware correto para 0a5c:bd27.

3. Tente carregar o driver manualmente (como root):
   ```bash
   sudo modprobe brcmfmac
   ```
   
4. Se o driver carregar, a interface deve aparecer:
   ```bash
   ip link show
   ```

**Passo 3: Usar Windows para re-flash do firmware (se disponível)**

Se houver um utilitário da GotWire (ou fabricante) no Windows, use-o para re-flashar o firmware do adaptador. Isso geralmente resolve o problema do modo "Remote Download". Após isso, conecte o adaptador no Manjaro e ele deverá ser reconhecido corretamente.

### Solução 2: Usar o adaptador MT7601U (apenas 2.4GHz)

Se você não depende exclusivamente da banda 5GHz, o adaptador MT7601U já está funcionando:

- Sua interface atual é `wlp9s0f3u1u1u1`
- Está conectado à rede "EveRick" na banda 2.4GHz
- Velocidade de transferência: 150 MBit/s

Para usar essa rede:
```bash
nmcli device wlp9s0f3u1u1u1 status
```

**Limitação**: Apenas 2.4GHz, velocidade máxima 150 Mbps. Não suporta 5GHz nem 1300 Mbps.

### Solução 3: Comprar um novo adaptador Wi-Fi 5GHz dual-band

Se o adaptador Broadcom não puder ser recuperado, recomendo comprar um novo adaptador compatível com Linux. Alguns modelos conhecidos por funcionarem bem no Manjaro/Kernel 6.x+:

| Modelo | Chipset | Preço Approx. | Compatibilidade |
|--------|---------|---------------|-----------------|
| Realtek RTL8812AU | RTL8812AU | R$ 80-120 | Excelente, modo AP e monitor |
| Realtek RTL8821AU | RTL8821AU | R$ 90-130 | Excelente |
| MediaTek MT7612U | MT7612U | R$ 70-110 | Bom suporte no kernel 5.15+ |
| TP-Link Archer T2U Plus | MT7610U | R$ 60-90 | Bom suporte |

**O que verificar na embalagem**:
- "Linux Compatible" ou suporte ao kernel 5.15+
- Chipset Realtek RTL8812AU/8821AU ou MediaTek MT7612U/MT7603U
- Evite adaptadores com chipset Atheros QCA9377 (muitas vezes requer firmware proprietário)

## Passos Imediatos para Testar

1. **Desconecte e reconecte** o adaptador GotWire em uma porta USB 2.0 (as vezes o USB 3.0 causa reinicializações inesperadas).

2. **Como root**, tente:
   ```bash
   modprobe brcmfmac
   ```
   Se der "Operation not permitted", reinicie o computador e entre no modo root ou use `sudo` corretamente.

3. **Verifique o dmesg** após reconectar:
   ```bash
   dmesg | tail -20
   ```
   Procure por mensagens como `brcmfmac: brcmf_cdc_send_msg: error` ou `firmware request`.

4. **Use o NetworkManager** após o driver ser carregado:
   ```bash
   nmcli device status
   ```

## Conclusão

O problema principal é que seu adaptador Wi-Fi GotWire (Broadcom 0a5c:bd27) entrou em modo de download de firmware e não inicializa como adaptador wireless normal. As chances de recuperação são boas se:

1. O firmware não estiver fisicamente queimado.
2. For possível re-flashá-lo usando o utilitário Windows do fabricante.

Se isso não funcionar, a Solução 3 (novo adaptador) é o caminho mais confiável para obter funcionalidade Wi-Fi 5GHz dual-band no Linux Manjaro.

---

**Informações do sistema**:
- Kernel: 7.2.9-1-MANJARO
- Distribuição: Manjaro Linux
- Adaptador funcional: MT7601U (2.4GHz apenas)
- Adaptador problemático: Broadcom 0a5c:bd27 (modo download)
- Driver disponível: brcmfmac (compatível com 0a5c:bd27)