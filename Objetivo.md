# Pets na Cesta de Consumo: Análise de Gastos com Animais de Estimação à Luz da Teoria do Consumidor

Este repositório contém o MVP desenvolvido para a disciplina de **Engenharia de Dados**, com foco em **teoria do consumidor** aplicada aos **gastos com animais de estimação (pets)**.  
O projeto implementa um pipeline de dados em nuvem para analisar como os gastos com pets são incorporados à cesta de consumo das famílias e em que medida esses gastos são priorizados ou ajustados diante de diferentes níveis de renda.

---

## 🎯 Objetivo do Trabalho

O objetivo deste MVP é analisar, à luz da teoria do consumidor, como os **gastos com animais de estimação (pets)** são incorporados à cesta de consumo das famílias e em que medida eles são priorizados ou ajustados diante de diferentes níveis de renda e de restrição orçamentária.

Para isso, foi construído um pipeline de dados em nuvem, no Databricks, a partir de dados oficiais de **despesa familiar por decil de renda** (Family Spending – ONS, FYE 2024), com destaque para a categoria de **gastos com pets** em comparação a outras categorias de consumo (como alimentação).

Do ponto de vista teórico, o trabalho se apoia em conceitos de **teoria do consumidor**, em especial:

- restrição orçamentária;
- **curvas de Engel**;
- classificação de bens em **normais, inferiores e de luxo**;
- decisões de alocação de renda entre bens essenciais e não essenciais.

A partir desses conceitos, pretende-se investigar se os gastos com pets se comportam como um bem de necessidade, de luxo ou como um item “protegido” no orçamento das famílias, mesmo em contextos de renda mais limitada.

De forma mais específica, o trabalho busca responder às seguintes **perguntas de negócio**:

1. **Participação de pets na cesta de consumo**
   - Qual é a participação média dos gastos com pets no orçamento das famílias?  
   - Como essa participação se compara a outras grandes categorias de despesa (por exemplo, alimentação)?

2. **Renda, curvas de Engel e priorização de gastos**
   - Como o valor e o **share** dos gastos com pets variam à medida que a renda das famílias aumenta (por decil de renda)?  
   - A participação dos gastos com pets no orçamento cresce, diminui ou se mantém relativamente estável entre faixas de renda?  
   - A forma da “curva de Engel” para gastos com pets sugere que eles se comportam como bens de necessidade, de luxo ou apresentam um padrão intermediário?

3. **Resiliência e trade-offs no orçamento (visão agregada)**
   - Em faixas de renda mais baixa, os gastos com pets são proporcionalmente mais comprimidos, menos comprimidos ou ajustados em intensidade semelhante a outras categorias de consumo (como alimentação)?  
   - A comparação entre o comportamento de **alimentação** e **pets** ao longo dos decis de renda traz indícios de priorização ou de forte preferência por manter o bem-estar do animal?

Algumas dessas perguntas são exploradas com maior profundidade (especialmente as relativas à participação de pets no orçamento e à comparação com alimentação), enquanto outras ficam como direcionamento para trabalhos futuros, dado que o dataset utilizado é agregado por decil e não contém microdados individuais de famílias.

---

## 📊 Dataset

- **Fonte oficial:** Office for National Statistics (ONS) – Reino Unido  
- **Pesquisa:** *Family spending in the UK*  
- **Arquivo utilizado:** *Family spending workbook 1: detailed expenditure and trends* – Tabela A6 (*Detailed household expenditure by gross income decile group*), ano **FYE 2024**  
- **Nível de agregação:** despesas médias semanais por categoria de consumo, agregadas por **decil de renda bruta**  
- **Unidade de medida:** gasto médio semanal (em libras) por categoria de despesa e por decil

Neste MVP, foi utilizada especificamente a tabela que traz:

- códigos e descrições das categorias de despesa;
- gastos médios semanais para cada decil de renda (`d1_lowest` … `d10_highest`);
- gastos médios para o agregado de todos os domicílios (`all_households`).

Os arquivos brutos (workbook do ONS) não são versionados neste repositório por questões de tamanho e licença, mas o passo a passo para leitura, limpeza e modelagem da tabela A6 está documentado nos notebooks.

---

## 🏗️ Arquitetura do Pipeline

A solução foi modelada em camadas lógicas, aproximando-se de uma abordagem de Data Warehouse (esquema estrela) construída diretamente em SQL no Databricks:

1. **Camada Bronze – Dados brutos (importação inicial)**  
   - Leitura da tabela A6 do arquivo Excel do ONS (*workbook 1*), com os valores de despesa por decil de renda.  
   - Criação da tabela `family_spending_a6` no Databricks, refletindo a estrutura original (colunas de decil em formato “wide”).

2. **Camada Silver – Dados tratados / estruturados**  
   - Criação da tabela `family_spending_a6_clean`, com:
     - padronização de nomes de colunas (`d1_lowest`, `d2_second`, ..., `d10_highest`, `all_households`);
     - seleção de colunas relevantes (`code`, `description`, gastos por decil).
   - Transformação do formato “wide” para “long” por meio da view `vw_despesa_long`, em que cada linha representa uma combinação **categoria × decil de renda**, com a respectiva despesa média semanal.
   - Cálculo do gasto médio total semanal por decil de renda e da variável `participacao_orcamento` (share da categoria no orçamento total do decil), na view `vw_despesa_com_total`.

3. **Camada Gold – Modelo analítico (esquema estrela)**  
   - Criação das dimensões:
     - `dim_renda` – informações por decil de renda (id_renda, decil, descrição);
     - `dim_categoria_consumo` – categorias de despesa (código, descrição, flag para categorias relacionadas a pets);
     - `dim_tempo` – período de referência (FYE 2024);
     - `dim_geografia` – agregado Reino Unido.
   - Criação da tabela fato `fato_despesa_familiar`, contendo:
     - chaves das dimensões (renda, categoria, tempo, geografia);
     - `gasto_medio_semanal` (por categoria × decil);
     - `gasto_medio_total_semanal` (total do decil);
     - `participacao_orcamento` (share da categoria no orçamento do decil).

Essa camada Gold é a base para as análises de teoria do consumidor, como:

- participação dos gastos com pets por decil de renda;
- comparação entre a participação de alimentação e de pets no orçamento (Lei de Engel);
- discussão sobre a classificação de pets como bem de necessidade, de luxo ou categoria de consumo relativamente estável.

---

## 🗂️ Estrutura do Repositório

```text
mvp-teoria-consumidor-pets/
├─ notebooks/
│  ├─ 01_busca_coleta.ipynb
├─ docs/
│  ├─ objetivo.md
│  ├─ modelo_dados.md
│  ├─ catalogo_dados.md
│  ├─ analise_resultados.md
│  └─ autoavaliacao.md
├─ img/
│  ├─ databricks_pipeline.png
│  ├─ exemplo_tabela_gold.png
│  └─ grafico_curva_engel_pets.png
├─ data/
│  └─ README.md   # explicando a fonte dos dados e instruções para obtê-los
└─ Objetivo.md # Este arquivo
