# Lead Scorer — Challenge 003

**Área:** Vendas / RevOps  
**Autor:** Abrahão Marques  
**Stack:** HTML + CSS + JavaScript (vanilla) — zero dependências de backend  
**Dataset:** [CRM Sales Predictive Analytics](https://www.kaggle.com/datasets/agungpambudi/crm-sales-predictive-analytics) (CC0)

---

## 🚀 Setup — Como rodar

### Opção 1: Direto no browser (mais simples)
```
Baixe o arquivo index.html e abra no browser.
Não precisa de servidor, instalação ou dependências.
```

### Opção 2: Via servidor local
```bash
# Python
python -m http.server 8080

# Node
npx serve .
```
Acesse: `http://localhost:8080`

### Opção 3: URL pública (Replit)
> Em breve

---

## 🧠 Lógica de Scoring

O score prioriza deals com maior probabilidade de fechar **agora**. Três critérios independentes somam até **100 pontos**:

### Critério 1 — Deal Stage (30pts)
| Stage | Pontos | Razão |
|---|---|---|
| Engaging | 30pts | Vendedor já está em contato ativo |
| Prospecting | 10pts | Fase inicial, menor maturidade |
| Won / Lost | 0pts | Deals fechados não são priorizados |

### Critério 2 — Urgência: dias no pipeline (35pts)
| Tempo | Pontos | Interpretação |
|---|---|---|
| < 150 dias | 35pts | Deal fresco, ainda aquecido |
| 150–250 dias | 25pts | Atenção necessária |
| 251–380 dias | 10pts | Esfriando |
| > 380 dias | 3pts | Provavelmente morto |
| Sem data | 0pts | Prospecting sem engage_date |

**Penalidade:** deals em Engaging há **mais de 165 dias** (mediana do dataset) recebem **-8pts** por estagnação no stage.

### Critério 3 — Valor Potencial do Produto (35pts)
`sales_price` normalizado de 0–35 dentro do portfolio de 7 produtos.  
Faixa: $55 (MG Special) → $26.768 (GTK 500)

### Por que excluímos o revenue da conta?
68% dos deals abertos não possuem `account` vinculada no pipeline — incluir esse critério penalizaria injustamente a maioria dos deals. O `sales_price` já captura o valor potencial com cobertura de 100%.

---

## 📊 Funcionalidades

### Aba Pipeline
- Tabela de 2.089 deals abertos ordenados por `$ Valor` (padrão)
- Filtros: Regional, Manager, Vendedor, Stage, Série, Status (Abertos/Won/Lost/Todos), Busca livre
- Score sutil com pill colorido + barra indicadora
- Badges: ⚠ Esfriando (>250 dias) | ⏸ Estagnado (Engaging >165 dias)
- Explicação do score expandível por clique
- Exportação CSV do filtro ativo

### Aba Vendedor
- Seleção por nome do vendedor
- 4 KPI cards clicáveis: Pipeline $, Score Alto, Esfriando, Estagnados
- Ao clicar num KPI: tabela de deals + gráficos filtram para aquele subconjunto
- Gráficos: $ por stage, $ por produto, distribuição de score (doughnut)
- Performance histórica: receita Won, win rate, receita por produto

### Aba Manager
- Seleção por manager
- Mesma lógica de KPIs interativos
- Ranking de vendedores por $ em pipeline
- Gráficos: $ por produto, $ por vendedor, $ em risco (esfriando + estagnado)
- Performance histórica da equipe: receita e win rate por vendedor
- Tabela detalhada com filtros

---

## 📋 Dados

| Dataset | Registros | Uso |
|---|---|---|
| `sales_pipeline.csv` | 8.800 | Tabela central — todos os deals |
| `accounts.csv` | 85 | Join para setor e receita da conta |
| `products.csv` | 7 | Join para preço e série do produto |
| `sales_teams.csv` | 35 | Join para manager e regional |

**Cruzamento:** `sales_pipeline` → join com `sales_teams` (por `sales_agent`) → join com `accounts` (por `account`) → join com `products` (por `product`).

**Correção aplicada:** o campo `product` no pipeline usa `GTXPro` (sem espaço) enquanto a tabela de produtos usa `GTX Pro` — normalizado antes do join para evitar 333 deals sem `sales_price`.

---

## ⚠️ Limitações

| Limitação | Impacto | O que precisaria para escalar |
|---|---|---|
| Dados embutidos no HTML | Arquivo pesado (~2.5MB); dados não são tempo real | Backend com API + banco de dados |
| Scoring por regras fixas | Não aprende com novos dados | Modelo de ML com retreinamento periódico |
| Win rate histórico não entra no score | Vendedor com 70% de WR tem mesmo peso que um com 55% | Adicionar WR do vendedor como multiplicador |
| Sem autenticação | Qualquer pessoa com o link vê todos os dados | Login por vendedor com dados filtrados por permissão |
| 68% dos deals sem account vinculada | Revenue da conta excluído do scoring | Enriquecimento de dados de CRM |
| Data de referência fixa (Dez/2017) | "Dias no pipeline" calculado a partir de data histórica | Integração com CRM em tempo real |

---

## 🤖 Process Log — Como a IA foi usada

Este projeto foi construído inteiramente com **Claude Sonnet** como par de programação.

### Processo
1. **Análise exploratória dos dados** — Claude leu os 4 CSVs, identificou o bug de `GTXPro` vs `GTX Pro`, mapeou os campos disponíveis para scoring e quantificou dados faltantes
2. **Definição da lógica de scoring** — feita em conversa iterativa: cada decisão (threshold de estagnação, exclusão de revenue, penalidade de -8pts) foi discutida com base nos dados reais antes de codificar
3. **Build do dashboard** — gerado em HTML/JS vanilla, sem framework, para máxima portabilidade
4. **Iterações de produto** — adição de impacto financeiro ($), KPIs clicáveis, aba Manager, histórico Won/Lost, tema Prumo, responsividade mobile — todas por prompts conversacionais
5. **QA de dados** — auditoria final cruzando CSV original vs dados embutidos no HTML (100% íntegros)

### Ferramentas de IA utilizadas
- **Claude Sonnet** (claude.ai) — análise de dados, arquitetura, geração de código, debugging, design

---

## 🎨 Design

**Paleta:**  
- Primária: `#B0AFFF` (lavanda)  
- Escura: `#0A0A14`  
- Superfície: `#F4F4FB`  
- Borda: `#E0E0EE`  

**Tipografia:** Plus Jakarta Sans + DM Mono  
**Charts:** Chart.js 4.4.1
