English:
# Patchmon by Flpems — Zabbix Template

Zabbix template (exported for **Zabbix 7.4**) to collect **PatchMon** data via **HTTP Agent** and automatically create per-host items using **LLD (Low-Level Discovery)**.

## Overview

The template queries PatchMon host endpoints and discovers hosts dynamically to collect:

- Docker integration counters/status
- System information (Kernel, SELinux, reboot pending)
- Package statistics (installed, pending total updates, pending security updates, repositories)
- OS/version (formatted via JavaScript preprocessing)

## Compatibility

- **Export format:** Zabbix **7.4**
- **Collection method:** HTTP Agent with **Basic** authentication
- **Default interval:** 1h (items and prototypes)

## Requirements

1. PatchMon instance reachable from **Zabbix Server** or **Zabbix Proxy**.
2. API credentials (key/user + secret).
3. Outbound HTTP/HTTPS allowed from Zabbix to PatchMon (firewall/ACL).
4. API endpoints used by this template:
   - `GET {$PM_URL}/api/v1/api/hosts`
   - `GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/integrations`
   - `GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/info`
   - `GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/system`
   - `GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/stats`

## Macros (required)

Set these macros on the template (or override at host/host group level):

- `{$PM_URL}`: PatchMon base URL (e.g. `https://patchmon.yourdomain.local`)
- `{$PM_KEY}`: API user/key (used as **Basic Auth username**)
- `{$PM_SECRET}`: API secret (used as **Basic Auth password**)

> Recommendations:
> - Mark `{$PM_SECRET}` as **Secret** in Zabbix.
> - Keep `{$PM_URL}` consistent (optionally without a trailing `/`).

## How it works

### Master item

- **PatchMon - Hosts list (raw)**
  - Key: `patchmon.hosts.raw`
  - Type: HTTP Agent (TEXT)
  - URL: `{$PM_URL}/api/v1/api/hosts`
  - Auth: Basic (`{$PM_KEY}` / `{$PM_SECRET}`)
  - Interval: `1h`

This item retrieves the full JSON payload containing the hosts list.

### Discovery (LLD)

- **PatchMon - Host discovery**
  - Key: `patchmon.host.discovery`
  - Type: **Dependent**
  - Master item: `patchmon.hosts.raw`
  - Preprocessing: JSONPath `$.hosts[*]`

**Extracted LLD macros:**
- `{#PM_HOSTID}` → `$.id`
- `{#PM_HOSTNAME}` → `$.hostname`
- `{#PM_IP}` → `$.ip`
- `{#PM_FRIENDLY}` → `$.friendly_name`
- `{#PM_GROUP}` → `$.host_groups[0].name` *(note: uses only the first array entry)*

## Per-host items (prototypes)

All prototypes run with default `1h` interval and `30s` timeout, using Basic Auth with `{$PM_KEY}` / `{$PM_SECRET}`.

### Docker (endpoint `/integrations`)

- **Docker Container Count**
  - Key: `patchmon.host.dckcnt[{#PM_HOSTID}]`
  - JSONPath: `$.integrations.docker.containers_count`

- **Docker Integration Status**
  - Key: `patchmon.host.dckint[{#PM_HOSTID}]`
  - Type: TEXT
  - JSONPath: `$.integrations.docker.enabled`

- **Docker Networks Count**
  - Key: `patchmon.host.dckntwcnt[{#PM_HOSTID}]`
  - JSONPath: `$.integrations.docker.networks_count`

- **Docker Volumes Count**
  - Key: `patchmon.host.dckvlcnt[{#PM_HOSTID}]`
  - JSONPath: `$.integrations.docker.volumes_count`

### System info (endpoints `/info` and `/system`)

- **System OS/Version**
  - Key: `patchmon.host.info[{#PM_HOSTID}]`
  - Type: TEXT
  - Preprocessing: JavaScript (concatenates `os_type` + `os_version`)

- **Kernel Version**
  - Key: `patchmon.host.krnversion[{#PM_HOSTID}]`
  - Type: TEXT
  - JSONPath: `$.kernel_version`

- **SELinux Status**
  - Key: `patchmon.host.selstatus[{#PM_HOSTID}]`
  - Type: TEXT
  - JSONPath: `$.selinux_status`

- **Reboot Status**
  - Key: `patchmon.host.rbtstatus[{#PM_HOSTID}]`
  - Type: TEXT
  - JSONPath: `$.needs_reboot`
  - STR_REPLACE:
    - `false` → `No reboot pending`
    - `true` → `Reboot pending`

### Package stats (endpoint `/stats`)

- **Installed Packages**
  - Key: `patchmon.host.stats[{#PM_HOSTID}]`
  - JSONPath: `$.total_installed_packages`

- **Pending Updates: Total**
  - Key: `patchmon.host.pending_updates[{#PM_HOSTID}]`
  - JSONPath: `$.outdated_packages`

- **Pending Updates: Security**
  - Key: `patchmon.host.sec_updates[{#PM_HOSTID}]`
  - JSONPath: `$.security_updates`

- **Total Repositories**
  - Key: `patchmon.host.repos[{#PM_HOSTID}]`
  - JSONPath: `$.total_repos`

> Note: several items use `DISCARD_UNCHANGED` to reduce storage when values do not change.

## Triggers (prototypes)

- **Security updates (DISASTER)**
  - Name: `{#PM_HOSTNAME} has more than 10 pending security updates`
  - Expression:
    - `last(/Patchmon by Flpems/patchmon.host.sec_updates[{#PM_HOSTID}])>=10`

- **Total updates (HIGH)**
  - Name: `{#PM_HOSTNAME} has more than 20 pending updates`
  - Expression:
    - `last(/Patchmon by Flpems/patchmon.host.pending_updates[{#PM_HOSTID}])>=20`
  - Dependency:
    - Depends on the security-updates trigger (prevents duplicate alerting)

## Installation

1. In Zabbix: **Data collection → Templates → Import**
2. Import the template YAML.
3. Open the imported template and set the macros:
   - `{$PM_URL}`, `{$PM_KEY}`, `{$PM_SECRET}`
4. Link the template to a host (you can use a “dummy” host just to run the checks).
5. Wait for the first run of `patchmon.hosts.raw` and verify:
   - **Latest data** shows JSON arriving
   - **Discovery** created host prototypes and items

## Troubleshooting

- **No data on the master item**
  - Validate `{$PM_URL}` and connectivity (DNS/routing/firewall)
  - Confirm credentials (`{$PM_KEY}` / `{$PM_SECRET}`)
  - Test `/api/v1/api/hosts` outside Zabbix

- **Discovery not creating items**
  - Check that the JSON includes `hosts` and that `$.hosts[*]` matches the payload
  - If `host_groups` is empty, `{#PM_GROUP}` may be blank (discovery still works, but tags/labels may be impacted)

- **Preprocessing errors**
  - Confirm expected fields exist (e.g. `kernel_version`, `needs_reboot`, `security_updates`, etc.)
  - Adjust JSONPath if your PatchMon schema differs

## Security

- Do not store `{$PM_SECRET}` in plain text outside Zabbix.
- Use a Zabbix Proxy if PatchMon is not directly reachable from the server.

## Credits

Template: **Patchmon by Flpems**

###########################################################################################################

Português Brasil:
# Patchmon by Flpems — Zabbix Template

Template Zabbix (export **Zabbix 7.4**) para coletar informações do **PatchMon** via **HTTP Agent** e criar itens automaticamente por host usando **LLD (Low-Level Discovery)**.

## Visão geral

O template consulta o endpoint de hosts do PatchMon e, a partir da lista retornada, descobre dinamicamente cada host para coletar:

- Contadores e status de integração Docker
- Informações do sistema (Kernel, SELinux, reboot pendente)
- Estatísticas de pacotes (instalados, atualizações pendentes totais e de segurança, repositórios)
- OS/versão (formatado via JavaScript no preprocessing)

## Compatibilidade

- **Formato do export:** Zabbix **7.4**
- **Tipo de coleta:** HTTP Agent (com autenticação **Basic**)
- **Frequência padrão:** 1h (itens e protótipos)

## Pré-requisitos

1. Instância do **PatchMon** acessível a partir do **Zabbix Server** ou **Zabbix Proxy**.
2. Credenciais para API (usuário/chave e secret).
3. Permissão de saída HTTP/HTTPS do Zabbix para o PatchMon (firewall/ACL).
4. Endpoints da API disponíveis (conforme configurado no template):
   - `GET {$PM_URL}/api/v1/api/hosts`
   - `GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/integrations`
   - `GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/info`
   - `GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/system`
   - `GET {$PM_URL}/api/v1/api/hosts/{#PM_HOSTID}/stats`

## Macros (obrigatórias)

Configure estas macros no template (ou em nível de host/host group, se preferir herança):

- `{$PM_URL}`: URL base do PatchMon (ex.: `https://patchmon.seudominio.local`)
- `{$PM_KEY}`: API user / key (usado como **Basic Auth username**)
- `{$PM_SECRET}`: API secret (usado como **Basic Auth password**)

> Recomendações:
> - Marque `{$PM_SECRET}` como **Secret** no Zabbix.
> - Garanta que `{$PM_URL}` não termine com `/` (opcional, mas ajuda a padronizar).

## Como o template funciona

### Item mestre

- **PatchMon - Hosts list (raw)**
  - Key: `patchmon.hosts.raw`
  - Tipo: HTTP Agent (TEXT)
  - URL: `{$PM_URL}/api/v1/api/hosts`
  - Auth: Basic (`{$PM_KEY}` / `{$PM_SECRET}`)
  - Intervalo: `1h`

Este item retorna o JSON completo com a lista de hosts.

### Discovery (LLD)

- **PatchMon - Host discovery**
  - Key: `patchmon.host.discovery`
  - Tipo: **Dependent**
  - Master item: `patchmon.hosts.raw`
  - Preprocessing: JSONPath `$.hosts[*]`

**Macros LLD extraídas:**
- `{#PM_HOSTID}`  → `$.id`
- `{#PM_HOSTNAME}` → `$.hostname`
- `{#PM_IP}` → `$.ip`
- `{#PM_FRIENDLY}` → `$.friendly_name`
- `{#PM_GROUP}` → `$.host_groups[0].name` *(observação: usa apenas o primeiro grupo do array)*

## Itens criados por host (protótipos)

Todos com intervalo padrão `1h` e `timeout 30s`, usando Basic Auth com `{$PM_KEY}` / `{$PM_SECRET}`.

### Docker (endpoint `/integrations`)

- **Docker Container Count**
  - Key: `patchmon.host.dckcnt[{#PM_HOSTID}]`
  - JSONPath: `$.integrations.docker.containers_count`

- **Docker Integration Status**
  - Key: `patchmon.host.dckint[{#PM_HOSTID}]`
  - Tipo: TEXT
  - JSONPath: `$.integrations.docker.enabled`

- **Docker Networks Count**
  - Key: `patchmon.host.dckntwcnt[{#PM_HOSTID}]`
  - JSONPath: `$.integrations.docker.networks_count`

- **Docker Volumes Count**
  - Key: `patchmon.host.dckvlcnt[{#PM_HOSTID}]`
  - JSONPath: `$.integrations.docker.volumes_count`

### Info do sistema (endpoints `/info` e `/system`)

- **System OS/Version**
  - Key: `patchmon.host.info[{#PM_HOSTID}]`
  - Tipo: TEXT
  - Preprocessing: JavaScript (concatena `os_type` + `os_version`)

- **Kernel Version**
  - Key: `patchmon.host.krnversion[{#PM_HOSTID}]`
  - Tipo: TEXT
  - JSONPath: `$.kernel_version`

- **SELinux Status**
  - Key: `patchmon.host.selstatus[{#PM_HOSTID}]`
  - Tipo: TEXT
  - JSONPath: `$.selinux_status`

- **Reboot Status**
  - Key: `patchmon.host.rbtstatus[{#PM_HOSTID}]`
  - Tipo: TEXT
  - JSONPath: `$.needs_reboot`
  - STR_REPLACE:
    - `false` → `No Reboot pending`
    - `true`  → `Reboot pending`

### Estatísticas de pacotes (endpoint `/stats`)

- **Installed Packages**
  - Key: `patchmon.host.stats[{#PM_HOSTID}]`
  - JSONPath: `$.total_installed_packages`

- **Pending Updates: Total**
  - Key: `patchmon.host.pending_updates[{#PM_HOSTID}]`
  - JSONPath: `$.outdated_packages`

- **Pending Updates: Security**
  - Key: `patchmon.host.sec_updates[{#PM_HOSTID}]`
  - JSONPath: `$.security_updates`

- **Total Repos**
  - Key: `patchmon.host.repos[{#PM_HOSTID}]`
  - JSONPath: `$.total_repos`

> Observação: vários itens aplicam `DISCARD_UNCHANGED`, reduzindo armazenamento quando não há mudança.

## Triggers (protótipos)

- **Security updates (DISASTER)**
  - Nome: `{#PM_HOSTNAME} has more than 10 pending security updates`
  - Expressão:
    - `last(/Patchmon by Flpems/patchmon.host.sec_updates[{#PM_HOSTID}])>=10`

- **Total updates (HIGH)**
  - Nome: `{#PM_HOSTNAME} has more than 20 pending updates`
  - Expressão:
    - `last(/Patchmon by Flpems/patchmon.host.pending_updates[{#PM_HOSTID}])>=20`
  - Dependência:
    - Depende do trigger de security updates (evita alertas duplicados)

## Instalação

1. No Zabbix: **Data collection → Templates → Import**
2. Importe o YAML do template.
3. Abra o template importado e configure as macros:
   - `{$PM_URL}`, `{$PM_KEY}`, `{$PM_SECRET}`
4. Associe o template a um host (pode ser um host “dummy” apenas para rodar a coleta).
5. Aguarde a primeira coleta do item `patchmon.hosts.raw` e verifique:
   - Em **Latest data** se o JSON está chegando
   - Em **Discovery** se os hosts foram descobertos e os itens criados

## Troubleshooting

- **Sem dados no item mestre**:
  - Valide `{$PM_URL}` e conectividade (DNS/rota/firewall)
  - Confirme credenciais (`{$PM_KEY}` / `{$PM_SECRET}`)
  - Teste o endpoint `/api/v1/api/hosts` fora do Zabbix

- **Discovery não cria itens**:
  - Verifique se o JSON contém `hosts` e se o JSONPath `$.hosts[*]` faz sentido para o payload retornado
  - Se `host_groups` vier vazio, `{#PM_GROUP}` pode ficar em branco (não impede discovery, mas afeta tags/uso)

- **Erros de preprocessing**:
  - Confirme os campos esperados no JSON (ex.: `kernel_version`, `needs_reboot`, `security_updates`, etc.)
  - Ajuste JSONPath caso sua versão do PatchMon mude o schema

## Segurança

- Não armazene `{$PM_SECRET}` em texto plano fora do Zabbix.
- Use proxy se o PatchMon não for acessível diretamente pelo server.

## Créditos

Template: **Patchmon by Flpems**
