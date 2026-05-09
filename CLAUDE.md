# CLAUDE.md — Bind Insurance CRM

> Este arquivo é atualizado automaticamente a cada deploy. Serve como contexto permanente para o assistente AI (Claude) sobre o estado do projeto.

## Visão Geral

CRM web completo para corretora de seguros (Bind Insurance), construído como **arquivo HTML único** hospedado no **Vercel**, com banco de dados no **Supabase**.

- **URL de produção:** https://crm-seven-neon-65.vercel.app
- **Backup (GitHub Pages):** https://bindinsurance.github.io/crm
- **Repositório:** https://github.com/Bindinsurance/crm
- **Arquivo principal:** `bind_insurance_FINAL.html`

---

## Arquitetura

```
bind_insurance_FINAL.html   ← app completo (HTML + CSS + JS inline)
vercel.json                 ← headers de segurança HTTP
supabase_schema.sql         ← schema do banco de dados
CLAUDE.md                   ← contexto AI (este arquivo)
README.md                   ← documentação pública
PRD.md                      ← requisitos do produto
deploy.js                   ← script de publicação automática
DEPLOY.bat                  ← launcher Windows para deploy
```

### Backend: Supabase
- **Projeto:** lhesgpbzwnchmumhcmpm
- **Tabela principal:** `clients` (NÃO `clientes`)
- **Colunas-chave:** `fn` (first name), `ln` (last name), `email`, `phone`, `services`, `status`, `carrier`, `mo`, `yr`, `agent`
- **Autenticação:** Supabase Auth com redirect para URL de produção
- **Paginação:** 1000 registros por página em loop (`loadClients()`)

### Frontend
- **JavaScript:** vanilla, sem framework
- **Biblioteca Excel:** SheetJS (XLSX) via CDN
- **Estilo:** CSS inline, paleta azul/branco
- **Chaves:** `SUPABASE_URL` e `SUPABASE_ANON_KEY` estão no HTML (chave pública — correto por design do Supabase; segurança via RLS)

---

## Funções Principais

| Função | Descrição |
|---|---|
| `loadClients()` | Carrega todos os clientes do Supabase (paginado) |
| `saveClient(c)` | Salva/atualiza um cliente (upsert por UUID) |
| `getFiltered()` | Filtra clientes locais por busca + filtros |
| `toDbRow(c)` | Converte objeto JS → formato de linha do Supabase |
| `render()` | Re-renderiza a tabela/lista de clientes |
| `ss(patch)` | Atualiza estado global (ST) e re-renderiza |

---

## Sistema de Importação Excel

O botão "Importar" aceita `.xlsx` / `.xls`. O sistema suporta duas estruturas de planilha:

**Planilha de Saúde (Health)**
- Linha 0: cabeçalhos individuais de colunas
- Linha 1: célula mesclada B2:W2
- Linhas 2+: dados dos clientes
- Detecta automaticamente se tem coluna `phone` (26 colunas) ou não (25 ou 24 colunas)

**Planilha de Carro (Car/Auto)**
- Identificada pelo nome da aba ou por colunas `policy number` / `number of cars`
- Mapeamento diferente de colunas

### Lógica de Merge Inteligente (v2 — atual)
1. Carrega todas as chaves existentes do Supabase (`fn|ln`)
2. Filtra apenas registros novos (não duplicados)
3. Insere apenas os novos via `insert()` em lotes de 100
4. Recarrega a lista completa após inserção
5. Exibe relatório: X adicionados, Y já existiam

### Tratamento de Campos em Branco
- Linhas completamente vazias são ignoradas
- Linhas com subtotais numéricos (`fn` = número puro) são ignoradas
- Registros com `fn` ou `ln` em branco recebem placeholder `(Sem Nome)` / `(Sem Sobrenome)`
- O contador `blankNames` acumula e exibe aviso ao usuário no final

---

## Headers de Segurança HTTP (vercel.json)

```
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' cdnjs/jsdelivr; ...
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
```

---

## Fluxo de Deploy

Todo deploy é feito pelo script `deploy.js` que:
1. Lê os arquivos locais
2. Obtém o commit atual do branch `main`
3. Cria uma nova árvore Git com todos os arquivos modificados
4. Cria um commit com mensagem descritiva
5. Atualiza o ref do branch

O Vercel detecta o push e faz redeploy automático em ~30 segundos.

**Para publicar:** Execute `DEPLOY.bat` ou `node deploy.js "mensagem do commit"`

---

## Histórico de Versões

| Data | Mudança |
|---|---|
| 2026-05-07 | Import v2: insert registro a registro com fallback — lotes de 100 falham graciosamente; nomes de registros com erro são exibidos ao usuário |
| 2026-05-07 | Merge inteligente no import; fix campos em branco (Sem Nome); deploy automático; CLAUDE.md + README + PRD |
| Sessão anterior | Deploy no Vercel; vercel.json com headers de segurança; fix paginação de clientes |
| Sessão anterior | Busca corrigida; mapeamento de colunas Excel; suporte planilha carro |

---

## Instruções para o AI (Claude)

- O arquivo principal **sempre** é `bind_insurance_FINAL.html` — nunca editar as versões numeradas (v1–v6), elas são backups históricos.
- Após **qualquer** mudança no código ou documentação, rodar `deploy.js` para publicar tudo de uma vez.
- A chave do Supabase no frontend **é pública por design** — não remover nem comentar.
- O banco usa a tabela `clients` (inglês), não `clientes`.
- O campo `fn` = primeiro nome, `ln` = sobrenome.
- Sempre manter este `CLAUDE.md` atualizado com cada mudança relevante.

---

## Histórico de Problemas e Soluções

### [2026-05-09] CAUSA RAIZ: Vercel servia código antigo apesar dos deploys

**Problema:** Após cada deploy, o app no Vercel continuava mostrando dados antigos (1973 clientes / $105k) e não encontrava clientes como "Justin Mullins" que existiam na planilha.

**Causa raiz identificada:** O repositório GitHub continha DOIS arquivos HTML:
- `bind_insurance_FINAL.html` — código novo (atualizado em cada deploy)
- `index.html` — código ANTIGO (nunca atualizado)

O Vercel serve o `index.html` na URL raiz (`/`). Como o `deploy.js` nunca incluía `index.html` na lista de arquivos, todos os deploys anteriores atualizavam apenas `bind_insurance_FINAL.html`, enquanto o `index.html` antigo continuava sendo servido aos usuários.

**Solução aplicada (09/05/2026):**
1. Adicionado ao `deploy.js` na lista FILES: `{ local: 'bind_insurance_FINAL.html', remote: 'index.html' }`
2. Executado deploy — commit `04386616` — 6 arquivos publicados
3. Verificado: Vercel agora serve `BUILD: 2026-05-07-merge-v3` na URL raiz ✅

**Outras correções aplicadas na mesma sessão:**
- `toDbRow()`: campos INTEGER (`ap`, `an2`, `months`, `num_cars`) usam `Math.round()` para evitar rejeição silenciosa de batch no Supabase
- Importação: registros sem nome recebem `(Sem Nome)` / `(Sem Sobrenome)` em vez de serem descartados
- Importação: merge inteligente (não destrói dados existentes) + fallback registro-a-registro quando batch falha
- `deploy.js` atualizado para sempre publicar `bind_insurance_FINAL.html` como `index.html` automaticamente

**Pasta do projeto movida:** De `Downloads\App_Clientes_Bind` para `OneDrive\Personal Vault\App_Clientes_Bind`


---

## Sessão 09/05/2026 — Correção: Valor Carro Anual duplicado no dashboard

**Problema:** Dashboard exibia "Carro Anual" = $19,781.58 mas planilha mostrava $9,890.79 (exatamente metade).

**Causa raiz:** A linha de TOTAIS da planilha Excel foi importada como se fosse um cliente real.
- Registro fantasma: fn="(Sem Nome)", ln="(Sem Sobrenome)", services=["car"], mo=9890.79
- O CRM somava os 50 clientes reais ($9,890.79) + essa linha fantasma ($9,890.79) = $19,781.58

**Solução aplicada (09/05/2026):**
1. Deletado o registro fantasma do Supabase: id=`e1d44a63-3aec-421a-8870-f5e9857e24e9`
2. Adicionado filtro no código de importação para ignorar linhas sem nome:
   - Em `newClients.filter(c=>{` adicionado: `if(!(c.fn||'').trim()&&!(c.ln||'').trim())return false;`
3. Resultado verificado: 50 clientes reais, soma = $9,890.79 ✅
4. Arquivo salvo e deploy executado via DEPLOY_NOW.bat

**Lição:** A planilha Excel contém uma linha de totais (SUM) no final. O importador agora ignora linhas onde firstName E lastName estão ambos vazios/nulos.

---

## [09/05/2026] Fix: Case-insensitive filters (visa, status, language, carrier, source, agent, type)

**Problem**: Client-side filters returning 0 results even though data exists in Supabase.
**Root cause**: DB values are stored lowercase (normalized in prior session), but the JavaScript filter comparison was case-sensitive. E.g. "green card" (DB) ≠ "Green Card" (dropdown).
**Architecture note**: Filtering is entirely client-side. `loadClients()` loads ALL records; `getFiltered()` filters them in memory using `ST.filter` state. There is no `.eq()` server-side filter for these fields.

**Fixes applied to `bind_insurance_FINAL.html`**:
1. `loadClients()`: `visa: r.visa||''` → `visa: (r.visa||'').toLowerCase()` — normalize on load
2. `toDbRow()`: `visa: c.visa||null` → `visa: (c.visa||'').toLowerCase()||null` — normalize on save
3. `getFiltered()`: `c.visa!==f.visa` → `(c.visa||"").toLowerCase()!==(f.visa||"").toLowerCase()` — case-insensitive comparison
4. `toDbRow()`: Also normalized status, type, language, carrier, source, agent fields to lowercase on save

**Total changes**: 9 replacements, file grew from 94273 → 94469 characters.
**Script used**: `fix_filters.js` (in Claude outputs folder) + `RUN_FIX.bat`
**Deployed via**: DEPLOY_NOW.bat → GitHub Bindinsurance/crm main → Vercel auto-deploy

---

## [09/05/2026] Fix: VISA_OPTS sincronizado com planilha Base para o APP (commit ac0723c0 — 17:09)

**Problem**: Opções de visto no filtro do app estavam incompletas — E2, L1 e Application existiam na planilha mas não apareciam como opção de filtro no CRM.
**Root cause**: `VISA_OPTS` definido manualmente no código sem ser conferido contra os dados reais da planilha.

**Diagnóstico**: Script `read_visa_opts.js` leu o xlsx via node e extraiu 14 valores únicos da coluna visa (sheet "2026 Health"). Comparados com o array `VISA_OPTS` no HTML.

**Planilha contém** (valores únicos case-insensitive): `application, citizen, E2, green card, I-94, I797, L1, O, R1, student, work permit`

**Antes**: `['Student','Green Card','Work Permit','I797','I-94','Citizen','797C','O','R1']`
**Depois**: `['Application','Citizen','E2','Green Card','I-94','I797','L1','O','R1','Student','Work Permit','797C']`

**Adicionados**: E2, L1, Application. **Mantido**: 797C (para não quebrar registros existentes).
**Arquivo alterado**: `bind_insurance_FINAL.html` linha 171 — `const VISA_OPTS`
**Deployed via**: DEPLOY_VISA_FIX.bat → GitHub Bindinsurance/crm main → Vercel auto-deploy
