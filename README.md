Patchmon by Flpems — Zabbix Template

Zabbix template (exported for Zabbix 7.0+) to collect PatchMon data via HTTP Agent and automatically create per-host items using LLD (Low-Level Discovery).

🇺🇸 English
Overview

The template queries PatchMon host endpoints and discovers hosts dynamically to collect:

Docker integration counters/status

System information (Kernel, SELinux, reboot pending)

Package statistics (installed, pending total updates, pending security updates, repositories)

OS/version (formatted via JavaScript preprocessing)

PatchMon API availability

Version 2 introduces an optimized Master + Dependent item architecture, reducing API calls and improving scalability.

Compatibility

Export format: Zabbix 7.0+

Collection method: HTTP Agent (Master items) + Dependent items

Hosts list interval: 1h

Per-host RAW endpoints interval: 30m

Requirements

PatchMon instance reachable from Zabbix Server or Zabbix Proxy

API credentials (key/user + secret)

Outbound HTTP/HTTPS allowed from Zabbix to PatchMon

API endpoints used by this template:

GET {$PM_URL}/api/v1/api/hosts
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/integrations
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/info
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/system
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/stats

Macros (required)

Set these macros on the template (or override at host/host group level):

{$PM_URL}    → PatchMon base URL
{$PM_KEY}    → API user/key (Basic Auth username)
{$PM_SECRET} → API secret (Basic Auth password)


Recommendations:

Mark {$PM_SECRET} as Secret in Zabbix

Keep {$PM_URL} consistent (optionally without trailing /)

How it works
Master item (hosts list)

PatchMon - Hosts list (raw)
Key: patchmon.hosts.raw
Type: HTTP Agent (TEXT)
URL: {$PM_URL}/api/v1/api/hosts
Auth: Basic ({$PM_KEY} / {$PM_SECRET})
Interval: 1h

Retrieves the full JSON payload containing the hosts list.

Discovery (LLD)

PatchMon - Host discovery
Key: patchmon.host.discovery
Type: Dependent
Master item: patchmon.hosts.raw
Preprocessing: JSONPath $.hosts[*]

Extracted LLD macros:

{#PM_HOSTID} → $.id

{#PM_HOSTNAME} → $.hostname

{#PM_IP} → $.ip

{#PM_FRIENDLY} → $.friendly_name

Per-host items (v2 architecture)

For each discovered host, v2 creates four RAW master items (HTTP Agent, 30m):

patchmon.host.info.raw

patchmon.host.system.raw

patchmon.host.stats.raw

patchmon.host.integrations.raw

All other metrics are Dependent items, using:

JSONPath

JavaScript preprocessing

DISCARD_UNCHANGED (where applicable)

This reduces the number of HTTP requests per host and improves performance.

Docker (endpoint /integrations)

Docker Container Count
Key: patchmon.host.dckcnt[{#PM_HOSTID}]
JSONPath: $.integrations.docker.containers_count

Docker Integration Status
Key: patchmon.host.dckint[{#PM_HOSTID}]
Type: TEXT
JSONPath: $.integrations.docker.enabled

Docker Networks Count
Key: patchmon.host.dckntwcnt[{#PM_HOSTID}]
JSONPath: $.integrations.docker.networks_count

Docker Volumes Count
Key: patchmon.host.dckvlcnt[{#PM_HOSTID}]
JSONPath: $.integrations.docker.volumes_count

If Docker integration is disabled, items return -1 (custom error handler).
Value map included:

-1 → Integration disabled

System info (endpoints /info and /system)

System OS/Version
Key: patchmon.host.info[{#PM_HOSTID}]
Type: TEXT
Preprocessing: JavaScript (concatenates os_type + os_version)

Kernel Version
Key: patchmon.host.krnversion[{#PM_HOSTID}]
JSONPath: $.kernel_version

SELinux Status
Key: patchmon.host.selstatus[{#PM_HOSTID}]
JSONPath: $.selinux_status

Reboot Status
Key: patchmon.host.rbtstatus[{#PM_HOSTID}]
JSONPath: $.needs_reboot

STR_REPLACE:

false → No reboot pending
true  → Reboot pending

Package stats (endpoint /stats)

Installed Packages
Key: patchmon.host.stats[{#PM_HOSTID}]
JSONPath: $.total_installed_packages

Pending Updates: Total
Key: patchmon.host.pending_updates[{#PM_HOSTID}]
JSONPath: $.outdated_packages

Pending Updates: Security
Key: patchmon.host.sec_updates[{#PM_HOSTID}]
JSONPath: $.security_updates

Total Repositories
Key: patchmon.host.repos[{#PM_HOSTID}]
JSONPath: $.total_repos

Note: several items use DISCARD_UNCHANGED to reduce storage when values do not change.

Triggers (prototypes)

Security updates (HIGH)

{#PM_HOSTNAME} has more than 10 pending security updates

last(/Patchmon by Flpems/patchmon.host.sec_updates[{#PM_HOSTID}])>=10


Total updates (AVERAGE)

{#PM_HOSTNAME} has more than 20 pending updates

last(/Patchmon by Flpems/patchmon.host.pending_updates[{#PM_HOSTID}])>=20


Dependency:
Depends on the security-updates trigger.

API Availability (NEW in v2)
Patchmon API availability: no data for 2h

nodata(/Patchmon by Flpems/patchmon.hosts.raw,2h)=1

Installation

Zabbix → Data collection → Templates → Import

Import the template YAML

Configure macros:

{$PM_URL}

{$PM_KEY}

{$PM_SECRET}

Link template to a host

Wait for:

patchmon.hosts.raw to receive JSON

Discovery to create per-host items

Security

Do not store {$PM_SECRET} in plain text outside Zabbix

Use Zabbix Proxy if PatchMon is not directly reachable

Credits

Template: Patchmon by Flpems
Version: v2

---

Patchmon by Flpems — Zabbix Template

Template Zabbix (exportado para Zabbix 7.0+) para coletar dados do PatchMon via HTTP Agent e criar automaticamente itens por host utilizando LLD (Low-Level Discovery).

🇧🇷 Português Brasil
Visão Geral

O template consulta os endpoints de hosts do PatchMon e descobre os hosts dinamicamente para coletar:

Contadores/status da integração Docker

Informações de sistema (Kernel, SELinux, reboot pendente)

Estatísticas de pacotes (instalados, total de updates pendentes, updates de segurança pendentes, repositórios)

OS/versão (formatado via pré-processamento JavaScript)

Disponibilidade da API do PatchMon

A Versão 2 introduz uma arquitetura otimizada de itens Master + Dependent, reduzindo chamadas à API e melhorando a escalabilidade.

Compatibilidade

Formato de exportação: Zabbix 7.0+

Método de coleta: HTTP Agent (itens Master) + Itens Dependentes

Intervalo da lista de hosts: 1h

Intervalo dos endpoints RAW por host: 30m

Requisitos

Instância do PatchMon acessível a partir do Zabbix Server ou Zabbix Proxy

Credenciais da API (key/usuário + secret)

Saída HTTP/HTTPS liberada do Zabbix para o PatchMon

Endpoints da API utilizados por este template:

GET {$PM_URL}/api/v1/api/hosts
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/integrations
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/info
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/system
GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/stats

Macros (obrigatórias)

Configure estas macros no template (ou sobrescreva em nível de host/grupo de hosts):

{$PM_URL} → URL base do PatchMon
{$PM_KEY} → Usuário/key da API (username do Basic Auth)
{$PM_SECRET} → Secret da API (senha do Basic Auth)

Recomendações:

Marque {$PM_SECRET} como Secret no Zabbix

Mantenha {$PM_URL} consistente (opcionalmente sem / no final)

Como funciona
Item Master (lista de hosts)

PatchMon - Hosts list (raw)
Key: patchmon.hosts.raw
Tipo: HTTP Agent (TEXT)
URL: {$PM_URL}/api/v1/api/hosts
Auth: Basic ({$PM_KEY} / {$PM_SECRET})
Intervalo: 1h

Recupera o payload JSON completo contendo a lista de hosts.

Discovery (LLD)

PatchMon - Host discovery
Key: patchmon.host.discovery
Tipo: Dependent
Item master: patchmon.hosts.raw
Pré-processamento: JSONPath $.hosts[*]

Macros LLD extraídas:

{#PM_HOSTID} → $.id

{#PM_HOSTNAME} → $.hostname

{#PM_IP} → $.ip

{#PM_FRIENDLY} → $.friendly_name

Itens por host (arquitetura v2)

Para cada host descoberto, a v2 cria quatro itens master RAW (HTTP Agent, 30m):

patchmon.host.info.raw

patchmon.host.system.raw

patchmon.host.stats.raw

patchmon.host.integrations.raw

Todas as demais métricas são Itens Dependentes, utilizando:

JSONPath

Pré-processamento JavaScript

DISCARD_UNCHANGED (quando aplicável)

Isso reduz o número de requisições HTTP por host e melhora a performance.

Docker (endpoint /integrations)

Quantidade de Containers Docker
Key: patchmon.host.dckcnt[{#PM_HOSTID}]
JSONPath: $.integrations.docker.containers_count

Status da Integração Docker
Key: patchmon.host.dckint[{#PM_HOSTID}]
Tipo: TEXT
JSONPath: $.integrations.docker.enabled

Quantidade de Redes Docker
Key: patchmon.host.dckntwcnt[{#PM_HOSTID}]
JSONPath: $.integrations.docker.networks_count

Quantidade de Volumes Docker
Key: patchmon.host.dckvlcnt[{#PM_HOSTID}]
JSONPath: $.integrations.docker.volumes_count

Se a integração Docker estiver desabilitada, os itens retornam -1 (handler de erro customizado).

Value map incluído:

-1 → Integração desabilitada

Informações de sistema (endpoints /info e /system)

Sistema Operacional/Versão
Key: patchmon.host.info[{#PM_HOSTID}]
Tipo: TEXT
Pré-processamento: JavaScript (concatena os_type + os_version)

Versão do Kernel
Key: patchmon.host.krnversion[{#PM_HOSTID}]
JSONPath: $.kernel_version

Status do SELinux
Key: patchmon.host.selstatus[{#PM_HOSTID}]
JSONPath: $.selinux_status

Status de Reboot
Key: patchmon.host.rbtstatus[{#PM_HOSTID}]
JSONPath: $.needs_reboot

STR_REPLACE:

false → Sem reboot pendente
true → Reboot pendente

Estatísticas de pacotes (endpoint /stats)

Pacotes Instalados
Key: patchmon.host.stats[{#PM_HOSTID}]
JSONPath: $.total_installed_packages

Updates Pendentes: Total
Key: patchmon.host.pending_updates[{#PM_HOSTID}]
JSONPath: $.outdated_packages

Updates Pendentes: Segurança
Key: patchmon.host.sec_updates[{#PM_HOSTID}]
JSONPath: $.security_updates

Total de Repositórios
Key: patchmon.host.repos[{#PM_HOSTID}]
JSONPath: $.total_repos

Observação: vários itens utilizam DISCARD_UNCHANGED para reduzir armazenamento quando os valores não mudam.

Triggers (protótipos)

Updates de segurança (HIGH)

{#PM_HOSTNAME} possui mais de 10 updates de segurança pendentes

last(/Patchmon by Flpems/patchmon.host.sec_updates[{#PM_HOSTID}])>=10

Updates totais (AVERAGE)

{#PM_HOSTNAME} possui mais de 20 updates pendentes

last(/Patchmon by Flpems/patchmon.host.pending_updates[{#PM_HOSTID}])>=20

Dependência:
Depende do trigger de updates de segurança.

Disponibilidade da API (NOVO na v2)

Disponibilidade da API do PatchMon: sem dados por 2h

nodata(/Patchmon by Flpems/patchmon.hosts.raw,2h)=1

Instalação

Zabbix → Data collection → Templates → Import

Importe o YAML do template

Configure as macros:

{$PM_URL}

{$PM_KEY}

{$PM_SECRET}

Vincule o template a um host

Aguarde:

patchmon.hosts.raw receber o JSON

A discovery criar os itens por host

Segurança

Não armazene {$PM_SECRET} em texto plano fora do Zabbix

Utilize Zabbix Proxy se o PatchMon não estiver diretamente acessível

Créditos

Template: Patchmon by Flpems
Versão: v2
