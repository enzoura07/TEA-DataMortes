# Consolidado — Regressão Linear com Ridge e Lasso

## Objetivo

Comparar o desempenho da Regressão Linear por Mínimos Quadrados Ordinários (MQO) com os métodos de regularização **Ridge** e **Lasso**, utilizando como variável resposta a **idade**.

## 1. Leitura e preparação dos dados

Os dados do arquivo `datasetDEF_translated.xlsx` foram carregados e filtrados com sucesso, resultando em **81.448 observações válidas** para a análise.

As variáveis utilizadas como preditoras foram:

- `ESCOLARIDADE_ANOS`
- `ANO`
- `SEXO`

## 2. Modelo original — MQO

Inicialmente, foi ajustado um modelo de Regressão Linear por **Mínimos Quadrados Ordinários (MQO)**. O modelo foi utilizado para realizar previsões de idade a partir de diferentes combinações de escolaridade, ano e sexo.

Os coeficientes obtidos foram:

| Variável | Coeficiente |
|---|---:|
| ESCOLARIDADE_ANOS | -0,9914 |
| ANO | 0,2466 |
| SEXOMasculino | -6,5469 |

## 3. Comparação entre MQO, Ridge e Lasso

Para realizar uma comparação equivalente, os dados foram divididos em **70% para treinamento e 30% para teste**.

| Modelo | RMSE Treino | RMSE Teste |
|---|---:|---:|
| MQO | 16,81 | 16,79 |
| Ridge | 16,81 | 16,79 |
| Lasso | 16,87 | 16,83 |

Os resultados mostram que **MQO e Ridge apresentaram praticamente o mesmo desempenho**, enquanto o Lasso apresentou um erro ligeiramente maior. Isso indica que a regularização Ridge teve pouco impacto sobre a capacidade preditiva do modelo, enquanto o Lasso simplificou o modelo com um pequeno custo em precisão.

## 4. Análise do Ridge

O Ridge utilizou **λ = 0,4001 (`lambda.min`)**.

Seus coeficientes permaneceram próximos aos obtidos pelo MQO, porém sofreram uma pequena contração em direção a zero, como esperado pela regularização Ridge.

Exemplos:

- **ANO:** 0,2415
- **SEXOMasculino:** -6,4185

O Ridge reduz a magnitude dos coeficientes, mas **não elimina variáveis do modelo**.

## 5. Análise do Lasso

O Lasso utilizou **λ = 0,9911 (`lambda.1se`)**, escolhido por produzir um modelo mais parcimonioso.

Os principais coeficientes foram:

| Variável | Coeficiente |
|---|---:|
| ESCOLARIDADE_ANOS | -0,7484 |
| ANO | 0 |
| SEXOMasculino | -4,7767 |

O coeficiente da variável **ANO foi reduzido a zero**, fazendo com que essa variável fosse retirada do modelo.

Esse resultado demonstra uma das principais características do Lasso: além de regularizar os coeficientes, ele pode realizar **seleção de variáveis**.

## 6. Conclusão

A comparação mostrou que **MQO e Ridge apresentaram o melhor desempenho preditivo**, ambos com RMSE de **16,79 no conjunto de teste**.

O Lasso apresentou RMSE de **16,83**, ligeiramente superior, mas produziu um modelo mais simples ao eliminar a variável **ANO**.

Dessa forma:

- **MQO:** apresentou o melhor desempenho junto ao Ridge.
- **Ridge:** manteve todas as variáveis e reduziu a magnitude dos coeficientes.
- **Lasso:** apresentou pequeno aumento do erro, mas proporcionou maior simplificação e seleção de variáveis.

Assim, o **Ridge** mostrou-se adequado quando o objetivo é manter as variáveis no modelo e controlar a magnitude dos coeficientes, enquanto o **Lasso** é interessante quando se busca um modelo mais simples e parcimonioso.
