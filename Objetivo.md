# Pets na Cesta de Consumo 🐶🐱  
**Análise de gastos com animais de estimação à luz da Teoria do Consumidor**

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

## 📊 Dados

Os dados utilizados são oficiais e públicos:

- **Fonte:** Office for National Statistics (ONS) – Reino Unido  
- **Pesquisa:** *Family spending in the UK* (Living Costs and Food Survey – LCF)  
- **Dataset:** *Family spending workbook 1: detailed expenditure and trends*  
- **Tabela utilizada:** A6 – *Detailed household expenditure by gross income decile group*  
- **Ano de referência:** **FYE 2024** (Financial Year Ending 2024)

Principais características:

- Gastos **médios semanais** por domicílio, em **libras (£)**;  
- Desagregação por **decil de renda bruta** (*gross income decile group*);  
- Categorias de despesa alinhadas à **COICOP** (ex.: alimentação, transporte, habitação, “Pets and pet food” etc.);  
- Linhas específicas de despesa com **pets**, que são a base da análise.

Mais detalhes sobre a origem, estrutura e licença dos dados estão descritos em:  
`data/README.md`

---

## 🏗️ Arquitetura do Pipeline

A solução foi implementada no **Databricks** (Community Edition) em camadas lógicas, aproximando-se de um ambiente de **Data Warehouse**:

### 1. Bronze – Dados brutos

- Leitura da tabela A6 do workbook do ONS (arquivo `.xlsx`).  
- Criação da tabela inicial `family_spending_a6` no Databricks, refletindo as colunas originais (decis em formato “wide”: `d1_lowest` … `d10_highest`).

### 2. Silver – Dados tratados / estruturados

- Criação da tabela `family_spending_a6_clean`, com:
  - padronização de nomes de colunas;
  - seleção das colunas relevantes (`code`, `description`, gastos por decil e total).
- Transformação do formato “wide” para “long” por meio da view `vw_despesa_long`:
  - cada linha passa a representar **[categoria × decil de renda]**.
- Cálculo do gasto médio total por decil e do **share da categoria no orçamento** (`participacao_orcamento`) na view `vw_despesa_com_total`.

### 3. Gold – Modelo analítico (esquema estrela)

- Tabelas de dimensão:
  - `dim_renda` – decis de renda (1 a 10) e descrições;
  - `dim_categoria_consumo` – categorias de despesa, com `flag_pet` para identificar gastos com pets;
  - `dim_tempo` – período de referência (FYE 2024);
  - `dim_geografia` – agregado “UK total”.
- Tabela fato:
  - `fato_despesa_familiar` – gasto médio semanal por categoria × decil, total do decil e participação da categoria no orçamento.

Esse modelo é utilizado para:

- calcular a **participação de pets no orçamento** por decil de renda;
- comparar **pets x alimentação** e discutir **Lei de Engel**;
- analisar o comportamento de gastos com pets em diferentes faixas de renda.

Documentação detalhada:

- Modelo de dados: `docs/modelo_dados.md`  
- Catálogo de dados (dicionário de campos): `docs/catalogo_dados.md`

---

## 🔎 Análise de Resultados

A análise econômica está detalhada em:  
`docs/analise_resultados.md`

Principais achados:

- Os gastos com **alimentação** apresentam comportamento compatível com a **Lei de Engel**:
  - maior participação nos decis de renda mais baixos;
  - queda da participação relativa à medida que a renda aumenta.
- Os gastos com **pets**:
  - mantêm participação **relativamente estável**, em torno de **1% a 1,5%** do orçamento, ao longo dos decis de renda;
  - apresentam leve aumento em decis intermediários, mas não um crescimento contínuo nos decis mais ricos.
- Interpretação:
  - alimentação se comporta como **bem de necessidade** (share cai com a renda);
  - pets formam uma categoria **discricionária, porém “protegida”**, com peso percentual relativamente estável na cesta de consumo.

---

## 🧱 Estrutura do Repositório

> A estrutura abaixo pode variar conforme a disciplina/evolução do projeto, mas resume a organização proposta:

```text
mvp-teoria-consumidor-pets/
├─ notebooks/
│  ├─ 01_busca_coleta.ipynb         # leitura e tratamento dos dados (Bronze/Silver)
│  ├─ 02_modelagem_carga.ipynb      # criação das tabelas fato/dimensão (Gold)
│  └─ 03_analise_consumidor_pets.ipynb  # consultas de análise e exploração
├─ docs/
│  ├─ objetivo.md                   # detalhamento do problema e perguntas de negócio
│  ├─ modelo_dados.md              # descrição do esquema estrela
│  ├─ catalogo_dados.md            # dicionário de dados (fato e dimensões)
│  ├─ analise_resultados.md        # interpretação econômica dos resultados
│  └─ autoavaliacao.md             # autoavaliação do MVP
├─ data/
│  ├─ README.md                    # instruções para download dos dados na fonte oficial
│  └─ (arquivos .xlsx/.csv opcionais, conforme orientação da disciplina)
└─ README.md                       # este arquivo
