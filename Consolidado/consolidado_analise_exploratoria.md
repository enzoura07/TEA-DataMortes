# Consolidado — Análise Exploratória de Dados (EDA)

## 1. Objetivo

A Análise Exploratória de Dados (EDA) constitui a etapa inicial do projeto do **SIM (Sistema de Informação sobre Mortalidade)**. Seu objetivo é compreender a estrutura dos dados, identificar padrões demográficos e epidemiológicos e fornecer uma visão geral dos óbitos registrados na **Baixada Santista**.

A análise concentra-se nos municípios de:

- Santos
- São Vicente
- Guarujá
- Praia Grande
- Cubatão

## 2. Corpus e preparação

Foi utilizado o arquivo `DatasetSIM_tratado.csv`, contendo registros de óbitos tratados. Antes da análise, foram realizadas etapas de preparação dos dados, incluindo:

- leitura da base;
- seleção dos municípios da Baixada Santista;
- conversão da variável de idade para formato numérico;
- tratamento de valores ausentes;
- organização das variáveis categóricas.

A variável `IDADE` foi utilizada em anos para possibilitar a análise estatística e visual da distribuição etária.

## 3. Análise da idade

Foram calculadas estatísticas descritivas da idade no momento do óbito, incluindo:

- mínimo;
- primeiro quartil;
- mediana;
- média;
- terceiro quartil;
- máximo;
- desvio padrão.

Também foram utilizados **histograma** e **boxplot** para visualizar a distribuição da idade, sua concentração e possíveis valores extremos.

## 4. Relação entre idade e escolaridade

Foi utilizado um boxplot para comparar a distribuição da idade no momento do óbito entre diferentes níveis de escolaridade.

Essa análise permite observar diferenças na distribuição etária entre os grupos de escolaridade e fornece uma visão inicial sobre uma possível associação entre escolaridade e idade no óbito.

## 5. Distribuição por sexo

Foi analisada a frequência de registros de óbito segundo o sexo, permitindo identificar a composição da base entre indivíduos classificados como:

- Masculino;
- Feminino.

Essa distribuição também serve de base para as etapas posteriores de regressão logística.

## 6. Principais causas de morte

Foi realizada a contagem das ocorrências da variável `CAUSABASMORTE` e ordenação decrescente das frequências para identificar as **cinco principais causas básicas de morte** presentes na base analisada.

## 7. Conclusão

A análise exploratória fornece uma visão inicial do perfil dos óbitos na Baixada Santista e serve como fundamento para as etapas de modelagem estatística.

A partir dela, é possível compreender a distribuição da idade, observar diferenças relacionadas à escolaridade, verificar a composição por sexo e identificar as principais causas básicas de morte. Essas informações orientam a escolha das variáveis utilizadas nos modelos de regressão linear e logística.
