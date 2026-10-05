# Consolidado — Regressão Logística com 2 Preditoras

## 1. Objetivo

A Regressão Logística foi utilizada para investigar a relação entre o **sexo** dos indivíduos registrados no SIM e duas variáveis preditoras:

- `IDADE`;
- `ESCOLARIDADE`.

O objetivo é modelar a probabilidade de o registro pertencer à categoria **Feminino** ou **Masculino**.

## 2. Preparação da variável resposta

A variável `SEXO` foi transformada em uma variável binária denominada `SEXO_bin`:

- `1` = Feminino;
- `0` = Masculino.

Foram removidos registros com sexo inválido ou ausente, idade ausente e escolaridade não informada ou classificada como `Ignorado`.

## 3. Tratamento da escolaridade

A variável `ESCOLARIDADE` foi transformada em fator com as seguintes categorias:

- Nenhuma;
- 1 a 3 anos;
- 4 a 7 anos;
- 8 a 11 anos;
- 12 anos ou mais.

A categoria **Nenhuma** foi definida como referência para a interpretação dos coeficientes.

## 4. Modelo

Foi ajustado o modelo:

`SEXO_bin ~ IDADE + ESCOLARIDADE`

A divisão dos dados foi realizada em:

- **70% para treinamento**;
- **30% para teste**.

Foi utilizada `set.seed(42)` para garantir reprodutibilidade da divisão.

## 5. Interpretação por Odds Ratio

Os coeficientes do modelo logístico podem ser transformados em **Odds Ratios (OR)** por meio de `exp(β)`.

O OR da variável `IDADE` representa a alteração nas chances de `SEXO_bin = 1` para cada aumento de um ano na idade.

Para `ESCOLARIDADE`, os ORs representam as diferenças nas chances em relação à categoria de referência **Nenhuma escolaridade**.

## 6. Avaliação do modelo

As previsões foram convertidas em classes utilizando um limite de probabilidade de **0,5**.

Foram utilizadas:

- matriz de confusão;
- acurácia no conjunto de teste;
- Pseudo R² de McFadden.

A matriz de confusão permite avaliar os acertos e erros das classificações entre Feminino e Masculino.

## 7. Comparação com regularização

Na análise regularizada realizada anteriormente, a regressão logística tradicional, Ridge e Lasso apresentaram desempenho muito próximo.

| Modelo | Acurácia de Teste | Desvio de Teste |
|---|---:|---:|
| GLM | 0,6032 | 1,323 |
| Ridge | 0,6038 | 1,323 |
| Lasso | 0,6057 | 1,324 |

O Lasso apresentou a maior acurácia entre os três modelos, embora a diferença tenha sido pequena.

## 8. Conclusão

A Regressão Logística com duas preditoras permite avaliar como **idade e escolaridade** estão associadas à classificação do sexo.

O modelo também estabelece a base para a expansão da análise com uma terceira variável explicativa, `LOCAL_MORTE`, permitindo avaliar se a inclusão do local do óbito acrescenta informação ao modelo.
