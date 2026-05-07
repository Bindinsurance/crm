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
