# Catálogo de Dados

Este catálogo descreve as principais tabelas e colunas do modelo de dados analítico utilizado no MVP, em formato de esquema estrela.  
Para cada campo, são apresentados: nome, descrição, tipo de dado, unidade (quando aplicável), valores esperados e observações sobre a origem.

---

## 1. Tabela Fato – `fato_despesa_familiar`

**Descrição geral:**  
Tabela que armazena o gasto médio semanal por domicílio em cada categoria de consumo, por grupo de renda e período.

**Grão:**  
1 linha = **[Ano de referência] × [Decil de renda] × [Categoria de consumo]** (no MVP, sempre para o agregado geográfico “UK total”).

| Nome da Coluna               | Descrição                                                                                         | Tipo de Dado  | Unidade   | Valores Esperados / Domínio                                                                 | Origem / Observações                                                                                                 |
|------------------------------|---------------------------------------------------------------------------------------------------|--------------|-----------|---------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| `id_fato`                    | Identificador único da linha da fato (chave substituta).                                         | INTEGER      | –         | Valores inteiros sequenciais (1, 2, 3, …).                                                  | Gerado no processo de carga (ETL) para facilitar joins e identificação de registros.                                |
| `id_renda`                   | Chave estrangeira para a dimensão de renda (`dim_renda`).                                        | INTEGER      | –         | Deve corresponder a um valor existente em `dim_renda.id_renda`.                             | Derivado da classificação em decil de renda apresentada no workbook do ONS.                                         |
| `id_categoria`               | Chave estrangeira para a dimensão de categoria de consumo (`dim_categoria_consumo`).             | INTEGER      | –         | Deve corresponder a um valor existente em `dim_categoria_consumo.id_categoria`.             | Atribuído a partir do mapeamento das linhas de categoria do workbook.                                               |
| `id_tempo`                   | Chave estrangeira para a dimensão de tempo (`dim_tempo`).                                        | INTEGER      | –         | Deve corresponder a um valor existente em `dim_tempo.id_tempo`.                             | No MVP, representa o ano FYE 2024, mas o modelo permite evoluir para séries de anos.                                |
| `id_geografia`               | Chave estrangeira para a dimensão geográfica (`dim_geografia`).                                  | INTEGER      | –         | Atualmente assume o valor 1 (agregado “UK total”).                                          | Preenchido no ETL; reservado para possíveis extensões com recortes regionais.                                       |
| `gasto_medio_semanal`        | Gasto médio semanal por domicílio na categoria de consumo correspondente à linha.                | DOUBLE / DECIMAL | Moeda (£) | Valores ≥ 0. Valores extremos devem ser analisados em etapa de qualidade de dados.          | Campo numérico obtido diretamente das colunas de decil da tabela A6 do workbook (expenditure per week).             |
| `gasto_medio_total_semanal`  | Gasto médio semanal total por domicílio (soma de todas as categorias de despesa do decil).      | DOUBLE / DECIMAL | Moeda (£) | Valores ≥ 0. Em geral, deve ser ≥ `gasto_medio_semanal`.                                    | Calculado no ETL por agregação do `gasto_medio_semanal` de todas as categorias dentro do mesmo ano e decil de renda. |
| `participacao_orcamento`     | Participação da categoria no orçamento total: gasto da categoria / gasto total do decil.         | DOUBLE / DECIMAL | Proporção | Valores entre 0 e 1 (0% a 100%).                                                            | Calculado em ETL: `gasto_medio_semanal / gasto_medio_total_semanal`. Em agregações, nulos são tratados como zero.   |

---

## 2. Dimensão de Renda – `dim_renda`

**Descrição geral:**  
Representa os grupos de renda (decis) utilizados pelo ONS na segmentação dos domicílios.

| Nome da Coluna       | Descrição                                                                                   | Tipo de Dado  | Unidade | Valores Esperados / Domínio                                | Origem / Observações                                                                                 |
|----------------------|---------------------------------------------------------------------------------------------|---------------|---------|------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| `id_renda`           | Identificador único da faixa de renda (chave primária).                                     | INTEGER       | –       | Inteiros sequenciais (1, 2, …).                            | Criado no ETL para servir como chave da dimensão.                                                   |
| `decil_renda`        | Número do decil de renda bruta.                                                             | INTEGER       | –       | Valores de 1 a 10, onde 1 = 10% mais pobres, 10 = 10% mais ricos. | Derivado diretamente das tabelas do ONS que segmentam por decil de **gross income**.                |
| `descricao_decil`    | Texto descritivo da faixa de renda.                                                         | STRING        | –       | Ex.: “Decil 1 – 10% mais pobres”, “Decil 10 – 10% mais ricos”. | Construído no ETL para facilitar leitura em análises e visualizações.                               |
| `tipo_renda`         | Tipo de renda utilizado na classificação.                                                   | STRING        | –       | Ex.: “renda bruta por decil (gross income decile group)”.  | Definido de acordo com a documentação metodológica da tabela A6 do ONS.                             |
| `faixa_renda_texto`  | Descrição opcional da faixa (caso existam faixas adicionais ou agregadas).                  | STRING (NULL) | –       | Pode ser nulo ou conter descrição adicional definida no projeto. | Campo opcional para futuras extensões (ex.: agrupamento de decis em baixa/média/alta renda).       |

---

## 3. Dimensão de Categoria de Consumo – `dim_categoria_consumo`

**Descrição geral:**  
Contém as categorias de despesa (linhas do workbook), com identificação de quais pertencem ao grupo “pets”.

| Nome da Coluna       | Descrição                                                                                                        | Tipo de Dado  | Unidade | Valores Esperados / Domínio                                                                    | Origem / Observações                                                                                             |
|----------------------|------------------------------------------------------------------------------------------------------------------|---------------|---------|------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `id_categoria`       | Identificador único da categoria de consumo (chave primária).                                                   | INTEGER       | –       | Inteiros sequenciais (1, 2, …).                                                                | Gerado no ETL.                                                                                                   |
| `codigo_categoria`   | Código da categoria no arquivo do ONS.                                                                          | STRING        | –       | Códigos como os apresentados na tabela A6 (ex.: “A06CC150P”).                                 | Lido diretamente do workbook.                                                                                    |
| `descricao_categoria`| Nome da categoria de gasto.                                                                                     | STRING        | –       | Ex.: “Pets and pet food”, “Veterinary and other services for pets”, “Food and non-alcoholic drinks” etc. | Lido diretamente do workbook.                                                                                    |
| `flag_pet`           | Indicador se a categoria está diretamente relacionada a gastos com pets.                                        | INTEGER (0/1) | –       | 1 = categoria de pets; 0 = demais categorias.                                                 | Definido no ETL a partir de regras de texto (ex.: `LOWER(descricao_categoria) LIKE '%pet%'`).                   |

---

## 4. Dimensão de Tempo – `dim_tempo`

**Descrição geral:**  
Representa o período de referência da pesquisa (ano fiscal – Financial Year Ending).

| Nome da Coluna   | Descrição                                                 | Tipo de Dado | Unidade | Valores Esperados / Domínio          | Origem / Observações                                      |
|------------------|-----------------------------------------------------------|-------------|---------|--------------------------------------|-----------------------------------------------------------|
| `id_tempo`       | Identificador único do período (chave primária).         | INTEGER     | –       | Inteiro (no MVP, sempre 1).          | Criado no ETL.                                            |
| `ano`            | Ano de referência do levantamento.                        | INTEGER     | –       | Ex.: 2024.                           | Definido conforme ano da tabela utilizada (FYE 2024).     |
| `periodo_label`  | Rótulo textual do período.                                | STRING      | –       | Ex.: “FYE 2024”.                     | Construído no ETL.                                        |
| `ano_inicio`     | Data de início do ano fiscal.                             | DATE        | –       | Ex.: 2023-04-01.                     | Data aproximada conforme definição do ano fiscal.         |
| `ano_fim`        | Data de fim do ano fiscal.                                | DATE        | –       | Ex.: 2024-03-31.                     | Data aproximada conforme definição do ano fiscal.         |

---

## 5. Dimensão Geográfica – `dim_geografia`

**Descrição geral:**  
Dimensão geográfica simples, pois o MVP utiliza apenas o agregado nacional do Reino Unido.

| Nome da Coluna       | Descrição                                               | Tipo de Dado | Unidade | Valores Esperados / Domínio | Origem / Observações                                   |
|----------------------|---------------------------------------------------------|-------------|---------|-----------------------------|--------------------------------------------------------|
| `id_geografia`       | Identificador único da região (chave primária).        | INTEGER     | –       | Inteiro (no MVP, sempre 1). | Criado no ETL.                                         |
| `regiao`             | Nome da região ou agregação geográfica.                | STRING      | –       | Ex.: “UK total”.            | Definido no ETL.                                       |
| `descricao_geografia`| Texto adicional sobre a agregação geográfica.          | STRING      | –       | Ex.: “Agregado Reino Unido”.| Campo descritivo para facilitar leitura das análises.  |

---

Esse catálogo fecha muito bem a documentação do seu DWzinho de pets 🐶📊  
Se você quiser, me manda agora o texto de `analise_resultados.md` que eu faço a última revisão fina e você fica com o pacote inteiro pronto pra subir no GitHub.
::contentReference[oaicite:0]{index=0}
