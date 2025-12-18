# Análise dos Resultados

## 1. Qualidade dos dados

Antes de avançar para a análise econômica, foi realizada uma checagem de qualidade dos dados na tabela fato `fato_despesa_familiar`.

- Foram identificados alguns valores nulos em `gasto_medio_semanal` e, consequentemente, em `participacao_orcamento`, relacionados a combinações específicas de categoria × decil em que o workbook não reporta valor. Esses casos foram tratados na etapa de análise substituindo nulos por zero nas agregações (uso de `COALESCE` em SQL), de forma a não distorcer os shares totais.
- Os valores mínimos e máximos de `gasto_medio_semanal` se mantêm em faixa não-negativa, variando aproximadamente entre **0,1** e **961,6 libras** por semana. 
- A variável `participacao_orcamento` apresentou valores entre **0,00002** e **0,18** (cerca de 0% a 18% do orçamento), isto é, dentro do intervalo esperado entre 0 e 1 (0% a 100% do orçamento), com shares máximos em torno de 17,9% do orçamento para as categorias de maior peso.

Esses resultados indicam que o conjunto de dados está consistente para uso nas análises de teoria do consumidor, sem necessidade de tratamentos adicionais mais pesados (remoção de outliers extremos, imputação etc.), desde que o tratamento dos nulos seja devidamente considerado nas agregações.

## 2. Participação dos gastos com pets no orçamento por faixa de renda

A partir da junção entre a tabela fato e as dimensões de renda e categorias, foi calculada a participação dos gastos com pets no orçamento total de cada decil de renda. 

A análise foi realizada somando a variável `participacao_orcamento` apenas para as categorias identificadas com `flag_pet = 1` (ex.: *"Pets and pet food"*). O resultado pode ser resumido da seguinte forma:

- No **1º decil de renda (10% mais pobres)**, a participação dos gastos com pets no orçamento médio semanal é de aproximadamente **1,1%**.
- No **decil intermediário (5º decil)**, essa participação sobe para cerca de **1,5%**.
- No **10º decil de renda (10% mais ricos)**, a participação retorna para algo em torno de **1,2%**.

Observa-se, portanto, que a participação de gastos com pets se mantém **relativamente estável**, variando em torno de 1% a 1,5% ao longo dos decis de renda: há um leve aumento entre os decis intermediários, mas sem um crescimento contínuo nos decis mais ricos. Isso indica que os gastos com animais de estimação não se comportam como um bem estritamente de primeira necessidade, mas também não exibem um padrão de bem de luxo forte, mantendo uma importância percentual relativamente constante na cesta de consumo das famílias.

## 3. Comparação com a categoria de alimentação (Lei de Engel)

Para contextualizar os resultados de pets em relação à teoria do consumidor, também foi analisada a participação da categoria de **alimentação** no orçamento, usando as linhas de despesa cujo texto de descrição contém "food" e agrupando sob o rótulo "Alimentação".

A comparação entre os grupos "Pets" e "Alimentação" por decil de renda mostra que:

- A participação da **alimentação** tende a ser **mais alta nos decis de renda mais baixos** e a diminuir à medida que a renda aumenta, comportamento alinhado à **Lei de Engel**, segundo a qual a proporção da renda gasta em alimentação cai com o aumento da renda.
- Já a participação de **gastos com pets** apresenta um comportamento **moderadamente estável**, variando em torno de 1% a 1,5% do orçamento ao longo dos decis de renda: sai de cerca de 1,1% no 1º decil, atinge aproximadamente 1,5% em decis intermediários e volta a ficar próxima de 1,2% no 10º decil.

Do ponto de vista da teoria do consumidor, esses resultados são coerentes com a ideia de que:

- a categoria **Alimentação** se aproxima de um **bem de necessidade**, cuja proporção no orçamento cai com a renda, ainda que o gasto absoluto aumente;
- a categoria **Pets** não se comporta como um bem estritamente essencial, mas também não apresenta o padrão típico de bem de luxo, no qual o share cresce fortemente com a renda. Em vez disso, os gastos com pets parecem compor uma categoria **discricionária, porém relativamente “protegida”**, mantendo um peso percentual estável na cesta de consumo à medida que a renda aumenta.

## 4. Discussão geral

De forma geral, o modelo de dados construído permite interpretar o comportamento dos gastos com animais de estimação em um contexto de cesta de consumo agregada, ainda que com base em dados médios por decil de renda.

Mesmo com a limitação de não trabalhar com microdados individuais de domicílios, o uso de decis de renda como unidade de análise é suficiente para identificar padrões consistentes com a teoria do consumidor, especialmente:

- a relação entre renda e composição da cesta de consumo;
- o contraste entre bens de primeira necessidade (alimentação) e categorias mais discricionárias (pets).

Esses resultados abrem espaço para trabalhos futuros, como:

- incorporar mais de um ano de dados para analisar séries temporais;
- detalhar a categoria de pets em subitens (ração, serviços veterinários, acessórios etc.);
- estimar elasticidades-renda aproximadas a partir da variação do share de cada decil.
