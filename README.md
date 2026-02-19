# Patchmon by Flpems - Zabbix Template

Zabbix template (exported for Zabbix 7.0+) to collect PatchMon data via
HTTP Agent and automatically create per-host items using LLD (Low-Level
Discovery).

------------------------------------------------------------------------

# 🇺🇸 English

## Overview

The template queries PatchMon host endpoints and dynamically discovers
hosts to collect:

-   Docker integration counters/status
-   System information (Kernel, SELinux, reboot pending)
-   Package statistics (installed, pending total updates, pending
    security updates, repositories)
-   OS/version (formatted via JavaScript preprocessing)
-   PatchMon API availability

Version 2 introduces an optimized **Master + Dependent item
architecture**, significantly reducing API calls and improving
scalability.

Additionally, items now use **Discard unchanged with heartbeat** to
prevent dashboard gaps when using larger time ranges.

------------------------------------------------------------------------

## Compatibility

-   Export format: Zabbix 7.0+
-   Collection method: HTTP Agent (Master items) + Dependent items
-   Hosts list interval: 1h
-   Per-host RAW endpoints interval: 30m

------------------------------------------------------------------------

## Requirements

-   PatchMon instance reachable from Zabbix Server or Zabbix Proxy
-   API credentials (key/user + secret)
-   Outbound HTTP/HTTPS allowed from Zabbix to PatchMon

### API endpoints used:

GET {$PM_URL}/api/v1/api/hosts\
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/integrations\
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/info\
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/system\
GET {\$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/stats

------------------------------------------------------------------------

## Required Macros

{$PM_URL} → PatchMon base URL {$PM_KEY} → API user/key (Basic Auth
username)\
{\$PM_SECRET} → API secret (Basic Auth password)

### Recommendations

-   Mark {\$PM_SECRET} as **Secret** in Zabbix
-   Keep {\$PM_URL} consistent (optionally without trailing `/`)

------------------------------------------------------------------------

## Installation

1.  Zabbix → Data collection → Templates → Import\
2.  Import the template YAML\
3.  Configure macros\
4.  Link template to a host

------------------------------------------------------------------------

## Credits

Template: Patchmon by Flpems\
Version: v2

---

# Patchmon by Flpems - Template Zabbix

Template Zabbix (exportado para Zabbix 7.0+) para coletar dados do
PatchMon via HTTP Agent e criar automaticamente itens por host
utilizando LLD (Low-Level Discovery).

------------------------------------------------------------------------

# 🇧🇷 Português Brasil

## Visão Geral

O template consulta os endpoints de hosts do PatchMon e descobre os
hosts dinamicamente para coletar:

-   Contadores/status da integração Docker
-   Informações de sistema (Kernel, SELinux, reboot pendente)
-   Estatísticas de pacotes (instalados, total de updates pendentes,
    updates de segurança pendentes, repositórios)
-   OS/versão (formatado via pré-processamento JavaScript)
-   Disponibilidade da API do PatchMon

A Versão 2 introduz uma arquitetura otimizada de **itens Master +
Dependent**, reduzindo significativamente as chamadas à API e melhorando
a escalabilidade.

Além disso, os itens agora utilizam **Discard unchanged with heartbeat**
para evitar lacunas em dashboards ao utilizar períodos de tempo maiores.

------------------------------------------------------------------------

## Compatibilidade

-   Formato de exportação: Zabbix 7.0+
-   Método de coleta: HTTP Agent (itens Master) + Itens Dependentes
-   Intervalo da lista de hosts: 1h
-   Intervalo dos endpoints RAW por host: 30m

------------------------------------------------------------------------

## Requisitos

-   Instância do PatchMon acessível a partir do Zabbix Server ou Zabbix
    Proxy
-   Credenciais da API (key/usuário + secret)
-   Saída HTTP/HTTPS liberada do Zabbix para o PatchMon

### Endpoints da API utilizados:

GET {$PM_URL}/api/v1/api/hosts\
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/integrations\
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/info\
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/system\
GET {\$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/stats

------------------------------------------------------------------------

## Macros Obrigatórias

{$PM_URL} → URL base do PatchMon {$PM_KEY} → Usuário/key da API
(username do Basic Auth)\
{\$PM_SECRET} → Secret da API (senha do Basic Auth)

### Recomendações

-   Marque {\$PM_SECRET} como **Secret** no Zabbix
-   Mantenha {\$PM_URL} consistente (opcionalmente sem `/` no final)

------------------------------------------------------------------------

## Instalação

1.  Zabbix → Data collection → Templates → Importar\
2.  Importe o YAML do template\
3.  Configure as macros\
4.  Vincule o template a um host

------------------------------------------------------------------------

## Créditos

Template: Patchmon by Flpems\
Versão: v2
