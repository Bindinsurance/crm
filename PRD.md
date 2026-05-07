# PRD — Bind Insurance CRM
**Product Requirements Document**
*Última atualização: 2026-05-07*

---

## 1. Visão do Produto

Um CRM web leve e seguro para a Bind Insurance gerenciar sua carteira de clientes de seguros de saúde e automóvel, acessível de qualquer dispositivo, sem instalação, com dados persistentes na nuvem.

**Usuário principal:** Edu (bindinsuranceus@gmail.com) e agentes da corretora.

---

## 2. Objetivos

1. Centralizar todos os clientes num único sistema acessível pela web
2. Importar e sincronizar dados da planilha Excel existente sem perder edições manuais
3. Monitorar comissões mensais e anuais por agente e por carteira
4. Ter segurança adequada (autenticação, headers HTTP, RLS)

---

## 3. Funcionalidades Entregues ✅

### 3.1 Gestão de Clientes
- [x] Cadastro completo: nome, sobrenome, email, telefone, idioma, endereço (cidade/estado/condado)
- [x] Serviços: saúde (health) e/ou automóvel (car)
- [x] Status: Application / Cancel
- [x] Campos financeiros: AP, AN2, valor, comissão mensal (mo), anual (yr), meses
- [x] Campos de seguro: seguradora (carrier), tipo de plano, data de vigência, agente
- [x] Campos de carro: policyNum, carCarrier, effDateCar, expireDateCar, numCars, deductible, valuePol
- [x] Observações livres (obs)
- [x] Controle de renovação e cancelamento

### 3.2 Busca e Filtros
- [x] Busca em tempo real por nome, email, cidade, telefone
- [x] Filtro por status (Application / Cancel)
- [x] Filtro por serviço (health / car)
- [x] Filtro por agente
- [x] Filtro por estado (state)

### 3.3 Importação Excel
- [x] Suporte a `.xlsx` / `.xls`
- [x] Detecção automática de planilha saúde vs. carro pelo nome da aba e cabeçalhos
- [x] Detecção automática de layout (24/25/26 colunas)
- [x] Merge inteligente: só insere registros novos, preserva dados existentes
- [x] Tratamento de campos em branco: substitui por `(Sem Nome)` / `(Sem Sobrenome)` em vez de ignorar silenciosamente
- [x] Relatório de resultado: X adicionados, Y já existiam, Z com nome em branco
- [x] Exclusão de linhas de subtotal (fn puramente numérico)

### 3.4 Dashboard / Relatórios
- [x] Totais de comissão mensal e anual
- [x] Breakdown por agente
- [x] Contagem de clientes ativos vs. cancelados

### 3.5 Infraestrutura
- [x] Hospedagem no Vercel (deploy automático via GitHub)
- [x] Backup no GitHub Pages
- [x] Headers HTTP de segurança (vercel.json)
- [x] Autenticação via Supabase Auth
- [x] RLS no Supabase
- [x] Paginação de 1000 registros por página no carregamento

### 3.6 Deploy Automático
- [x] Script `deploy.js` publica CRM + CLAUDE.md + README + PRD num único commit
- [x] `DEPLOY.bat` para execução com 1 clique no Windows

---

## 4. Requisitos Técnicos

| Requisito | Especificação |
|---|---|
| Arquivo único | Todo o app em `bind_insurance_FINAL.html` |
| Sem framework | JavaScript vanilla |
| Compatibilidade | Chrome, Edge, Firefox modernos |
| Banco de dados | Supabase PostgreSQL |
| Autenticação | Supabase Auth (email/password) |
| Hosting | Vercel (free tier) |
| Excel | SheetJS via CDN |

---

## 5. Backlog / Melhorias Futuras

### Prioridade Alta
- [ ] Busca por campos em branco — listar todos os `(Sem Nome)` para correção manual
- [ ] Edição em massa — selecionar múltiplos clientes e alterar agente/status
- [ ] Exportação para Excel da lista filtrada atual

### Prioridade Média
- [ ] Notificações de renovação — alertas para apólices próximas do vencimento
- [ ] Upload de documentos por cliente (integração com Storage do Supabase)
- [ ] Histórico de alterações por cliente (audit log)
- [ ] Multi-usuário com permissões por agente (cada agente vê só seus clientes)

### Prioridade Baixa
- [ ] Versão mobile (PWA)
- [ ] Relatório PDF mensal de comissões
- [ ] Integração com calendário para follow-ups
- [ ] Dark mode

---

## 6. Decisões de Design

| Decisão | Motivo |
|---|---|
| Arquivo HTML único | Simplicidade de deploy e manutenção |
| Chave anon Supabase no frontend | Por design do Supabase; segurança via RLS |
| Merge por fn+ln | Chave natural de identificação do cliente |
| Placeholder `(Sem Nome)` | Não perder dados; deixar visível para correção manual |
| Vercel em vez de só GitHub Pages | Headers HTTP reais (GitHub Pages não suporta) |

---

## 7. Contato

**Produto:** Bind Insurance CRM
**Responsável:** Edu — bindinsuranceus@gmail.com
**Repositório:** https://github.com/Bindinsurance/crm
