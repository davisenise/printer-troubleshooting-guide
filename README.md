# Guia de Diagnóstico — Problemas Comuns em Impressoras

**Autor:** Davi Senise
**Cargo:** Técnico de Suporte TI (N1/N2)
**Repositório:** printer-troubleshooting-guide

---

## Objetivo

Referência rápida pros chamados mais recorrentes de impressora em ambiente corporativo — rede, driver, fila de impressão, hardware. Cada seção segue: **Sintoma → Causa provável → Como confirmar → Solução → Como evitar de novo.**

---

## Índice

1. [Impressora não aparece na lista / não é encontrada](#1-impressora-não-aparece-na-lista--não-é-encontrada)
2. [Trabalho fica preso na fila de impressão](#2-trabalho-fica-preso-na-fila-de-impressão)
3. [Impressora aparece offline mesmo ligada](#3-impressora-aparece-offline-mesmo-ligada)
4. [Erro de driver / spooler trava](#4-erro-de-driver--spooler-trava)
5. [Qualidade de impressão ruim](#5-qualidade-de-impressão-ruim)
6. [Impressora compartilhada não funciona pra outros usuários](#6-impressora-compartilhada-não-funciona-pra-outros-usuários)
7. [Checklist rápido de coleta de dados](#7-checklist-rápido-de-coleta-de-dados)

---

## 1. Impressora não aparece na lista / não é encontrada

**Sintoma:** Usuário tenta adicionar a impressora e ela não aparece na busca de rede.

**Causas prováveis:**
- Impressora em VLAN/sub-rede diferente do PC
- Serviço de descoberta de rede desligado no Windows
- Impressora desligada ou sem IP fixo (perdeu o IP depois de reiniciar)

**Como confirmar:**
```
ping <IP da impressora>
```
Se não responder, ela não está acessível dessa rede — não adianta procurar via descoberta automática.

**Solução:**
- Confirma o IP atual da impressora (no painel dela ou por relatório de configuração impresso)
- Adiciona manualmente por IP: `Configurações > Impressoras > Adicionar > "A impressora que eu quero não está na lista" > Adicionar por endereço TCP/IP`
- Ativa "Descoberta de Rede" em Central de Redes, se estiver desligada

**Como evitar de novo:**
Configurar IP fixo (ou reserva de DHCP) pra impressora evita que o endereço mude e quebre conexões já configuradas.

---

## 2. Trabalho fica preso na fila de impressão

**Sintoma:** Documento enviado não imprime, fica "Em fila" ou "Imprimindo" indefinidamente e trava os próximos.

**Causas prováveis:**
- Spooler de impressão travado
- Arquivo de trabalho corrompido no spool
- Impressora sem papel/toner e ninguém percebeu

**Como confirmar:**
Verifica fisicamente a impressora primeiro (papel, toner, mensagem de erro no display) — resolve boa parte sem precisar mexer no PC.

**Solução (via CMD como administrador):**
```
net stop spooler
del /Q /F "%systemroot%\System32\spool\PRINTERS\*.*"
net start spooler
```
Isso reinicia o serviço de spool e limpa trabalhos travados. Depois disso, reenviar a impressão.

**Como evitar de novo:**
Se esse travamento é recorrente na mesma máquina/impressora, vale investigar driver desatualizado — costuma ser a causa raiz por trás do sintoma repetido.

---

## 3. Impressora aparece offline mesmo ligada

**Sintoma:** Windows mostra a impressora como "Offline", mas ela está ligada e conectada na rede.

**Causas prováveis:**
- Windows guardou um IP antigo da impressora (ela pegou um IP novo)
- Porta de impressão configurada errada
- Impressora em modo de economia de energia demorando pra responder

**Como confirmar:**
```
ping <IP configurado da impressora>
```
Compara com o IP real dela (visto no painel físico ou relatório de configuração). Se forem diferentes, achou a causa.

**Solução:**
- Atualiza o IP na porta: `Painel de Controle > Dispositivos e Impressoras > clique direito na impressora > Propriedades > Portas > selecionar a porta > Configurar Porta > atualizar IP`
- Ou simplesmente remove e readiciona a impressora com o IP correto

**Como evitar de novo:**
IP fixo/reserva de DHCP resolve esse problema de raiz — é a causa mais comum desse sintoma específico.

---

## 4. Erro de driver / spooler trava

**Sintoma:** Erro ao tentar imprimir, spooler trava com frequência, ou impressão nunca sai mesmo sem fila visível.

**Causas prováveis:**
- Driver corrompido ou incompatível com a versão do Windows
- Driver genérico instalado no lugar do driver correto do fabricante
- Conflito entre múltiplas versões de driver da mesma impressora instaladas

**Como confirmar:**
`Gerenciador de Dispositivos > Filas de impressão` — verifica se tem erro ou aviso amarelo no driver.

**Solução:**
1. Remove a impressora e o driver completamente (`Configurações > Impressoras > Remover dispositivo` + `Gerenciador de Impressão > Drivers > Remover driver`)
2. Baixa o driver correto direto do site do fabricante (nunca usa o genérico do Windows Update pra impressoras de uso profissional/multifuncional)
3. Reinstala do zero

**Como evitar de novo:**
Manter um repositório interno com os drivers corretos de cada modelo de impressora da empresa evita reinstalação às cegas toda vez que der problema.

---

## 5. Qualidade de impressão ruim

**Sintoma:** Impressão sai borrada, com listras, cor errada ou desbotada.

**Causas prováveis:**
- Cartucho/toner baixo ou com defeito
- Cabeça de impressão suja (impressoras jato de tinta)
- Configuração de qualidade de impressão baixa
- Papel incompatível com o tipo de impressão

**Como confirmar:**
Roda relatório de teste/diagnóstico direto no painel da impressora (opção varia por marca, geralmente em "Manutenção" ou "Configuração").

**Solução:**
- Jato de tinta: rodar limpeza de cabeça de impressão pelo painel ou driver
- Toner baixo: substituir (mesmo que ainda "imprima", qualidade cai antes de acabar de vez)
- Ajustar qualidade de impressão no driver pra "Alta" ou "Melhor"

**Como evitar de novo:**
Monitorar nível de suprimento (a maioria das impressoras de rede reporta isso via SNMP ou painel web) evita chamado de última hora.

---

## 6. Impressora compartilhada não funciona pra outros usuários

**Sintoma:** Um PC consegue imprimir direto, mas outros que dependem do compartilhamento não conseguem.

**Causas prováveis:**
- PC que compartilha a impressora está desligado (compartilhamento via PC, não via rede direta)
- Permissão de compartilhamento não inclui o usuário/grupo
- Firewall bloqueando a porta de compartilhamento (445/139)

**Como confirmar:**
Verifica se a impressora está compartilhada via um PC específico (`\\nome-do-pc\impressora`) ou direto na rede (IP próprio). Se for via PC, esse PC precisa estar ligado sempre.

**Solução:**
- Migra pra compartilhamento direto de rede (impressora com IP próprio) sempre que possível — elimina a dependência de um PC específico estar ligado
- Se precisar manter via PC: confirma permissões em `Propriedades da Impressora > Compartilhamento > Segurança`

**Como evitar de novo:**
Impressora compartilhada via PC individual é uma prática frágil pra ambiente corporativo — o ideal é sempre impressora de rede com IP próprio.

---

## 7. Checklist rápido de coleta de dados

- [ ] IP configurado no Windows vs. IP real da impressora
- [ ] Impressora liga e mostra mensagem de erro no display?
- [ ] `ping` no IP da impressora responde?
- [ ] Driver instalado é o do fabricante ou genérico?
- [ ] Erro acontece em um PC só ou em vários?
- [ ] Fila de impressão tem trabalho travado?

---

## Notas

Guia vivo — adicionar novas seções conforme padrões de falha aparecerem em atendimentos reais, seguindo a mesma estrutura.
