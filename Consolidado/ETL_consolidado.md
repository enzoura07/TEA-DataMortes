# Consolidado: Tratamento dos dados do SIM 2020 (R)

Script em R que importa a base de mortalidade do SIM (Sistema de Informações sobre Mortalidade), decodifica variáveis, cruza com tabelas auxiliares (municípios, CBO e CID-10) e exporta um CSV tratado.

---

## 1. Visão geral

| Item | Descrição |
|---|---|
| Linguagem | R |
| Pacotes | `tidyverse`, `lubridate`, `janitor` |
| Entradas | 4 arquivos CSV |
| Saída | `SIM_tratado.csv` |
| Fluxo | Importação → Tratamento → 3 Joins → Seleção final → Exportação |

```
SIM_2020.csv ──► limpeza/decodificação ──► + municípios ──► + CBO ──► + CID ──► SIM_tratado.csv
```

---

## 2. Arquivos de entrada

| Constante | Arquivo | Encoding | Conteúdo |
|---|---|---|---|
| `ARQ_SIM` | `SIM_2020.csv` | Latin1 | Base principal de óbitos (`na = c("", "NA", "NULL")`) |
| `ARQ_CBO` | `CBO_2002.csv` | Latin1 | Classificação Brasileira de Ocupações |
| `ARQ_MUN` | `municipios.csv` | UTF-8 | Tabela de municípios |
| `ARQ_CID` | `CID10.csv` | UTF-8 | Classificação Internacional de Doenças (CID-10) |

Todas as bases passam por `janitor::clean_names()` (nomes em minúsculas, snake_case).

---

## 3. Tratamento das variáveis

Criadas dentro de `mutate()` na base `sim`.

### 3.1 Data
| Nova variável | Origem | Regra |
|---|---|---|
| `DATA_OBITO` | `dtobito` | `dmy()` (formato dia/mês/ano) |

### 3.2 Sexo (`SEXO_DESC`)
| Código | Descrição |
|---|---|
| `M`, `1` | Masculino |
| `F`, `2` | Feminino |
| `I`, `0`, `9` | Ignorado |
| outros | `NA` |

### 3.3 Raça/Cor (`RACA_DESC`)
| Código | Descrição |
|---|---|
| 1 | Branca |
| 2 | Preta |
| 3 | Amarela |
| 4 | Parda |
| 5 | Indígena |
| outros | Ignorado |

### 3.4 Escolaridade (`ESC_DESC`)
| Código | Descrição |
|---|---|
| 1 | Nenhuma |
| 2 | 1 a 3 anos |
| 3 | 4 a 7 anos |
| 4 | 8 a 11 anos |
| 5 | 12 anos ou mais |
| 9 | Ignorado |
| outros | `NA` |

### 3.5 Local de ocorrência (`LOCAL_MORTE`)
| Código | Descrição |
|---|---|
| 1 | Hospital |
| 2 | Outros estabelecimentos de saúde |
| 3 | Domicílio |
| 4 | Via pública |
| 5 | Outros |
| 6 | Aldeia indígena |
| 9 | Ignorado |
| outros | `NA` |

### 3.6 Idade (`IDADE_ANOS`)
O campo `idade` é lido como texto: o **1º dígito** indica a unidade e os **2 dígitos seguintes** o valor.

| Unidade (1º dígito) | Conversão para anos |
|---|---|
| 1 | `valor / (365.25 * 24 * 60)` |
| 2 | `valor / (365.25 * 24)` |
| 3 | `valor / 12` |
| 4 | `valor` |
| 5 | `100` |
| outros | `NA` |

Variáveis auxiliares: `idade_cod`, `idade_unidade`, `idade_valor`.

---

## 4. Joins

| Etapa | Tabela | Chave (SIM → tabela) | Campo usado depois |
|---|---|---|---|
| 3 | `municipios` | `codmunres` → `codigo_municipio` | `nome_municipio` |
| 4 | `cbo` | `ocup` → `codigo_cbo` | `descricao_cbo` |
| 5 | `cid` | `causabas` → `codigo_cid` | `descricao_causa` |

Todos são `left_join()`, preservando todas as linhas do SIM.

---

## 5. Dataset final (`dados_final`)

Montado com `transmute()` (mantém apenas as colunas abaixo):

| Coluna final | Origem |
|---|---|
| `ANO` | `as.integer(ano)` |
| `DATA_DE_OBITO` | `DATA_OBITO` |
| `IDADE` | `IDADE_ANOS` |
| `SEXO` | `SEXO_DESC` |
| `RACA` | `RACA_DESC` |
| `ESCOLARIDADE` | `ESC_DESC` |
| `MUNICIPIO` | `nome_municipio` |
| `LOCAL_MORTE` | `LOCAL_MORTE` |
| `OCUP` | `descricao_cbo` |
| `BASICA_DA_MORTE` | `descricao_causa` |

## 6. Exportação

```r
write_csv(dados_final, "SIM_tratado.csv", na = "")
```

Valores ausentes são gravados como células vazias.

---

## 7. Pontos de atenção

1. **Código de unidade da idade.** No dicionário oficial do SIM, o 1º dígito é: `0` = minutos, `1` = horas, `2` = dias, `3` = meses, `4` = anos, `5` = 100 anos ou mais. O script trata `1` como minutos e `2` como horas, o que parece deslocado. Sugestão de correção:
   ```r
   IDADE_ANOS = case_when(
     idade_unidade == "0" ~ idade_valor / (365.25 * 24 * 60),
     idade_unidade == "1" ~ idade_valor / (365.25 * 24),
     idade_unidade == "2" ~ idade_valor / 365.25,
     idade_unidade == "3" ~ idade_valor / 12,
     idade_unidade == "4" ~ idade_valor,
     idade_unidade == "5" ~ 100 + idade_valor,
     TRUE ~ NA_real_
   )
   ```
   Confira no dicionário da sua versão da base antes de aplicar.
2. **Tipo de colunas na leitura.** `read_csv` pode inferir `sexo`, `racacor`, `esc` e `lococor` como numéricas; a comparação com strings (`"1"`) ainda funciona, mas vale usar `col_types = cols(.default = "c")` para garantir consistência (inclusive para `codmunres`, `ocup` e `causabas`, que são chaves de join).
3. **Chaves dos joins.** Os códigos de município (6 vs. 7 dígitos), CBO e CID (ex.: `I21.9` vs. `I219`) precisam ter o mesmo formato nas duas tabelas; caso contrário o join gera `NA` silenciosamente.
4. **Duplicatas nas tabelas auxiliares.** Se alguma tabela tiver chaves repetidas, o `left_join` multiplica linhas. Verifique com `count(chave) %>% filter(n > 1)`.
5. **Nomes das colunas.** O script assume que `nome_municipio`, `descricao_cbo` e `descricao_causa` existem após `clean_names()`; ajuste se os nomes reais forem diferentes.
6. **Ano.** `ano` precisa existir no SIM; se não existir, pode ser derivado de `year(DATA_OBITO)`.

---

## 8. Código completo

```r
library(tidyverse)
library(lubridate)
library(janitor)

# ============================================================
# CONFIGURAÇÃO
# ============================================================

ARQ_SIM <- "SIM_2020.csv"
ARQ_CBO <- "CBO_2002.csv"
ARQ_MUN <- "municipios.csv"
ARQ_CID <- "CID10.csv"

# ============================================================
# 1. IMPORTAÇÃO
# ============================================================

sim <- read_csv(
  ARQ_SIM,
  locale = locale(encoding = "Latin1"),
  na = c("", "NA", "NULL")
) %>%
  clean_names()

cbo <- read_csv(
  ARQ_CBO,
  locale = locale(encoding = "Latin1")
) %>%
  clean_names()

municipios <- read_csv(
  ARQ_MUN,
  locale = locale(encoding = "UTF-8")
) %>%
  clean_names()

cid <- read_csv(
  ARQ_CID,
  locale = locale(encoding = "UTF-8")
) %>%
  clean_names()


# ============================================================
# 2. TRATAMENTO DAS VARIÁVEIS
# ============================================================

sim <- sim %>%
  mutate(

    # DATA
    DATA_OBITO = dmy(dtobito),

    # SEXO
    SEXO_DESC = case_when(
      sexo %in% c("M", "1") ~ "Masculino",
      sexo %in% c("F", "2") ~ "Feminino",
      sexo %in% c("I", "0", "9") ~ "Ignorado",
      TRUE ~ NA_character_
    ),

    # RAÇA/COR
    RACA_DESC = case_when(
      racacor == "1" ~ "Branca",
      racacor == "2" ~ "Preta",
      racacor == "3" ~ "Amarela",
      racacor == "4" ~ "Parda",
      racacor == "5" ~ "Indígena",
      TRUE ~ "Ignorado"
    ),

    # ESCOLARIDADE
    ESC_DESC = case_when(
      esc == "1" ~ "Nenhuma",
      esc == "2" ~ "1 a 3 anos",
      esc == "3" ~ "4 a 7 anos",
      esc == "4" ~ "8 a 11 anos",
      esc == "5" ~ "12 anos ou mais",
      esc == "9" ~ "Ignorado",
      TRUE ~ NA_character_
    ),

    # LOCAL DE OCORRÊNCIA
    LOCAL_MORTE = case_when(
      lococor == "1" ~ "Hospital",
      lococor == "2" ~ "Outros estabelecimentos de saúde",
      lococor == "3" ~ "Domicílio",
      lococor == "4" ~ "Via pública",
      lococor == "5" ~ "Outros",
      lococor == "6" ~ "Aldeia indígena",
      lococor == "9" ~ "Ignorado",
      TRUE ~ NA_character_
    ),

    # IDADE
    idade_cod = as.character(idade),

    idade_unidade = str_sub(idade_cod, 1, 1),

    idade_valor = suppressWarnings(
      as.numeric(str_sub(idade_cod, 2, 3))
    ),

    IDADE_ANOS = case_when(
      idade_unidade == "1" ~ idade_valor / (365.25 * 24 * 60),
      idade_unidade == "2" ~ idade_valor / (365.25 * 24),
      idade_unidade == "3" ~ idade_valor / 12,
      idade_unidade == "4" ~ idade_valor,
      idade_unidade == "5" ~ 100,
      TRUE ~ NA_real_
    )
  )


# ============================================================
# 3. JOIN MUNICÍPIOS
# ============================================================

sim <- sim %>%
  left_join(
    municipios,
    by = c("codmunres" = "codigo_municipio")
  )


# ============================================================
# 4. JOIN CBO
# ============================================================

sim <- sim %>%
  left_join(
    cbo,
    by = c("ocup" = "codigo_cbo")
  )


# ============================================================
# 5. JOIN CID
# ============================================================

sim <- sim %>%
  left_join(
    cid,
    by = c("causabas" = "codigo_cid")
  )


# ============================================================
# 6. DATASET FINAL
# ============================================================

dados_final <- sim %>%
  transmute(
    ANO = as.integer(ano),
    DATA_DE_OBITO = DATA_OBITO,
    IDADE = IDADE_ANOS,
    SEXO = SEXO_DESC,
    RACA = RACA_DESC,
    ESCOLARIDADE = ESC_DESC,
    MUNICIPIO = nome_municipio,
    LOCAL_MORTE = LOCAL_MORTE,
    OCUP = descricao_cbo,
    BASICA_DA_MORTE = descricao_causa
  )


# ============================================================
# 7. EXPORTAÇÃO
# ============================================================

write_csv(
  dados_final,
  "SIM_tratado.csv",
  na = ""
)
```
