# Pets na Cesta de Consumo: Análise de Gastos com Animais de Estimação à Luz da Teoria do Consumidor

Este repositório contém o MVP desenvolvido para a disciplina de **Engenharia de Dados**, com foco em **teoria do consumidor** aplicada aos **gastos com animais de estimação (pets)**.  
O projeto implementa um pipeline de dados em nuvem para analisar como os gastos com pet são incorporados à cesta de consumo das famílias, e em que medida esses gastos são priorizados ou ajustados diante de diferentes níveis de renda.

---

## 🎯 Objetivo do Trabalho

O objetivo deste MVP é analisar, à luz da teoria do consumidor, como os **gastos com animais de estimação (pets)** são incorporados à cesta de consumo das famílias e em que medida eles são priorizados ou ajustados diante de diferentes níveis de renda e de restrição orçamentária. 

Para isso, é construído um pipeline de dados em nuvem que integra e trata bases públicas contendo informações de **renda** e **despesa das famílias**, com destaque para a categoria de **gastos com pet** em comparação a outras categorias de consumo (alimentação, habitação, transporte, lazer, etc.).

Do ponto de vista teórico, o trabalho se apoia em conceitos de **teoria do consumidor**, em especial:

- restrição orçamentária;
- **curvas de Engel**;
- classificação de bens em **normais, inferiores e de luxo**;
- decisões de alocação de renda entre bens essenciais e não essenciais.

A partir desses conceitos, pretende-se investigar se os gastos com pet se comportam como um bem de necessidade, de luxo ou como um item “protegido” no orçamento das famílias, mesmo em contextos de renda mais limitada.

De forma mais específica, o trabalho busca responder às seguintes **perguntas de negócio**:

1. **Participação de pets na cesta de consumo**
   - Qual é a participação média dos gastos com pet no orçamento das famílias?  
   - Como essa participação se compara às demais grandes categorias de despesa (por exemplo, alimentação, habitação, transporte, lazer)?

2. **Renda, curvas de Engel e priorização de gastos**
   - Como o valor e o **share** dos gastos com pet variam à medida que a renda das famílias aumenta?  
   - A participação dos gastos com pet no orçamento cresce, diminui ou se mantém relativamente estável entre faixas de renda?  
   - A forma da “curva de Engel” para gastos com pet sugere que eles se comportam como bens de necessidade, de luxo ou apresentam um padrão intermediário?

3. **Composição interna dos gastos com pet (quando a base permitir)**
   - Dentro da categoria “pet”, como se distribuem os gastos entre alimentação (ração, petiscos), cuidados de saúde (veterinário, medicamentos), higiene/beleza e produtos/serviços de lazer?  
   - Famílias de faixas de renda diferentes apresentam cestas de consumo de pet com composições distintas (por exemplo, maior peso de serviços e produtos premium para rendas mais altas)?

4. **Resiliência e trade-offs no orçamento**
   - Em faixas de renda mais baixa, os gastos com pet são reduzidos proporcionalmente mais, menos ou em intensidade semelhante a outras categorias de consumo?  
   - Há indícios de que as famílias ajustam primeiro outras despesas (como lazer ou bens supérfluos) antes de reduzir gastos com pet, sugerindo priorização ou forte preferência por manter o bem-estar do animal?

Mesmo que nem todas as perguntas sejam respondidas integralmente, elas definem o escopo analítico deste MVP e servem como planejamento do trabalho, orientando o desenho do pipeline de dados e a análise final dos resultados.

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

A solução foi pensada em camadas, seguindo uma abordagem próxima a Data Lake / Data Warehouse:

1. **Camada Bronze (Dados Brutos)**  
   - Armazena os arquivos conforme baixados da fonte original (CSV, Parquet etc.), com o mínimo de transformação possível.

2. **Camada Silver (Dados Tratados / Integrados)**  
   - Limpeza de nulos e inconsistências;  
   - Ajustes de tipos;  
   - Cálculo de renda total, despesa total e agregações por categoria;  
   - Identificação dos gastos com pets e outras grandes categorias de consumo.

3. **Camada Gold (Modelo Analítico / DW)**  
   - Modelo em **esquema estrela**, com:
     - tabela fato de **despesa por família × categoria** (incluindo pet);
     - dimensões de **família/renda**, **categoria de consumo**, **tempo** e, quando disponível, **região**.
   - Essa camada é usada diretamente na análise de teoria do consumidor (curvas de Engel, shares, etc.).

---

## 🗂️ Estrutura do Repositório

```text
mvp-teoria-consumidor-pets/
├─ notebooks/
│  └─ 01_busca_coleta.ipynb
├─ docs/
│  ├─ modelo_dados.md
│  ├─ catalogo_dados.md
│  ├─ analise_resultados.md
│  └─ autoavaliacao.md
├─ data/
│  └─ README.md   # explicando a fonte dos dados e instruções para obtê-los
└─ README.md
