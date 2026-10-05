# Consolidado — Regressão Logística com 3 Preditoras

## 1. Objetivo

A terceira etapa de modelagem expande a Regressão Logística anterior com a inclusão da variável **`LOCAL_MORTE`**.

O objetivo é analisar a associação entre o **sexo** e três grupos de variáveis:

- `IDADE`;
- `ESCOLARIDADE`;
- `LOCAL_MORTE`.

## 2. Preparação dos dados

A variável `SEXO` foi transformada em variável binária:

- `1` = Feminino;
- `0` = Masculino.

Foram mantidos apenas registros com informações válidas de idade, escolaridade e local de morte.

A escolaridade foi organizada nas categorias:

- Nenhuma;
- 1 a 3 anos;
- 4 a 7 anos;
- 8 a 11 anos;
- 12 anos ou mais.

A categoria **Nenhuma** foi utilizada como referência.

## 3. Tratamento do local da morte

A variável `LOCAL_MORTE` foi organizada nas categorias:

- Hospital;
- Outros estabelecimentos de saúde;
- Domicílio;
- Outros;
- Via pública.

A categoria **Hospital** foi definida como referência.

Dessa forma, os coeficientes das demais categorias representam a diferença em relação aos óbitos ocorridos em hospitais.

## 4. Modelo

Foi ajustado o modelo:

`SEXO_bin ~ IDADE + ESCOLARIDADE + LOCAL_MORTE`

Assim como no modelo anterior, os dados foram divididos em:

- **70% para treinamento**;
- **30% para teste**.

A divisão utilizou `set.seed(42)` para garantir reprodutibilidade.

## 5. Odds Ratios

Os coeficientes foram transformados em **Odds Ratios (OR)** para facilitar a interpretação.

Na análise realizada, a variável `IDADE` apresentou associação positiva com `SEXO_bin`.

As categorias de escolaridade apresentaram, em geral, associações negativas em relação à categoria de referência.

Para `LOCAL_MORTE`, os resultados também indicaram diferenças importantes em relação à categoria **Hospital**.

Entre os efeitos observados, `Via pública` apresentou um dos maiores efeitos negativos, com coeficiente de aproximadamente **-1,4209** no modelo Ridge e **-1,2635** no Lasso.

No modelo logístico original, o OR para `Via pública` foi aproximadamente **0,212**, indicando chances consideravelmente menores de `SEXO_bin = 1` em comparação com a categoria Hospital, considerando as demais variáveis do modelo.

## 6. Avaliação do modelo

Foram utilizadas:

- matriz de confusão;
- acurácia no conjunto de teste;
- Pseudo R² de McFadden;
- Odds Ratios com intervalos de confiança.

O limite de classificação utilizado foi de **0,5**.

## 7. Comparação com regularização

A comparação entre GLM, Ridge e Lasso apresentou:

| Modelo | Acurácia de Teste | Desvio de Teste |
|---|---:|---:|
| GLM | 0,6032 | 1,323 |
| Ridge | 0,6038 | 1,323 |
| Lasso | 0,6057 | 1,324 |

Os três modelos apresentaram desempenho bastante semelhante.

O **Lasso apresentou a maior acurácia**, com aproximadamente **60,57%**, enquanto o GLM apresentou 60,32% e o Ridge 60,38%.

Apesar da pequena vantagem do Lasso em acurácia, GLM e Ridge apresentaram desvio marginalmente menor.

## 8. Seleção de variáveis pelo Lasso

Uma das principais características observadas no Lasso foi a capacidade de seleção de variáveis.

Com `lambda.1se`, o coeficiente correspondente à categoria **ESCOLARIDADE de 1 a 3 anos** foi reduzido a zero, tornando essa variável efetivamente removida do modelo regularizado.

Isso demonstra como a regularização Lasso pode produzir modelos mais simples e interpretáveis.

## 9. Conclusão

A inclusão de `LOCAL_MORTE` amplia a análise da relação entre sexo, idade e escolaridade, permitindo incorporar uma característica relacionada ao contexto do óbito.

Os resultados mostram que **IDADE, ESCOLARIDADE e LOCAL_MORTE** apresentam diferentes níveis de associação com a variável resposta.

Entre os modelos regularizados, o **Lasso apresentou a maior acurácia**, além de realizar seleção de variáveis. O **Ridge manteve todas as variáveis**, reduzindo seus coeficientes, enquanto o **GLM tradicional** apresentou desempenho muito próximo aos dois métodos regularizados.
