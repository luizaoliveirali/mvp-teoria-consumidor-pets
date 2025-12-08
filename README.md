# Pets na Cesta de Consumo 🐶🐱  
**MVP de Engenharia de Dados – Teoria do Consumidor aplicada a gastos com animais de estimação**

Este repositório reúne o MVP desenvolvido para a disciplina de **Engenharia de Dados**, aplicando conceitos de **teoria do consumidor** ao tema dos **gastos com animais de estimação (pets)**.

O projeto constrói um pequeno **Data Warehouse em nuvem (Databricks)**, a partir de dados oficiais do **Office for National Statistics (ONS)**, para analisar como os gastos com pets são incorporados à cesta de consumo das famílias ao longo dos **decil de renda**.

---

## 🎯 Objetivo

O objetivo do trabalho é analisar, à luz da teoria do consumidor, como os **gastos com animais de estimação (pets)** são incorporados à cesta de consumo das famílias e em que medida eles são priorizados ou ajustados diante de diferentes níveis de renda.

Mais especificamente, o MVP busca responder:

- Qual é a **participação média** dos gastos com pets no orçamento das famílias?
- Como essa participação se compara a categorias clássicas como **alimentação**?
- À medida que a renda aumenta (por decil), a participação de pets:
  - cresce,
  - cai,
  - ou se mantém relativamente estável?
- O comportamento dos gastos com pets se aproxima mais de:
  - um **bem de necessidade**,
  - um **bem de luxo**,
  - ou uma categoria **discricionária “protegida”** no orçamento?

As respostas são construídas combinando:

- **modelagem de dados** (esquema estrela),  
- **pipeline de ETL em SQL no Databricks**,  
- e **análise econômica** com base em curvas de Engel e composição da cesta de consumo.

---

## 📊 Dados utilizados

Os dados utilizados são oficiais e públicos:

- **Fonte:** Office for National Statistics (ONS) – Reino Unido  
- **Pesquisa:** *Family spending in the UK* (Living Costs and Food Survey – LCF)  
- **Tabela principal:** A6 – *Detailed household expenditure by gross income decile group*  
- **Ano de referência:** **FYE 2024** (Financial Year Ending 2024)

O dataset traz o gasto **médio semanal por domicílio**, por **categoria de despesa** (COICOP) e por **decil de renda bruta**. Dentro dessas categorias existem linhas específicas para gastos com **pets**, que são a base das análises.

> 🔎 Detalhes completos sobre:
> - origem dos dados,  
> - link oficial,  
> - estrutura das variáveis  
> - e licença de uso  
> estão documentados em [`data/README.md`](data/README.md).

---

## 🏗️ Arquitetura do Pipeline

A solução foi implementada no **Databricks Community Edition**, em camadas lógicas inspiradas em um Data Warehouse:

### 1. Bronze – Dados brutos

- Upload da tabela A6 do workbook do ONS (arquivo `.xlsx`).  
- Criação da tabela inicial `family_spending_a6` no Databricks, refletindo as colunas originais (formato “wide”: `d1_lowest` … `d10_highest`).

### 2. Silver – Dados tratados / estruturados

- Criação de uma tabela limpa (`family_spending_a6_clean`), com:
  - seleção das colunas relevantes (`code`, `description`, gastos por decil e total);
  - padronização de nomes.
- Transformação do formato **“wide” → “long”** na view `vw_despesa_long`:
  - cada linha passa a representar **[categoria × decil de renda]**.
- Cálculo do gasto médio total por decil e da **participação da categoria no orçamento** (`participacao_orcamento`) na view `vw_despesa_com_total`.

### 3. Gold – Modelo analítico (esquema estrela)

- Dimensões:
  - `dim_renda` – decis de renda (1 a 10) e descrições;
  - `dim_categoria_consumo` – categorias de despesa, com flag para identificar gastos com pets;
  - `dim_tempo` – período de referência (FYE 2024);
  - `dim_geografia` – agregado “UK total”.
- Tabela fato:
  - `fato_despesa_familiar` – gasto médio semanal por categoria × decil, gasto total do decil e participação da categoria no orçamento.

Documentação detalhada:

- Modelo de dados: `docs/modelo_dados.md`  
- Catálogo de dados (dicionário de campos): `docs/catalogo_dados.md`

---

## 🔎 Análise dos Resultados

A análise econômica baseia-se na tabela fato e em views adicionais (como `vw_fato_join` e `vw_fato_grupos`), permitindo:

- calcular a **participação de pets no orçamento** por decil de renda;  
- comparar pets com a categoria **Alimentação** (Lei de Engel);  
- interpretar o comportamento dos gastos com pets à luz da teoria do consumidor.

Principais achados (resumidos):

- **Alimentação** apresenta comportamento compatível com a **Lei de Engel**:
  - maior participação nos decis de renda mais baixos;
  - queda da participação relativa à medida que a renda aumenta.
- Os **gastos com pets**:
  - mantêm participação **relativamente estável**, em torno de **1% a 1,5%** do orçamento ao longo dos decis;
  - apresentam leve aumento nos decis intermediários, mas sem crescimento contínuo nos decis mais ricos.
- Interpretação:
  - alimentação se comporta como **bem de necessidade** (share cai com a renda);
  - pets formam uma categoria **discricionária, porém “protegida”**, cujo peso percentual não é comprimido de forma significativa mesmo entre famílias de menor renda.

A discussão completa está em:  
`docs/analise_resultados.md`

---

## 🧱 Estrutura do Repositório

```text
mvp-teoria-consumidor-pets/
├─ notebooks/
│  ├─ 01_busca_coleta.ipynb              # leitura + limpeza + transformação wide→long
│  ├─ 02_modelagem_carga.ipynb           # criação das dimensões e tabela fato (Gold)
│  └─ 03_analise_consumidor_pets.ipynb   # consultas e análises (teoria do consumidor)
├─ docs/
│  ├─ objetivo.md                        # detalhamento do problema e perguntas
│  ├─ modelo_dados.md                    # descrição do esquema estrela
│  ├─ catalogo_dados.md                  # dicionário de dados
│  ├─ analise_resultados.md              # interpretação dos resultados
│  └─ autoavaliacao.md                   # autoavaliação do MVP
├─ data/
│  ├─ README.md                          # detalhes da fonte de dados e licença (ONS)
│  └─ (arquivos .xlsx/.csv)
├─ img/
│  ├─ exemplo_tabela_gold.png            # exemplo da tabela fato Gold no Databricks
│  └─ grafico_curva_engel_pets.png       # curva de Engel aproximada para gastos com pets
└─ README.md                             # este arquivo
