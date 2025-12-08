# Modelo de Dados – Esquema Estrela

Este documento descreve o modelo de dados analítico utilizado no MVP, em formato de **esquema estrela**, inspirado em um ambiente de **Data Warehouse**. O objetivo é facilitar as análises de **teoria do consumidor** sobre a participação dos gastos com pets no orçamento das famílias, por faixa de renda.

---

## Visão geral

A partir do dataset bruto (Family spending workbook 1 – ONS), o modelo foi organizado em:

- **1 tabela fato principal**: `fato_despesa_familiar`
- **4 dimensões** principais:
  - `dim_renda`
  - `dim_categoria_consumo`
  - `dim_tempo`
  - `dim_geografia` (simplificada, pois o MVP utiliza o agregado “UK total”)

A unidade de análise da tabela fato é:

> **Um registro por combinação de:**  
> **(Ano / período, grupo de renda, categoria de consumo)**  
> contendo o gasto médio semanal nessa categoria e a participação da categoria no orçamento.

---

## Tabela Fato: `fato_despesa_familiar`

**Descrição:**  
Tabela que armazena o gasto médio semanal das famílias com cada categoria de consumo, por grupo de renda e período.

**Grão (granularidade):**

- 1 linha = **[Ano do levantamento] × [Decil de renda] × [Categoria de consumo]**

**Campos principais:**

- `id_fato` – chave substituta (inteiro)  
- `id_renda` – chave estrangeira para `dim_renda`  
- `id_categoria` – chave estrangeira para `dim_categoria_consumo`  
- `id_tempo` – chave estrangeira para `dim_tempo`  
- `id_geografia` – chave estrangeira para `dim_geografia`

- `gasto_medio_semanal` – gasto médio semanal por domicílio naquela categoria (em libras, £)  
- `gasto_medio_total_semanal` – gasto médio semanal total por domicílio (todas as categorias somadas)  
- `participacao_orcamento` – participação da categoria no gasto total:  
  - `participacao_orcamento = gasto_medio_semanal / gasto_medio_total_semanal`

**Observação:**  
A partir dessa tabela é possível:

- calcular a **participação de pets** no orçamento por decil de renda;  
- comparar com outras categorias (por exemplo, alimentação);  
- aproximar uma “curva de Engel” para o grupo de consumo “pets”, observando como `participacao_orcamento` varia com a renda.

---

## Dimensão de Renda: `dim_renda`

**Descrição:**  
Representa os **grupos de renda** (decis) e suas características derivadas.

**Campos:**

- `id_renda` – chave primária  
- `decil_renda` – número do decil (1 a 10)  
- `descricao_decil` – texto, por exemplo “Decil 1 – 10% mais pobres”, “Decil 10 – 10% mais ricos”  
- `tipo_renda` – tipo de renda usado na classificação (neste MVP, “renda bruta por decil”, conforme definição do ONS)

**Uso nas análises:**

- Permite comparar a participação dos gastos com pets entre os decis de renda;  
- Auxilia na interpretação econômica (pobres x ricos, classes de renda).

---

## Dimensão de Categoria de Consumo: `dim_categoria_consumo`

**Descrição:**  
Representa as categorias de despesa (linhas do workbook), baseadas na classificação utilizada pelo ONS.

**Campos:**

- `id_categoria` – chave primária  
- `codigo_categoria` – código da categoria no arquivo do ONS  
- `descricao_categoria` – nome da categoria de gasto, por exemplo:
  - “Pets and pet food”
  - “Veterinary and other services for pets”
  - “Food and non-alcoholic drinks”
  - “Transport”
  - etc.  
- `flag_pet` – indicador (0/1) se a categoria está diretamente relacionada a gastos com pets  

**Uso nas análises:**

- Permite selecionar e agrupar categorias:
  - comparar **pets** versus outras categorias (como alimentação);  
  - somar todas as categorias com `flag_pet = 1` para obter o gasto total com pets;  
  - analisar a composição interna da categoria “pets”, caso haja detalhamento suficiente.

---

## Dimensão de Tempo: `dim_tempo`

**Descrição:**  
Representa o período de referência da pesquisa.

Como o workbook é anual (Financial Year Ending), a dimensão é simples neste MVP.

**Campos:**

- `id_tempo` – chave primária  
- `ano` – ano de referência (ex.: 2024)  
- `periodo_label` – texto, ex.: “FYE 2024”  
- `ano_inicio` – data de início do ano fiscal  
- `ano_fim` – data de fim do ano fiscal  

**Uso nas análises:**

- Neste MVP é utilizado apenas um ano (FYE 2024);  
- A dimensão permite **futuras extensões**, como comparação de vários anos (evolução dos gastos com pets ao longo do tempo).

---

## Dimensão Geográfica: `dim_geografia`

**Descrição:**  
Dimensão geográfica simples, pois o MVP utiliza o agregado nacional do Reino Unido.

**Campos:**

- `id_geografia` – chave primária  
- `regiao` – nome da região (neste MVP, “UK total”)  
- `descricao_geografia` – texto adicional sobre a agregação geográfica

---

## Relacionamentos

Resumo dos relacionamentos do modelo estrela:

- `fato_despesa_familiar.id_renda` → `dim_renda.id_renda`  
- `fato_despesa_familiar.id_categoria` → `dim_categoria_consumo.id_categoria`  
- `fato_despesa_familiar.id_tempo` → `dim_tempo.id_tempo`  
- `fato_despesa_familiar.id_geografia` → `dim_geografia.id_geografia`

---

## Justificativa do modelo

A escolha de um **esquema estrela** com uma tabela fato de despesa e dimensões de renda, categoria, tempo e geografia é adequada porque:

- reflete a **lógica de orçamento do consumidor**, em que as famílias distribuem sua renda entre diferentes categorias de gasto;  
- facilita consultas típicas de **Business Intelligence e teoria do consumidor**, como:
  - participação de uma categoria no orçamento por faixa de renda;  
  - comparação de categorias (pets vs alimentação);  
  - análise de como uma categoria se comporta ao longo do espectro de renda (interpretação tipo “curva de Engel” para pets);  
- é compatível com ferramentas analíticas (SQL, visualizações, dashboards), caso o MVP seja estendido.

Esse modelo foi implementado sobre os dados tratados (camada Silver) na plataforma de nuvem, formando a **camada Gold** do pipeline de dados.
