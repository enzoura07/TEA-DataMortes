# Consolidado — Regressão Logística com Ridge e Lasso

## Objetivo

Comparar a Regressão Logística tradicional (GLM) com os métodos de regularização **Ridge** e **Lasso**, buscando prever a variável `SEXO_bin` a partir das características presentes nos dados.

## 1. Preparação dos dados

Os dados foram carregados e filtrados com sucesso, resultando em **81.443 observações válidas** para a análise.

O modelo utilizou como variáveis explicativas:

- `IDADE`
- `ESCOLARIDADE`
- `LOCAL_MORTE`

A variável resposta foi `SEXO_bin`, representando o sexo de forma binária.

## 2. Modelo original — Regressão Logística (GLM)

Inicialmente, foi ajustado um modelo de Regressão Logística tradicional.

A análise das **Razões de Chance (Odds Ratios — OR)** apresentou os seguintes resultados:

- **IDADE:** OR = 1,021, indicando que cada ano adicional de idade está associado a um aumento de aproximadamente **2,1% nas chances de `SEXO_bin = 1`**, considerando a codificação utilizada.
- **ESCOLARIDADE:** em comparação com a categoria de referência **“Nenhuma escolaridade”**, os níveis mais elevados apresentaram ORs inferiores a 1, variando aproximadamente entre **0,773 e 0,517**.
- **LOCAL_MORTE:** em comparação com **“Hospital”**, locais como **“Domicílio”** apresentaram OR = 0,766, enquanto **“Via pública”** apresentou OR = 0,212.

Esses resultados indicam associações entre idade, escolaridade e local de morte e a variável resposta.

## 3. Comparação entre GLM, Ridge e Lasso

| Modelo | Acurácia de Teste | Desvio de Teste |
|---|---:|---:|
| GLM | 0,6032 | 1,323 |
| Ridge | 0,6038 | 1,323 |
| Lasso | 0,6057 | 1,324 |

Os três modelos apresentaram resultados **muito semelhantes**.

O Lasso apresentou a maior acurácia, com **60,57%**, enquanto GLM e Ridge apresentaram **60,32% e 60,38%**, respectivamente.

Apesar disso, as diferenças foram pequenas. Em relação ao desvio, **GLM e Ridge apresentaram desempenho marginalmente melhor**, com 1,323, contra 1,324 do Lasso.

## 4. Análise do Ridge

O Ridge foi ajustado utilizando **λ = `lambda.min`**.

Como esperado, o método provocou uma redução na magnitude dos coeficientes, mantendo, entretanto, **todas as variáveis no modelo**.

A variável **IDADE apresentou coeficiente positivo**, enquanto as categorias de **ESCOLARIDADE** e **LOCAL_MORTE** apresentaram, em geral, coeficientes negativos em relação às suas categorias de referência.

Entre os efeitos observados, `LOCAL_MORTE = Via pública` apresentou um dos maiores efeitos negativos, com coeficiente de aproximadamente **-1,4209**.

## 5. Análise do Lasso

O Lasso foi ajustado utilizando **λ = `lambda.1se`**, priorizando um modelo mais simples.

Nesse modelo, o coeficiente correspondente a **ESCOLARIDADE de 1 a 3 anos** foi reduzido a **zero**, fazendo com que essa variável fosse removida do modelo.

Além disso, os demais coeficientes também sofreram redução.

Novamente, `LOCAL_MORTE = Via pública` apresentou forte associação negativa, com coeficiente de aproximadamente **-1,2635**.

## 6. Conclusão

Os três modelos apresentaram **desempenho bastante semelhante**, com pequenas diferenças de acurácia e desvio.

O **Lasso apresentou a maior acurácia**, atingindo **60,57%**, além de produzir um modelo mais parcimonioso ao eliminar a variável **ESCOLARIDADE de 1 a 3 anos**.

O **Ridge**, por sua vez, manteve todas as variáveis no modelo, reduzindo apenas a magnitude dos coeficientes. Já o modelo **GLM tradicional** apresentou desempenho praticamente equivalente aos modelos regularizados.

De forma geral:

- **GLM:** apresentou desempenho muito próximo dos modelos regularizados.
- **Ridge:** manteve todas as variáveis e reduziu a magnitude dos coeficientes.
- **Lasso:** apresentou a maior acurácia e realizou seleção de variáveis.

A variável **IDADE apresentou associação positiva** com a variável resposta, enquanto **ESCOLARIDADE e LOCAL_MORTE** apresentaram, em diversas categorias, associações negativas em relação aos níveis de referência.
