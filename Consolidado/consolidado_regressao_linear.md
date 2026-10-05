# Consolidado — Regressão Linear

## 1. Objetivo

A Regressão Linear Múltipla foi utilizada para analisar a relação entre a **idade no momento do óbito** e as variáveis **escolaridade, ano e sexo**.

O objetivo é verificar como essas características estão associadas à idade registrada no óbito e utilizar o modelo para realizar previsões.

## 2. Preparação e engenharia de variáveis

Os dados foram carregados a partir do arquivo `DatasetSIM_tratado.csv`.

A variável `ESCOLARIDADE` foi transformada em uma representação numérica aproximada em anos:

| Escolaridade | Anos atribuídos |
|---|---:|
| Nenhuma | 0 |
| 1 a 3 anos | 2 |
| 4 a 7 anos | 5,5 |
| 8 a 11 anos | 9,5 |
| 12 anos ou mais | 12 |

A variável `SEXO` foi transformada em fator, utilizando **Feminino** como categoria de referência e **Masculino** como categoria comparada.

Foram mantidos apenas os registros com informações válidas para escolaridade, idade, ano e sexo.

## 3. Modelo

Foi ajustado o modelo:

`IDADE ~ ESCOLARIDADE_ANOS + ANO + SEXO`

A variável resposta é:

- `IDADE`

As variáveis preditoras são:

- `ESCOLARIDADE_ANOS`;
- `ANO`;
- `SEXO`.

## 4. Previsões

Após o ajuste do modelo, foram realizadas previsões para diferentes combinações de escolaridade, ano e sexo.

Foram considerados exemplos com:

- escolaridade de 0 anos;
- escolaridade de 5,5 anos;
- escolaridade de 12 anos;
- ano de 2023;
- diferentes categorias de sexo.

As previsões permitem estimar a idade esperada no momento do óbito para cada combinação de características.

## 5. Regularização — Ridge e Lasso

Além do modelo MQO tradicional, foram avaliadas versões regularizadas com **Ridge** e **Lasso**.

Na comparação realizada anteriormente:

| Modelo | RMSE Treino | RMSE Teste |
|---|---:|---:|
| MQO | 16,81 | 16,79 |
| Ridge | 16,81 | 16,79 |
| Lasso | 16,87 | 16,83 |

O **MQO e o Ridge apresentaram desempenho praticamente idêntico**, enquanto o Lasso apresentou um pequeno aumento no erro.

O Ridge apresentou coeficientes próximos aos do MQO, com redução de sua magnitude. Já o Lasso realizou seleção de variáveis, reduzindo o coeficiente de `ANO` a zero no modelo com `lambda.1se`.

## 6. Conclusão

A Regressão Linear mostrou que é possível modelar a idade no momento do óbito a partir de escolaridade, ano e sexo.

A comparação com métodos regularizados indicou que o **Ridge manteve desempenho equivalente ao MQO**, enquanto o **Lasso produziu um modelo mais parcimonioso**, eliminando a contribuição da variável `ANO`, embora com pequeno aumento do erro de previsão.

Assim, o Ridge é adequado para regularizar os coeficientes sem remover variáveis, enquanto o Lasso é especialmente útil quando o objetivo inclui seleção de variáveis.
