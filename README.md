# Topologia Router-on-a-Stick

A topologia **router-on-a-stick** (roteador em um bastão) é uma arquitetura de rede que permite a comunicação entre **VLANs** (Redes Virtuais Locais) usando um único roteador físico com **sub-interfaces lógicas**. É uma solução econômica e simplificada para redes que necessitam de roteamento entre VLANs sem investir em switches de camada 3.

---

## Como Funciona?  
1. **Sub-interfaces Virtuais**: O roteador divide uma interface física em múltiplas sub-interfaces (ex: `Gi0/0.10`, `Gi0/0.20`), cada uma associada a uma VLAN.  
2. **Encapsulamento 802.1Q**: Cada sub-interface usa o protocolo **dot1Q** para identificar e rotular o tráfego de VLANs específicas.  
3. **Porta Trunk**: O switch conectado ao roteador configura uma porta como **trunk**, transportando tráfego de todas as VLANs para o roteador.  
4. **Roteamento Centralizado**: O roteador recebe pacotes de uma VLAN, remove a tag, roteia para a sub-interface de destino e reencaminha com a nova tag VLAN.  

---

## Características Principais  
- **Custo-Benefício**: Elimina a necessidade de múltiplos roteadores ou switches de camada 3.  
- **Escalabilidade Simples**: Adicione VLANs criando novas sub-interfaces, sem alterar cabos.  
- **Isolamento de Tráfego**: Mantém a segurança e a segmentação entre redes virtuais.  

---

## Vantagens vs. Desvantagens  
| **✔️ Prós**                     | **❌ Contras**                             |
| ------------------------------- | ----------------------------------------- |
| Redução de custos com hardware  | Largura de banda compartilhada (gargalos) |
| Fácil configuração e manutenção | Latência aumentada (processamento duplo)  |
| Ideal para redes pequenas       | Ponto único de falha (roteador)           |

---

## Aplicações Recomendadas  
- Redes com **até 10 VLANs** e tráfego leve (ex: escritórios, escolas).  
- Cenários onde a prioridade é **simplicidade**, não desempenho máximo.  
- Ambientes de teste ou laboratórios para estudo de VLANs e roteamento.  

---

## Melhores Práticas  

### 1. Planejamento da Rede
- **Defina as VLANs necessárias**: Agrupe dispositivos por função (ex: VLAN 10 para TI, VLAN 20 para Voz IP).  
- **Escolha uma interface de alta velocidade**: Prefira portas Gigabit Ethernet no roteador para minimizar gargalos.  
- **Atribua sub-redes lógicas**: Use blocos IP distintos para cada VLAN (ex: `192.168.10.0/24` para VLAN 10, `192.168.20.0/24` para VLAN 20).  

### 2. Priorização de Tráfego
- **Priorize o tráfego crítico**: Use QoS para dar prioridade a VoIP ou vídeo.  
- **Limite de largura de banda**: Restrinja o uso por VLAN para evitar congestionamento.  

### 3. Segurança
- **Filtre tráfego** com ACLs entre VLANs.  
- **Desative o trunking dinâmico**: `switchport nonegotiate`.  

### 4. Monitoramento e Otimização
- Verifique a tabela ARP do roteador: `show arp`.  
- Monitore a utilização da interface trunk: `show interfaces trunk`.  
- Ajuste MTU para 1500 bytes + 4 bytes da tag VLAN (1504 bytes).  

### 5. Cenários Ideais de Uso
- **Escritórios pequenos**: 20-50 dispositivos com tráfego web/email.  
- **Laboratórios educacionais**: Para ensino de conceitos de VLAN.  
- **Ambientes temporários**: Feiras ou eventos com rede efêmera.  

### 6. Quando Evitar
- Se o tráfego inter-VLAN ultrapassar **40% da capacidade do link**.  
- Em redes com requisitos de latência < 5ms.  
- Para mais de 15 VLANs simultâneas.  

---

## Configuração do Switch  

enable\
conf t
hostname S1

vlan 10
name "GERENCIA"
no shut

vlan 20
name "DADOS"
no shut
exit

interface "interface-ID"
switchport trunk encapsulation dot1q
switchport mode trunk
no shut

interface fast0/1
switchport mode access
switchport access vlan 20
no shut
exit

> **Nota**: O switch opera na camada 2, funcionando como ponte para encaminhar pacotes.

---

## Configuração do Roteador  

enable
conf t
hostname R1

interface "interface-ID"
no shut
exit

interface "interface-ID".10
description TRUNK_S1
encapsulation dot1Q 10
ip address 192.168.10.2 255.255.255.0
exit

interface "interface-ID".20
description DADOS
encapsulation dot1Q 20
ip address 192.168.20.2 255.255.255.0
exit

! Configuração DHCP para VLAN 20
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp pool DADOS
network 192.168.20.0 255.255.255.0
default-router 192.168.20.2
dns-server 8.8.8.8
end
wr

! Configuração DHCP para VLAN 10
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool GERENCIA
network 192.168.10.0 255.255.255.0
default-router 192.168.10.2
dns-server 8.8.8.8
end
wr

! Configuração de interface serial
interface serial "serial-ID"
ip address 200.0.0.1 255.255.255.252
no shut
end
wr

---

**Conclusão**: Essa configuração mantém a simplicidade operacional enquanto oferece roteamento básico entre VLANs, sendo uma solução econômica para cenários específicos. Para redes críticas ou de alto tráfego, recomenda-se switches de camada 3.
