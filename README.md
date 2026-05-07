# Bind Insurance CRM

Sistema de gestão de clientes (CRM) para a corretora **Bind Insurance**, desenvolvido como aplicação web de arquivo único com backend no Supabase.

## 🌐 Acesso

| Ambiente | URL |
|---|---|
| **Produção (Vercel)** | https://crm-seven-neon-65.vercel.app |
| **Backup (GitHub Pages)** | https://bindinsurance.github.io/crm |

## ✨ Funcionalidades

- **Gestão de clientes** — cadastro completo com nome, email, telefone, endereço, agente, seguradora, plano
- **Busca em tempo real** — por nome, email, cidade, telefone
- **Filtros avançados** — por status, tipo de serviço, agente, estado
- **Importação Excel** — importa planilhas `.xlsx` de saúde e automóvel com **merge inteligente** (não apaga dados existentes)
- **Dashboard financeiro** — comissões mensais/anuais por agente e carteira
- **Autenticação** — login seguro via Supabase Auth
- **Segurança** — headers HTTP completos (HSTS, CSP, X-Frame-Options, etc.)

## 🛠 Tecnologia

| Camada | Tecnologia |
|---|---|
| Frontend | HTML + CSS + JavaScript (arquivo único) |
| Backend/DB | Supabase (PostgreSQL + PostgREST) |
| Hospedagem | Vercel (deploy automático via GitHub) |
| Excel | SheetJS (XLSX) |
| Segurança | vercel.json com headers HTTP |

## 📁 Estrutura

```
bind_insurance_FINAL.html   → aplicação completa
vercel.json                 → headers de segurança
supabase_schema.sql         → schema do banco de dados
CLAUDE.md                   → contexto para AI
PRD.md                      → requisitos do produto
deploy.js                   → publicação automática
DEPLOY.bat                  → deploy com 1 clique (Windows)
```

## 🚀 Deploy

Para publicar alterações no Vercel:

```bash
node deploy.js "descrição das mudanças"
```

Ou no Windows, clique duas vezes em **`DEPLOY.bat`**.

O Vercel detecta o push e faz redeploy automático em ~30 segundos.

## 📊 Banco de Dados

- **Tabela:** `clients`
- **Campos principais:** `fn`, `ln`, `email`, `phone`, `services`, `status`, `carrier`, `mo` (mensal), `yr` (anual), `agent`, `city`, `state`

## 🔐 Segurança

- Autenticação obrigatória via Supabase Auth
- Row Level Security (RLS) ativo no Supabase
- Headers HTTP de segurança via `vercel.json`
- Chave anon do Supabase é pública por design (acesso controlado por RLS)

---

*Última atualização: 2026-05-07*
