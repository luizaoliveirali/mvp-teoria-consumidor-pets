# Autoavaliação

## 1. Atingimento dos objetivos

O objetivo deste MVP foi analisar a participação dos gastos com animais de estimação (pets) na cesta de consumo das famílias, à luz da teoria do consumidor, utilizando um pipeline de dados em nuvem. A partir dos dados oficiais de orçamento familiar do Office for National Statistics (ONS), busquei responder principalmente:

- qual é a participação dos gastos com pets no orçamento total por faixa de renda;  
- como essa participação se compara a categorias clássicas, como alimentação;  
- se o comportamento dos gastos com pets se aproxima mais de um bem de necessidade, de luxo ou de uma categoria relativamente estável na cesta de consumo.

Na minha visão, os objetivos foram **atingidos de forma satisfatória**. Consegui:

- construir um modelo de dados em esquema estrela (fato + dimensões de renda, categoria, tempo e geografia);  
- implementar a camada de transformação e análise em SQL no Databricks;  
- calcular o share de pets no orçamento por decil de renda;  
- comparar esse share com o de alimentação e discutir os resultados à luz da teoria do consumidor (Lei de Engel, bens essenciais x discricionários).

Algumas análises complementares que imaginei inicialmente (como comparação em mais de um ano ou detalhamento de subcategorias de pets) não foram incluídas, principalmente por questão de tempo e escopo. Ainda assim, considero que o MVP responde bem às perguntas centrais definidas no objetivo.

## 2. Dificuldades encontradas

As principais dificuldades estiveram ligadas à **parte de engenharia de dados na nuvem**, especialmente porque foi meu primeiro contato prático com o Databricks:

- Tive dificuldade inicial com a interface do Databricks Community (diferença entre SQL Warehouse e cluster de compute, escolha da linguagem do notebook e forma correta de anexar o compute ao notebook).
- A preparação dos dados exigiu entender o formato do workbook do ONS, que não está pronto para análise direta (títulos, cabeçalhos múltiplos, colunas de decil em formato wide). Isso demandou uma etapa de limpeza prévia e a estruturação em um formato “long” (categoria × decil).
- Encontrei valores nulos em `gasto_medio_semanal` e, consequentemente, em `participacao_orcamento`, o que exigiu decisões de tratamento. Optei por utilizar `COALESCE` nas agregações, tratando valores nulos como zero na etapa de análise, e documentei essa escolha na seção de qualidade de dados.

Mesmo com essas dificuldades, considero que elas contribuíram para o aprendizado. Hoje me sinto mais confiante para:

- trabalhar com dados em nuvem;  
- criar tabelas e views em SQL no Databricks;  
- estruturar pipelines simples com camadas lógicas (Bronze → Silver → Gold).

## 3. Aprendizados

Ao longo do desenvolvimento do MVP, tive alguns aprendizados importantes:

- Do ponto de vista técnico, aprendi a:
  - fazer upload de dados e criar tabelas no Databricks via interface (Catalog);
  - transformar tabelas “largas” (com colunas de decil) em formato analítico “longo” usando SQL;
  - modelar um esquema estrela e implementar dimensões e fato diretamente em SQL;
  - analisar qualidade de dados (nulos, faixas mínimas e máximas, coerência de shares) e refletir sobre impacto nas conclusões.

- Do ponto de vista conceitual em economia/teoria do consumidor, pude:
  - observar empiricamente a Lei de Engel para alimentação (queda da participação relativa nos decis mais ricos);
  - analisar como os gastos com pets se comportam em relação à renda, com share relativamente estável em torno de 1%–1,5% do orçamento, sugerindo um bem discricionário mas com peso percentual protegido dentro da cesta de consumo.

Esse cruzamento entre teoria econômica e prática de engenharia de dados foi um ponto alto do trabalho para mim.

## 4. Trabalhos futuros

Há várias possibilidades de evolução deste MVP que poderiam enriquecer o portfólio:

- **Séries temporais**: incluir mais anos da pesquisa Family Spending (FYE 2023, FYE 2022, etc.) para analisar a evolução dos gastos com pets ao longo do tempo, incluindo períodos de choque (pandemia, crises econômicas).
- **Maior detalhamento de categorias**: separar gastos com pets em subgrupos (ração, serviços veterinários, acessórios) caso a base permita, para entender qual componente pesa mais no orçamento.
- **Elasticidades aproximadas**: utilizar a variação do share entre decis como proxy para elasticidade-renda, aprofundando a classificação de pets como bem de luxo, normal ou de necessidade.
- **Visualizações**: integrar o modelo com uma ferramenta de visualização (por exemplo, Power BI ou dashboards SQL do próprio Databricks) para construir gráficos mais intuitivos para públicos não técnicos.

## 5. Conclusão pessoal

De forma geral, estou satisfeita com o resultado do MVP. Consegui sair de um cenário de pouca familiaridade com a plataforma em nuvem para a construção de um pipeline funcional, com modelagem, carga e análise consistentes com o problema proposto. 

Embora tenha havido limitações de tempo e escopo, especialmente na exploração de outras dimensões de análise, o trabalho atingiu os objetivos principais e me trouxe um aprendizado prático importante tanto em **engenharia de dados** quanto na **aplicação da teoria do consumidor** a um tema atual e próximo da realidade de muitas famílias: os gastos com animais de estimação.

