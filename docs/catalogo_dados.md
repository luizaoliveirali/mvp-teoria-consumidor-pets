# Catálogo de Dados

Este catálogo descreve as principais tabelas e colunas do modelo de dados analítico utilizado no MVP, em formato de esquema estrela.  
Para cada campo, são apresentados: nome, descrição, tipo de dado, unidade (quando aplicável), valores esperados e observações sobre a origem.

---

## 1. Tabela Fato – `fato_despesa_familiar`

**Descrição geral:**  
Tabela que armazena o gasto médio semanal por domicílio em cada categoria de consumo, por grupo de renda e período.

**Grão:**  
1 linha = **[Ano de referência] × [Decil de renda] × [Categoria de consumo]** (e, opcionalmente, [Região]).

| Nome da Coluna             | Descrição                                                                                         | Tipo de Dado      | Unidade        | Valores Esperados / Domínio                                                                                 | Origem / Observações                                                                                                 |
|----------------------------|---------------------------------------------------------------------------------------------------|-------------------|----------------|-------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| `id_fato`                  | Identificador único da linha da fato (chave substituta).                                         | INTEGER           | –              | Valores inteiros sequenciais (1, 2, 3, …).                                                                  | Gerado no processo de carga (ETL) para facilitar joins e identificação de registros.                                |
| `id_renda`                | Chave estrangeira para a dimensão de renda (`dim_renda`).                                         | INTEGER           | –              | Deve corresponder a um valor existente em `dim_renda.id_renda`.                                             | Derivado da classificação em decil de renda apresentada no workbook do ONS.                                         |
| `id_categoria`            | Chave estrangeira para a dimensão de categoria de consumo (`dim_categoria_consumo`).             | INTEGER           | –              | Deve corresponder a um valor existente em `dim_categoria_consumo.id_categoria`.                             | Atribuído a partir do mapeamento das linhas de categoria do workbook.                                               |
| `id_tempo`                | Chave estrangeira para a dimensão de tempo (`dim_tempo`).                                         | INTEGER           | –              | Deve corresponder a um valor existente em `dim_tempo.id_tempo`.                                             | Normalmente um único ano (ex.: FYE 2024), mas o modelo permite evoluir para séries de anos.                         |
| `id_geografia`            | Chave estrangeira para a dimensão geográfica (`dim_geografia`).                                   | INTEGER (NULL OK) | –              | Nulo quando só houver agregado nacional; preenchido caso haja recortes regionais.                           | Opcional neste MVP; pode ser usado em extensões com recortes geográficos.                                           |
| `gasto_medio_semanal`     | Gasto médio semanal por domicílio na categoria de consumo correspondente à linha.               | DECIMAL(12,2)     | Moeda (£)      | Valores ≥ 0. Valores extremos devem ser analisados em etapa de qualidade de dados.                          | Campo numérico obtido diretamente das tabelas do workbook (expenditure per week).                                   |
| `gasto_medio_total_semanal` | Gasto médio semanal total por domicílio (soma de todas as categorias de despesa).              | DECIMAL(12,2)     | Moeda (£)      | Valores ≥ 0. Em geral, deve ser ≥ `gasto_medio_semanal`.                                                    | Pode vir de coluna específica do workbook ou ser calculado por agregação sobre todas as categorias.                 |
| `participacao_orcamento`  | Participação da categoria no orçamento total: gasto da categoria / gasto total.                  | DECIMAL(8,4)      | Proporção      | Valores entre 0 e 1 (0% a 100%).                                                                            | Calculado em ETL: `gasto_medio_semanal / gasto_medio_total_semanal`.                                                |

---

## 2. Dimensão de Renda – `dim_renda`

**Descrição geral:**  
Representa os grupos de renda (decis) utilizados pelo ONS na segmentação dos domicílios.

| Nome da Coluna       | Descrição                                                                                   | Tipo de Dado  | Unidade | Valores Esperados / Domínio                                | Origem / Observações                                                                                 |
|----------------------|---------------------------------------------------------------------------------------------|---------------|---------|------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| `id_renda`          | Identificador único da faixa de renda (chave primária).                                     | INTEGER       | –       | Inteiros sequenciais (1, 2, …).                            | Criado no ETL para servir como chave da dimensão.                                                   |
| `decil_renda`       | Número do decil de renda disponível equivalente.                                            | INTEGER       | –       | Valores de 1 a 10, onde 1 = 10% mais pobres, 10 = 10% mais ricos. | Derivado diretamente das tabelas do ONS que segmentam por decil de renda.                           |
| `descricao_decil`   | Texto descritivo da faixa de renda.                                                         | STRING        | –       | Ex.: “Decil 1 – 10% mais pobres”, “Decil 10 – 10% mais ricos”.     | Construído no ETL para facilitar leitura em análises e visualizações.                               |
| `tipo_renda`        | Tipo de renda utilizado na classificação.                                                   | STRING        | –       | Ex.: “renda disponível equivalente”.                       | Definido de acordo com a documentação metodológica do ONS.                                          |
| `faixa_renda_texto` | Descrição opcional da faixa (caso existam faixas adicionais ou faixas agregadas).          | STRING (NULL) | –       | Pode ser nulo ou conter descrição adicional definida no projeto. | Campo opcional para futuras extensões (ex.: agrupamento de decis em baixa/média/alta renda).       |

---

## 3. Dimensão de Categoria de Consumo – `dim_categoria_consumo`

**Descrição geral:**  
Contém as categorias de despesa (linhas do workbook), com identificação de quais pertencem ao grupo “pets”.

| Nome da Coluna       | Descrição                                                                                                        | Tipo de Dado  | Unidade | Valores Esperados / Domínio                                                                    | Origem / Observações
