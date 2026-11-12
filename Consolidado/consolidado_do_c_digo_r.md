# Consolidado do Código R: Análise de PCA em Dados de Mortalidade

Este documento apresenta a consolidação da rotina em linguagem R utilizada para a leitura, tratamento, execução de Análise de Componentes Principais (PCA) e extração de métricas sobre o conjunto de dados de mortalidade.

---

## 1. Carregamento de Pacotes e Leitura dos Dados

```R
# Carregamento das bibliotecas necessárias
library(tidyverse)
library(caret)

# Leitura do conjunto de dados
# Subsitua 'dados_mortes.csv' pelo caminho real do seu arquivo
df <- read.csv("dados_mortes.csv", stringsAsFactors = TRUE)

# Visualização inicial da estrutura
str(df)
summary(df)
```

---

## 2. Pré-processamento e Encoding

Tratamento de variáveis categóricas via One-Hot Encoding e remoção de colunas com variância próxima de zero ou valores ausentes.

```R
# Seleção das variáveis de interesse para o PCA
# Exemplo considerando variáveis sociodemográficas e espaciais
var_interesse <- c("MUNICIPIO", "RACACOR", "ESC", "ESTCIV", "LOCOCOR", "IDADE")

df_pca_raw <- df %>% 
  select(all_of(var_interesse)) %>% 
  na.omit()

# Separação da variável alvo/continua (IDADE) se desejado para validação cruzada
idade_vector <- df_pca_raw$IDADE
df_categoricas <- df_pca_raw %>% select(-IDADE)

# One-Hot Encoding das variáveis categóricas
dummys <- dummyVars(" ~ .", data = df_categoricas)
df_encoded <- data.frame(predict(dummys, newdata = df_categoricas))

# Remoção de variáveis com variância zero ou nula
nzv <- nzv(df_encoded)
if(length(nzv) > 0) {
  df_encoded <- df_encoded[, -nzv]
}

# Inclusão da idade padronizada caso se deseje rodar o PCA com ela
df_encoded_full <- df_encoded
df_encoded_full$IDADE <- scale(idade_vector)
```

---

## 3. Execução da Análise de Componentes Principais (PCA)

Aplicação do `prcomp` centralizando e escalonando os dados binários/contínuos.

```R
# Execução do PCA
pca_result <- prcomp(df_encoded, center = TRUE, scale. = TRUE)

# Resumo da variância explicada pelas componentes
pca_summary <- summary(pca_result)
print(pca_summary$importance[, 1:10]) # Primeiras 10 componentes
```

---

## 4. Extração de Métricas e Diagnósticos

Cálculo da variância acumulada, peso das variáveis nas componentes (loadings) e correlação com a idade.

```R
# 1. Variância explicada por componente
var_explicada <- pca_summary$importance[2, ] # Proporção da variância
var_acumulada <- pca_summary$importance[3, ] # Variância acumulada

cat("Variância Explicada por PC1:", round(var_explicada[1] * 100, 2), "%\n")
cat("Variância Acumulada (PC1 a PC10):", round(var_acumulada[10] * 100, 2), "%\n")

# 2. Principais variáveis que impactam a PC1 (Loadings)
loadings_pc1 <- pca_result$rotation[, 1]
top_loadings_pc1 <- sort(abs(loadings_pc1), decreasing = TRUE)[1:10]
cat("\nTop 10 Variáveis com maior peso na PC1:\n")
print(top_loadings_pc1)

# 3. Correlação dos Scores do PCA com a Variável Idade
scores_pca <- data.frame(pca_result$x)

cor_pc1_idade <- cor(scores_pca$PC1, idade_vector)
cor_pc2_idade <- cor(scores_pca$PC2, idade_vector)

cat("\nCorrelação de Pearson com a IDADE:\n")
cat("PC1 x Idade:", round(cor_pc1_idade, 4), "\n")
cat("PC2 x Idade:", round(cor_pc2_idade, 4), "\n")
```

---

## 5. Visualização (Scree Plot)

```R
# Gráfico de Variância Acumulada
plot(var_acumulada[1:20], type = "b", pch = 19, col = "blue",
     xlab = "Número de Componentes Principais",
     ylab = "Proporção da Variância Acumulada",
     main = "Scree Plot - Variância Acumulada (Top 20 PCs)")
abline(h = 0.70, col = "red", lty = 2) # Linha de corte de 70%
```