# Análise GAM da Idade ao Morrer (Baixada Santista)

Documento consolidado do notebook `spline.ipynb` (R / Google Colab). Reúne o código, os resultados obtidos e a interpretação dos gráficos.

## Resumo

- **Objetivo:** modelar a idade ao morrer (`IDADE`) em função de tempo, sazonalidade, escolaridade, sexo, raça, município e local do óbito.
- **Dados:** `dataset_final.xlsx` / `.csv` com 474.651 registros de óbitos (2020 em diante). Após a limpeza, restaram **458.897** linhas.
- **Modelos comparados (validação cruzada 5-fold):** linha de base (média), regressão linear (`lm`) e GAM rápido (`bam`, pacote `mgcv`).
- **Resultado principal:** o GAM teve o menor RMSE (14,71), seguido do `lm` (14,76) e da linha de base (16,86). O ganho do GAM sobre o `lm` é pequeno.

| Modelo | RMSE médio (5-fold) |
|---|---|
| Linha de base (média do treino) | 16,8554 |
| Regressão linear (`lm`) | 14,7603 |
| GAM (`bam`) | **14,7129** |

---

## 1. Carga e inspeção dos dados

Carrega os pacotes, lê o CSV (ou o Excel, se o CSV não existir), identifica a coluna de data de óbito e converte as datas tentando dois formatos (`dd/mm/aaaa` e `aaaa-mm-dd`).

```r
if(!require(mgcv)) install.packages("mgcv", repos="https://cloud.r-project.org")
if(!require(readxl)) install.packages("readxl", repos="https://cloud.r-project.org")
library(mgcv)
library(readxl)
library(splines)


filepath_xlsx <- "/content/dataset_final.xlsx"
filepath_csv <- "/content/dataset_final.csv"

if (file.exists(filepath_csv)) {
  cat("Lendo arquivo CSV...\n")
  df <- read.csv(filepath_csv, check.names = FALSE)
} else if (file.exists(filepath_xlsx)) {
  cat("Lendo arquivo Excel e convertendo para DataFrame...\n")
  df <- as.data.frame(read_excel(filepath_xlsx))
} else {
  stop("Arquivo de dados não encontrado!")
}


cat("\n--- Primeiras linhas ---\n")
print(head(df))

cat("\n--- Valores únicos de ESCOLARIDADE ---\n")
print(unique(df$ESCOLARIDADE))



col_data <- grep("DATA.*OBITO|DATA.*ÓBITO", colnames(df), ignore.case = TRUE, value = TRUE)[1]
if(is.na(col_data)) col_data <- "DATA DE ÓBITO" # fallback

cat("\nColuna de data identificada como:", col_data, "\n")


dates_dmy <- as.Date(as.character(df[[col_data]]), format = "%d/%m/%Y")
dates_ymd <- as.Date(as.character(df[[col_data]]), format = "%Y-%m-%d")

final_dates <- dates_dmy
na_dmy <- sum(is.na(dates_dmy))
na_ymd <- sum(is.na(dates_ymd))

cat(sprintf("NAs usando %%d/%%m/%%Y: %d\n", na_dmy))
cat(sprintf("NAs usando %%Y-%%m-%%d: %d\n", na_ymd))


final_dates[is.na(final_dates)] <- dates_ymd[is.na(final_dates)]
na_sobrando <- sum(is.na(final_dates))
cat(sprintf("Total de NAs restantes na DATA DE ÓBITO após tentar ambos os formatos: %d\n", na_sobrando))

df$DATA_OBITO_CONV <- final_dates
```

**Saída:**

```text
Lendo arquivo Excel e convertendo para DataFrame...

--- Primeiras linhas ---
   ANO DATA DE ÓBITO IDADE      SEXO   RAÇA    ESCOLARIDADE MUNICIPIO
1 2020    2020-12-05    24 Masculino  Parda 12 anos ou mais   Guarujá
2 2020    2020-12-05    24 Masculino  Parda 12 anos ou mais   Guarujá
3 2020    2020-12-05    24 Masculino  Parda 12 anos ou mais   Guarujá
4 2020    2020-12-05    24 Masculino  Parda 12 anos ou mais   Guarujá
5 2020    2020-03-04    19 Masculino Branca      4 a 7 anos    Santos
6 2020    2020-06-07    49  Feminino  Parda         Nenhuma   Guarujá
  LOCAL_MORTE                    OCUP
1    Hospital Auxiliar administrativo
2    Hospital     Auxiliar de compras
3    Hospital  Auxiliar de escritório
4    Hospital            Escriturário
5 Via pública            Motofretista
6    Hospital                    <NA>
                                                   CAUSA BASICA DA MORTE
1 Afogamento e submersão em águas naturais - outros locais especificados
2 Afogamento e submersão em águas naturais - outros locais especificados
3 Afogamento e submersão em águas naturais - outros locais especificados
4 Afogamento e submersão em águas naturais - outros locais especificados
5         Exposição a corrente elétrica não especificada - rua e estrada
6                           Insuficiência renal crônica não especificada

--- Valores únicos de ESCOLARIDADE ---
[1] "12 anos ou mais" "4 a 7 anos"      "Nenhuma"         "8 a 11 anos"    
[5] "Ignorado"        "1 a 3 anos"      NA               

Coluna de data identificada como: DATA DE ÓBITO 
NAs usando %d/%m/%Y: 474651
NAs usando %Y-%m-%d: 0
Total de NAs restantes na DATA DE ÓBITO após tentar ambos os formatos: 0
```

> As datas estavam todas no formato `aaaa-mm-dd`. O formato `dd/mm/aaaa` gerou NA em todos os registros.

---

## 2. Engenharia de variáveis e limpeza

Cria as variáveis do modelo:

- `dia_ano`: dia do ano (1 a 366), para a sazonalidade.
- `tempo`: dias decorridos desde 01/01/2020, para a tendência.
- `anos_estudo`: escolaridade convertida em valor numérico (ponto médio de cada faixa).
- Fatores para sexo, raça, município e local da morte.

Depois remove registros com idade, sexo, raça ou escolaridade ausentes/ignorados.

| Faixa de escolaridade | `anos_estudo` |
|---|---|
| Nenhuma | 0 |
| 1 a 3 anos | 2 |
| 4 a 7 anos | 5,5 |
| 8 a 11 anos | 9,5 |
| 12 anos ou mais | 13 |

```r
# dia_ano: dia do ano (1 a 366)
df$dia_ano <- as.numeric(format(df$DATA_OBITO_CONV, "%j"))

# tempo: dias desde 2020-01-01
df$tempo <- as.numeric(df$DATA_OBITO_CONV - as.Date("2020-01-01"))

# anos_estudo: Nenhuma=0, 1 a 3 anos=2, 4 a 7 anos=5.5, 8 a 11 anos=9.5, 12 anos e mais=13

esc_clean <- trimws(tolower(as.character(df$ESCOLARIDADE)))

df$anos_estudo <- NA
df$anos_estudo[grepl("nenhuma", esc_clean)] <- 0
df$anos_estudo[grepl("1 a 3", esc_clean)] <- 2
df$anos_estudo[grepl("4 a 7", esc_clean)] <- 5.5
df$anos_estudo[grepl("8 a 11", esc_clean)] <- 9.5
df$anos_estudo[grepl("12 anos e mais|12 anos ou mais", esc_clean)] <- 13

# Fatores para SEXO, RACA, MUNICIPIO e LOCAL_MORTE
get_col_name <- function(pattern, cols) {
  res <- grep(pattern, cols, ignore.case = TRUE, value = TRUE)
  if(length(res) > 0) return(res[1]) else return(NULL)
}

col_sexo <- get_col_name("^SEXO$", colnames(df))
col_raca <- get_col_name("^RACA$|^RAÇA$", colnames(df))
col_municipio <- get_col_name("^MUNICIPIO$|^MUNICÍPIO$", colnames(df))
col_local_morte <- get_col_name("^LOCAL_MORTE$|^LOCAL.*MORTE$", colnames(df))
col_idade <- get_col_name("^IDADE$", colnames(df))

df$SEXO_f <- as.factor(df[[col_sexo]])
df$RACA_f <- as.factor(df[[col_raca]])
df$MUNICIPIO_f <- as.factor(df[[col_municipio]])
df$LOCAL_MORTE_f <- as.factor(df[[col_local_morte]])


linhas_iniciais <- nrow(df)

# Condição de remoção para IDADE, SEXO, RACA e ESCOLARIDADE
idade_valida <- !is.na(df[[col_idade]]) & df[[col_idade]] != "Ignorado" & df[[col_idade]] != ""
sexo_valido <- !is.na(df[[col_sexo]]) & df[[col_sexo]] != "Ignorado" & df[[col_sexo]] != ""
raca_valida <- !is.na(df[[col_raca]]) & df[[col_raca]] != "Ignorado" & df[[col_raca]] != ""
escolaridade_valida <- !is.na(df$anos_estudo) # os ignorados ou incompatíveis viraram NA aqui

data_valida <- !is.na(df$DATA_OBITO_CONV)

d_filtrado <- df[idade_valida & sexo_valido & raca_valida & escolaridade_valida & data_valida, ]

# Converter idade para numerico
d_filtrado$IDADE_num <- as.numeric(as.character(d_filtrado[[col_idade]]))
d_filtrado <- d_filtrado[!is.na(d_filtrado$IDADE_num), ]

linhas_removidas <- linhas_iniciais - nrow(d_filtrado)
cat(sprintf("Linhas iniciais: %d\n", linhas_iniciais))
cat(sprintf("Linhas restantes: %d\n", nrow(d_filtrado)))
cat(sprintf("Linhas removidas: %d\n", linhas_removidas))
```

**Saída:**

```text
Linhas iniciais: 474651
Linhas restantes: 458897
Linhas removidas: 15754
```

---

## 3. Validação cruzada (5-fold)

Compara três abordagens, medindo o RMSE em cada dobra:

1. **Linha de base:** prediz sempre a média da idade no treino.
2. **`lm`:** regressão linear com todas as covariáveis.
3. **`bam`:** GAM rápido, com splines para `tempo`, `dia_ano` (cíclica, `bs = "cc"`) e `anos_estudo` (`k = 4`).

Como a base é grande, a validação usa uma amostra aleatória de 100 mil linhas (`set.seed(1)`).

```r
set.seed(1)


if (nrow(d_filtrado) > 100000) {
  cat("Dataset muito grande (~458 mil linhas). Rodando validação cruzada em amostra de 100 mil linhas para maior agilidade.\n")
  idx_amostra <- sample(1:nrow(d_filtrado), 100000)
  d_cv <- d_filtrado[idx_amostra, ]
} else {
  d_cv <- d_filtrado
}

dobra <- sample(rep(1:5, length = nrow(d_cv)))

rmse_baseline <- numeric(5)
rmse_lm <- numeric(5)
rmse_bam <- numeric(5)

for(i in 1:5) {
  treino <- d_cv[dobra != i, ]
  teste  <- d_cv[dobra == i, ]

  # a) Linha de base (média do treino)
  pred_base <- mean(treino$IDADE_num)
  rmse_baseline[i] <- sqrt(mean((teste$IDADE_num - pred_base)^2))

  # b) lm
  m_lm <- lm(IDADE_num ~ tempo + dia_ano + anos_estudo + SEXO_f + RACA_f + MUNICIPIO_f + LOCAL_MORTE_f, data = treino)
  pred_lm <- predict(m_lm, newdata = teste)
  rmse_lm[i] <- sqrt(mean((teste$IDADE_num - pred_lm)^2, na.rm=TRUE))

  # c) bam (GAM rápido do mgcv)
  m_bam <- bam(IDADE_num ~ s(tempo) + s(dia_ano, bs = "cc") + s(anos_estudo, k = 4) +
                 SEXO_f + RACA_f + MUNICIPIO_f + LOCAL_MORTE_f, data = treino, discrete = TRUE)
  pred_bam <- predict(m_bam, newdata = teste)
  rmse_bam[i] <- sqrt(mean((teste$IDADE_num - pred_bam)^2, na.rm=TRUE))
}

cat(sprintf("RMSE Médio - Linha de Base: %.4f\n", mean(rmse_baseline)))
cat(sprintf("RMSE Médio - Regressão Linear (lm): %.4f\n", mean(rmse_lm)))
cat(sprintf("RMSE Médio - GAM (bam): %.4f\n", mean(rmse_bam)))
```

**Saída:**

```text
Dataset muito grande (~458 mil linhas). Rodando validação cruzada em amostra de 100 mil linhas para maior agilidade.
RMSE Médio - Linha de Base: 16.8554
RMSE Médio - Regressão Linear (lm): 14.7603
RMSE Médio - GAM (bam): 14.7129
```

---

## 4. Modelo final (GAM) e gráficos

Ajusta o `bam` em todos os dados filtrados e plota as três curvas suavizadas.

```r
# Passo 4 e 5: Ajuste do BAM final em todos os dados e Plotagem
g_final <- bam(IDADE_num ~ s(tempo) + s(dia_ano, bs = "cc") + s(anos_estudo, k = 4) +
                 SEXO_f + RACA_f + MUNICIPIO_f + LOCAL_MORTE_f, data = d_filtrado, discrete = TRUE)

print(summary(g_final))

# Plotando as 3 curvas suavizadas
options(repr.plot.width = 12, repr.plot.height = 4)
plot(g_final, pages = 1, shade = TRUE, scheme = 1)
```

**Saída (`summary`):**

```text
Family: gaussian 
Link function: identity 

Formula:
IDADE_num ~ s(tempo) + s(dia_ano, bs = "cc") + s(anos_estudo, 
    k = 4) + SEXO_f + RACA_f + MUNICIPIO_f + LOCAL_MORTE_f

Parametric coefficients:
                                               Estimate Std. Error  t value
(Intercept)                                    72.48816    0.24981  290.169
SEXO_fMasculino                                -2.63014    0.04955  -53.076
RACA_fBranca                                   -5.43667    0.17683  -30.745
RACA_fIndígena                                 -4.75458    1.04095   -4.568
RACA_fParda                                   -13.91055    0.18093  -76.882
RACA_fPreta                                   -12.15210    0.19643  -61.864
MUNICIPIO_fCubatão                              6.37393    0.20747   30.722
MUNICIPIO_fGuarujá                              0.36313    0.18335    1.981
MUNICIPIO_fItanhaém                             8.85422    0.18162   48.751
MUNICIPIO_fMongaguá                             2.88989    0.23262   12.423
MUNICIPIO_fPeruíbe                              4.15593    0.23379   17.777
MUNICIPIO_fPraia Grande                         7.69905    0.17253   44.623
MUNICIPIO_fSantos                              11.44784    0.16932   67.611
MUNICIPIO_fSão Vicente                          7.40909    0.17830   41.555
LOCAL_MORTE_fHospital                           0.05306    0.06127    0.866
LOCAL_MORTE_fIgnorado                          -7.63863    3.14504   -2.429
LOCAL_MORTE_fOutros                            -5.83073    0.14311  -40.742
LOCAL_MORTE_fOutros estabelecimentos de saúde  -1.56706    0.07650  -20.484
LOCAL_MORTE_fVia pública                      -21.04635    0.14993 -140.375
                                               Pr(>|t|)    
(Intercept)                                    < 2e-16 ***
SEXO_fMasculino                                < 2e-16 ***
RACA_fBranca                                   < 2e-16 ***
RACA_fIndígena                                4.94e-06 ***
RACA_fParda                                    < 2e-16 ***
RACA_fPreta                                    < 2e-16 ***
MUNICIPIO_fCubatão                             < 2e-16 ***
MUNICIPIO_fGuarujá                              0.0476 *  
MUNICIPIO_fItanhaém                            < 2e-16 ***
MUNICIPIO_fMongaguá                            < 2e-16 ***
MUNICIPIO_fPeruíbe                             < 2e-16 ***
MUNICIPIO_fPraia Grande                        < 2e-16 ***
MUNICIPIO_fSantos                              < 2e-16 ***
MUNICIPIO_fSão Vicente                         < 2e-16 ***
LOCAL_MORTE_fHospital                           0.3864    
LOCAL_MORTE_fIgnorado                           0.0151 *  
LOCAL_MORTE_fOutros                            < 2e-16 ***
LOCAL_MORTE_fOutros estabelecimentos de saúde  < 2e-16 ***
LOCAL_MORTE_fVia pública                       < 2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Approximate significance of smooth terms:
                 edf Ref.df        F p-value    
s(tempo)       8.947  8.999   204.06  <2e-16 ***
s(dia_ano)     7.663  8.000    24.54  <2e-16 ***
s(anos_estudo) 2.999  3.000 14727.54  <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

R-sq.(adj) =  0.239   Deviance explained = 23.9%
fREML = 1.8862e+06  Scale est. = 217.46    n = 458897
```

> O gráfico com as três curvas suavizadas (`s(tempo)`, `s(dia_ano)`, `s(anos_estudo)`) é gerado ao executar o notebook. A imagem não está incluída neste arquivo.

### Leitura rápida dos coeficientes paramétricos

Cada coeficiente é a diferença estimada na idade ao morrer, em anos, em relação à categoria de referência do fator (a primeira em ordem alfabética, que não aparece na tabela; para sexo, o feminino), mantendo o restante constante.

- **Sexo:** homens morrem em média 2,6 anos mais cedo.
- **Raça:** pessoas pardas (-13,9) e pretas (-12,2) têm as maiores diferenças negativas. Brancas (-5,4) e indígenas (-4,8) vêm em seguida.
- **Município:** Santos (+11,4), Itanhaém (+8,9), Praia Grande (+7,7) e São Vicente (+7,4) têm as maiores idades médias estimadas.
- **Local da morte:** via pública está associada a idades 21 anos menores. Hospital não difere significativamente da referência (p = 0,39).
- **Ajuste geral:** R² ajustado de 0,239, ou seja, o modelo explica cerca de 24% da variação na idade ao morrer.

---

## 5. Análise dos gráficos de efeitos suavizados (GAM)

Os três gráficos representam os termos não lineares *s(tempo)*, *s(dia_ano)* e *s(anos_estudo)*. O eixo vertical mostra a contribuição (positiva ou negativa) de cada variável para a **idade ao morrer** estimada. A área sombreada representa o intervalo de confiança.

### 5.1 Tendência temporal: `s(tempo)`

- **O que o gráfico mostra:** uma linha com oscilações suaves (ondas) e uma leve inclinação geral para cima ao longo do tempo (dias decorridos desde o início de 2020).
- **Interpretação:**
  - Existe uma oscilação cíclica de longo prazo na idade média de óbito.
  - A tendência de subida indica que, de forma geral e muito sutil, a idade média ao morrer na amostra aumentou discretamente ao longo dos anos analisados.

### 5.2 Sazonalidade anual: `s(dia_ano)`

- **O que o gráfico mostra:** uma linha quase horizontal e muito próxima de zero durante todo o intervalo de 1 a 365 dias.
- **Interpretação:**
  - O efeito do dia do ano é estatisticamente significante devido ao tamanho massivo da amostra, mas seu efeito prático na idade do óbito é **muito pequeno** (quase nulo).
  - As estações do ano não exercem impacto visualmente expressivo na idade em que as pessoas morrem nesta base de dados.

### 5.3 Impacto da educação: `s(anos_estudo)`

- **O que o gráfico mostra:** uma curva descendente muito acentuada. O efeito começa positivo (acima de zero) para quem tem baixíssima ou nenhuma escolaridade (0 a 2 anos) e cai de forma íngreme para valores negativos conforme os anos de estudo aumentam (chegando a cerca de -8 para 12+ anos).
- **Interpretação:**
  - **Efeito de coorte (geracional):** o gráfico não significa que estudar reduz a expectativa de vida. Ele reflete uma realidade demográfica histórica brasileira: as pessoas mais idosas que estão falecendo hoje (80 ou 90 anos) nasceram na primeira metade do século XX, quando o acesso à escola era extremamente restrito. Assim, a imensa maioria dos óbitos em idades muito avançadas se concentra na faixa de baixa escolaridade.
  - Por outro lado, indivíduos mais jovens que vêm a óbito (por exemplo, por causas externas ou doenças precoces) nasceram em gerações recentes, com muito mais acesso ao ensino formal, concentrando-se nas faixas de escolaridade mais altas.

---

## Observações metodológicas

- O modelo descreve **idade ao morrer entre pessoas que morreram**, e não expectativa de vida da população. Os coeficientes não devem ser lidos como risco de morte ou efeito causal.
- A composição etária por escolaridade e raça é afetada por efeito de coorte, o que explica os sinais contraintuitivos de `anos_estudo` e de raça.
- A validação cruzada usou uma amostra de 100 mil linhas. O modelo final usa as 458.897 linhas.
- O ganho do GAM sobre o `lm` no RMSE é pequeno (cerca de 0,05 ano), o que sugere que as não linearidades de `tempo` e `dia_ano` têm pouco poder preditivo adicional.
