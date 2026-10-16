# Consolidado — Métodos de Suavização

## 1. Visão geral

Este material documenta o notebook `metodosSuavização.ipynb`, que investiga a relação entre **escolaridade** e **idade ao óbito** por meio de modelos de regressão e métodos de suavização. O objetivo computacional é comparar modelos com diferentes níveis de flexibilidade e avaliar se uma relação linear simples descreve adequadamente os dados ou se um método não paramétrico consegue capturar padrões adicionais.


## 2. Pergunta analítica e variáveis

### 2.1 Pergunta central

A análise procura avaliar como a idade ao óbito varia conforme a escolaridade, e se essa relação é melhor representada por uma reta ou por curvas mais flexíveis obtidas por suavização.

A pergunta pode ser expressa de forma operacional como:

> Qual método prevê melhor a idade ao óbito a partir da escolaridade numérica: uma previsão constante pela média, uma regressão linear, o K-vizinhos mais próximos (KNN) ou a suavização por kernel gaussiano?

Essa formulação delimita a comparação feita no notebook. Embora o conjunto de dados possua outras variáveis, os modelos comparados nesta etapa usam **ESCOLARIDADE_NUM como único preditor**.

### 2.2 Variável resposta

- **`IDADE`**: idade registrada ao óbito, convertida para tipo numérico.
- O código mantém apenas valores não ausentes, finitos e maiores ou iguais a zero.

A variável resposta é quantitativa. O objetivo dos modelos é produzir uma estimativa numérica da idade, e não classificar o registro em categorias.

### 2.3 Variável explicativa principal

O notebook transforma a variável categórica `ESCOLARIDADE` em `ESCOLARIDADE_NUM`, por meio do seguinte mapeamento:

| Categoria original | Valor numérico atribuído |
|---|---:|
| Sem escolaridade | 0 |
| 1 a 3 anos | 2 |
| 4 a 7 anos | 5,5 |
| 8 a 11 anos | 9,5 |
| 12 anos ou mais | 14 |
| Qualquer outra categoria | 6 |

Os valores representam aproximações numéricas dos níveis de escolaridade. Para as faixas intermediárias, foram usados valores representativos; para “12 anos ou mais”, o código utiliza 14. Categorias não reconhecidas recebem 6.

**Limitação importante:** o valor 6 para categorias não reconhecidas não equivale necessariamente a uma observação com seis anos de estudo. Antes de interpretar os resultados substantivamente, convém conferir quais categorias caem no caso padrão (`TRUE ~ 6`) e quantos registros são afetados. Essa observação não altera o que o código executa, mas é essencial para interpretar o preditor.

### 2.4 Outras variáveis

Na preparação, o notebook também seleciona `SEXO`, `LOCAL_MORTE`, `ANO`, `MUNICIPIO` e `RAÇA`. Elas são mantidas no conjunto preparado, mas **não entram diretamente nos quatro modelos comparados nesta análise**, que usam somente `ESCOLARIDADE_NUM` para prever `IDADE`.

## 3. Bibliotecas e ambiente

O notebook instala e/ou carrega pacotes de R:

- **`readr`**: ferramentas de leitura de dados.
- **`dplyr`**: seleção, transformação e filtragem de dados.
- **`FNN`**: cálculo de vizinhos mais próximos, usado no KNN.
- **`readxl`**: leitura de arquivos Excel (`.xlsx`).
- **`ggplot2`**: elaboração de gráficos.

A primeira célula de código executa `install.packages(...)`. Em ambientes nos quais os pacotes já estão instalados, essa etapa pode ser dispensável. O arquivo de entrada deve estar acessível no diretório de trabalho com o nome `dataset_final.xlsx`.

## 4. Preparação e limpeza dos dados

### 4.1 Leitura e dimensão inicial

O comando `read_excel("dataset_final.xlsx")` importa a planilha para `dados_brutos`. O notebook registra a quantidade inicial de linhas para documentar o tamanho do conjunto antes da limpeza.

### 4.2 Identificação e remoção de duplicatas

A duplicidade é identificada considerando conjuntamente três campos:

- `DATA DE ÓBITO`;
- `IDADE`;
- `CAUSA BASICA DA MORTE`.

O código conta quantas linhas repetem essa combinação e conserva apenas a primeira ocorrência de cada combinação. A variável `OCUP` é explicitamente deixada de fora do critério, conforme o comentário do notebook.

Essa decisão reduz repetições com a mesma data, idade e causa básica, mas exige cuidado: dois registros diferentes podem compartilhar esses três valores. Portanto, a remoção não prova por si só que todos os registros excluídos eram duplicatas reais. O critério deve ser justificado em função da estrutura e das regras do banco de origem.

### 4.3 Seleção das colunas

Depois da remoção de duplicatas, o código mantém:

- resposta: `IDADE`;
- preditores originalmente definidos: `ESCOLARIDADE`, `SEXO`, `LOCAL_MORTE`, `ANO`, `MUNICIPIO` e `RAÇA`.

A seleção explícita reduz o conjunto às variáveis necessárias para a análise planejada.

### 4.4 Conversão de tipos e filtros

O código converte `IDADE` para numérico e as demais variáveis selecionadas para texto. Em seguida, remove linhas em que a idade é ausente (`NA`), não é finita ou é negativa.

Essas verificações evitam que valores impossíveis para o cálculo numérico contaminem o ajuste dos modelos. O filtro não estabelece que todas as idades restantes estejam corretas; ele apenas aplica os critérios de validade implementados.

### 4.5 Tratamento de valores ausentes nos preditores categóricos

Para cada preditor, os valores ausentes são substituídos pelo texto `"Ignorado"`. Isso evita que esses registros sejam descartados automaticamente por ausência nessas colunas.

Na sequência, a escolaridade é convertida para a escala numérica descrita anteriormente. A quantidade de observações após o tratamento é exibida por `cat(...)`, permitindo comparar a dimensão final com a inicial.

## 5. Separação entre treino e teste

O notebook fixa a semente aleatória em `123` e seleciona, aleatoriamente, 80% das observações para treino. Os 20% restantes formam o conjunto de teste.

- **Treino:** utilizado para ajustar os modelos e selecionar parâmetros.
- **Teste:** reservado para avaliar o desempenho final em registros que não participaram do ajuste nem da escolha dos parâmetros.

A semente torna a amostragem reproduzível sob as mesmas condições de execução e versões compatíveis das ferramentas.

### 5.1 Subamostra de treinamento

Para tornar a validação cruzada computacionalmente mais viável, o código extrai uma subamostra de até **15.000 registros** do conjunto de treino, novamente com semente `123`. Essa subamostra, chamada `treino_sub`, é usada na validação cruzada e como base dos modelos comparados.

Assim, há três conjuntos conceitualmente distintos:

1. `treino`: os 80% inicialmente destinados ao treinamento;
2. `treino_sub`: até 15.000 registros selecionados dentro de `treino`;
3. `teste`: os 20% separados para avaliação final.

A amostragem reduz custo computacional, especialmente para métodos que comparam cada ponto com observações de treino. Entretanto, a seleção aleatória pode afetar a representação de categorias raras; os resultados devem ser interpretados considerando essa redução.

## 6. Métrica de avaliação: MSE e RMSE

### 6.1 Erro quadrático médio (MSE)

O erro quadrático médio é definido por:

\[
\operatorname{MSE} = \frac{1}{n}\sum_{i=1}^{n}(y_i-\hat{y}_i)^2
\]

em que:

- \(y_i\) é a idade observada;
- \(\hat{y}_i\) é a idade prevista;
- \(n\) é o número de observações avaliadas.

O MSE penaliza mais fortemente os erros grandes porque as diferenças são elevadas ao quadrado. Quanto menor o MSE, melhor o ajuste preditivo segundo essa métrica.

### 6.2 Raiz do erro quadrático médio (RMSE)

O RMSE é a raiz quadrada do MSE:

\[
\operatorname{RMSE} = \sqrt{\operatorname{MSE}}
\]

Como a resposta está em anos, o RMSE também é expresso em anos. Isso facilita interpretar a magnitude típica dos erros de previsão. Por exemplo, um RMSE hipotético de 10 significaria um erro quadrático médio equivalente a uma escala de aproximadamente 10 anos; não significa que todos os erros individuais sejam de 10 anos.

O notebook define uma função `rmse(real, previsto)` que calcula essa métrica ignorando pares que gerem valores ausentes na operação.

### 6.3 MSE de validação cruzada e RMSE de teste

O código apresenta tanto o MSE médio obtido na validação cruzada quanto o RMSE no conjunto de teste. São avaliações relacionadas, mas não idênticas:

- a validação cruzada orienta a seleção dos parâmetros;
- o teste fornece uma avaliação final em observações mantidas à parte.

Para comparar os modelos, valores menores são preferíveis, desde que os conjuntos e procedimentos de avaliação sejam comparáveis.

## 7. Modelos comparados

### 7.1 Modelo de referência: média do treino

O modelo de referência prevê a mesma idade para todas as observações: a média de `IDADE` no conjunto de treinamento usado na validação cruzada.

Em cada dobra, a média é calculada apenas com os dados da parte de ajuste daquela dobra. No teste final, a previsão constante é a média de `y_cv`, isto é, das idades em `treino_sub`.

**Função:** estabelecer uma linha de base simples. Um modelo mais complexo deve demonstrar melhora em relação a essa referência para justificar sua complexidade.

### 7.2 Regressão linear simples

A regressão linear é ajustada por:

```r
lm(IDADE ~ ESCOLARIDADE_NUM, data = treino_sub)
```

Sua forma matemática é:

\[
\widehat{\operatorname{IDADE}} = \hat{\beta}_0 + \hat{\beta}_1\,\operatorname{ESCOLARIDADE\_NUM}
\]

- \(\hat{\beta}_0\): intercepto estimado;
- \(\hat{\beta}_1\): variação média estimada na idade ao óbito associada a uma unidade adicional na escala numérica de escolaridade.

O modelo impõe uma relação linear: cada aumento de uma unidade no preditor corresponde à mesma mudança prevista na resposta, independentemente do nível inicial.

**Vantagens:** é simples, rápido, interpretável e serve como referência paramétrica.

**Limitação:** pode não capturar curvaturas, platôs ou diferenças entre faixas de escolaridade. Além disso, como a escolaridade numérica representa categorias ordenadas aproximadas, uma unidade de diferença na escala não deve ser interpretada automaticamente como um ano real de estudo em todos os intervalos.

### 7.3 K-vizinhos mais próximos (KNN) para regressão

O KNN estima a resposta de um ponto usando as respostas das \(k\) observações de treino mais próximas em termos do preditor. Neste notebook, há apenas uma variável explicativa, `ESCOLARIDADE_NUM`.

Para cada valor \(x\), o procedimento identifica os \(k\) vizinhos e calcula a média das idades correspondentes:

\[
\hat{m}(x) = \frac{1}{k}\sum_{j\in N_k(x)} y_j
\]

em que \(N_k(x)\) é o conjunto dos \(k\) vizinhos mais próximos de \(x\).

O código utiliza `FNN::get.knnx(...)` com algoritmo `"brute"` e testa os valores:

`k = 5, 10, 20, 50, 100, 200, 400, 800, 1500`.

Como a variável explicativa tem valores repetidos e poucos níveis numéricos possíveis, o código calcula a previsão uma vez para cada valor distinto de `x_new` e depois a replica para as observações com o mesmo valor. Isso evita repetir cálculos idênticos.

**Efeito do parâmetro \(k\):**

- \(k\) pequeno: modelo mais local e flexível, mas potencialmente mais sensível à variação amostral;
- \(k\) grande: previsão mais suavizada, com maior influência de observações distantes e possível perda de padrões locais.

O parâmetro é selecionado por validação cruzada, em vez de ser escolhido arbitrariamente.

### 7.4 Suavização por kernel gaussiano: estimador de Nadaraya–Watson

O estimador de Nadaraya–Watson prevê a resposta por uma média ponderada das observações de treino. Pontos mais próximos do valor de interesse recebem pesos maiores.

A fórmula é:

\[
\hat{m}(x) =
\frac{\sum_{i=1}^{n} K\left(\frac{x_i-x}{h}\right)y_i}
{\sum_{i=1}^{n} K\left(\frac{x_i-x}{h}\right)}
\]

O notebook utiliza o kernel gaussiano, com pesos calculados por `dnorm((x_train - x0) / h)`.

- \(x\): valor de escolaridade no qual se deseja prever a idade;
- \(x_i\): escolaridade da observação de treino;
- \(y_i\): idade observada nessa observação;
- \(K\): função kernel gaussiana;
- \(h\): largura de banda (*bandwidth*), que controla o grau de suavização.

**Interpretação de \(h\):**

- \(h\) pequeno: os pesos diminuem rapidamente com a distância, gerando uma curva mais local e potencialmente mais irregular;
- \(h\) grande: observações mais distantes mantêm influência, produzindo uma curva mais suave.

A grade avaliada é:

`h = 0.05, 0.1, 0.2, 0.5, 1, 1.5, 2, 3, 4, 6`.

O código calcula a previsão para cada valor distinto do preditor e replica os resultados. Se a soma dos pesos não for finita ou for igual a zero, a previsão é marcada como `NA`.

**Limitação de interpretação:** a suavização é feita sobre a escala numérica criada para escolaridade, e não sobre anos de estudo observados individualmente. Portanto, a curva descreve a relação entre idade e essa codificação ordinal.

## 8. Validação cruzada com cinco dobras

O código utiliza `K_folds` e `folds` para realizar validação cruzada. O trecho apresentado pressupõe que essas variáveis estejam definidas antes de executar a célula de validação; no notebook fornecido, não há uma definição explícita delas nas células anteriores. Para executar o fluxo integralmente, é necessário definir cinco dobras e atribuir cada observação a uma delas, por exemplo, garantindo que cada registro pertença a exatamente uma dobra.

Na validação cruzada de cinco dobras:

1. divide-se a subamostra de treino em cinco grupos;
2. em cada rodada, quatro grupos são usados para ajustar o modelo e um grupo para validação;
3. o processo se repete até que cada grupo tenha sido usado como validação;
4. calcula-se a média do erro entre as cinco rodadas.

O código compara os métodos usando as mesmas dobras, o que torna a comparação mais justa: as diferenças observadas não se devem simplesmente a cada modelo ter recebido partições distintas.

### 8.1 Função `cv_metodo`

Essa função avalia cada valor de parâmetro fornecido. Para cada parâmetro e dobra, ajusta/prepara a previsão, calcula o MSE de validação e registra quantas previsões resultaram em `NA`.

Se uma previsão estiver ausente, o código a substitui pela média das idades da parte de ajuste daquela dobra. Isso permite concluir o cálculo do MSE, mas a quantidade de previsões substituídas deve ser monitorada, pois muitas substituições indicariam um problema na cobertura do estimador.

A tabela retornada contém:

- `par`: parâmetro avaliado;
- `mse`: MSE médio entre as dobras;
- `ep`: erro-padrão do MSE médio;
- `nan`: total de previsões ausentes contabilizadas.

O erro-padrão é calculado como o desvio-padrão dos MSEs das dobras dividido pela raiz quadrada do número de dobras. Ele expressa a incerteza da média dos erros entre as partições, não a dispersão dos erros individuais de previsão.

### 8.2 Regra de um erro-padrão (1 EP)

A função `escolher_1ep` implementa a regra de um erro-padrão:

1. encontra o parâmetro com menor MSE médio;
2. calcula um limite igual ao menor MSE mais o erro-padrão associado;
3. identifica os parâmetros cujo MSE está abaixo desse limite;
4. entre eles, escolhe o maior parâmetro.

Para KNN, isso tende a selecionar um \(k\) maior, que representa um modelo mais suavizado. Para o kernel, tende a selecionar um \(h\) maior, também mais suavizado. A escolha favorece uma solução mais simples/estável quando seu desempenho está suficientemente próximo do melhor valor observado.

A regra não significa que o maior parâmetro sempre seja melhor. Ele só é escolhido se seu erro estiver dentro do limite definido pela regra.

### 8.3 Modelos de referência na validação

Além de KNN e kernel, o notebook calcula a validação cruzada para:

- `cv_media`: previsão pela média da parte de ajuste;
- `cv_reta`: regressão linear ajustada em cada dobra.

Essas referências ajudam a avaliar se a flexibilidade dos métodos não paramétricos realmente reduz o erro.

## 9. Avaliação no conjunto de teste

Depois de selecionar `k_escolhido` e `h_escolhido`, o código gera previsões no conjunto `teste` para os quatro modelos:

1. média do treino;
2. regressão linear;
3. KNN com o \(k\) selecionado;
4. kernel gaussiano com o \(h\) selecionado.

Para o kernel, o código conta as previsões `NA` e substitui esses casos pela média de `y_cv`. A tabela final inclui:

- **Modelo:** método avaliado;
- **Parâmetro:** `k` ou `h`, quando aplicável;
- **MSE_CV:** erro quadrático médio obtido na validação cruzada;
- **RMSE_CV:** raiz do MSE de validação cruzada;
- **RMSE_teste:** raiz do erro quadrático médio no conjunto de teste.

A tabela é ordenada pelo `RMSE_teste`, do menor para o maior. Assim, o primeiro modelo listado é o de menor RMSE no teste entre os modelos avaliados.

**Atenção à leitura:** não é possível afirmar qual modelo venceu sem executar o notebook e observar a tabela impressa. O código define a comparação, mas não fornece, no conteúdo analisado, os números finais.

## 10. Visualizações e interpretação

O notebook produz gráficos para examinar a escolha de parâmetros e a forma da relação entre escolaridade e idade ao óbito.

### 10.1 Gráficos de seleção de parâmetros

São apresentados dois gráficos:

- MSE de validação cruzada em função de \(k\), para KNN;
- MSE de validação cruzada em função de \(h\), para o kernel.

Os eixos dos parâmetros são logarítmicos, permitindo visualizar uma grade que varia por ordens de grandeza. A linha horizontal vermelha representa o limite de um erro-padrão acima do menor MSE, e o marcador destacado identifica o parâmetro selecionado pela regra de 1 EP.

Esses gráficos ajudam a verificar se o erro é muito sensível ao parâmetro e se a seleção ocorreu em uma extremidade da grade. O código imprime avisos quando o valor escolhido ou o valor de menor erro está em determinadas bordas da grade. Nesses casos, ampliar a grade pode ser útil, pois o melhor parâmetro pode estar fora dos valores testados.

### 10.2 Gráfico da relação entre escolaridade e idade

O gráfico final apresenta:

- **pontos cinza:** observações individuais de uma amostra de até 3.000 registros;
- **linha azul:** estimativa suavizada pelo kernel gaussiano;
- **linha vermelha tracejada:** regressão linear;
- **pontos laranja:** idade média observada em cada nível de `ESCOLARIDADE_NUM`.

A dispersão dos pontos mostra a variabilidade da idade dentro de cada nível. A curva do kernel permite verificar se a tendência parece mudar de forma ao longo da escala, enquanto a reta impõe uma tendência linear. As médias por nível ajudam a comparar os padrões agregados com os modelos.

O gráfico é exploratório: ele auxilia a compreender a forma da relação, mas não substitui a avaliação preditiva no conjunto de teste nem demonstra causalidade.

### 10.3 Tabela de médias por nível e diferenças em relação à reta

O código cria uma tabela com:

- nível numérico de escolaridade;
- número de observações (`n`);
- idade média observada (`media_obs`);
- idade prevista pela reta (`reta`);
- diferença entre a média observada e a previsão linear (`diferenca`).

Uma diferença positiva indica que a média observada naquele nível está acima da previsão da reta; uma diferença negativa indica que está abaixo. Diferenças sistemáticas ao longo dos níveis podem sugerir que uma reta não descreve perfeitamente o padrão médio, embora sua importância deva ser avaliada em conjunto com os erros preditivos.

## 11. Critério final: reta ou suavizador?

A última parte do código compara o menor MSE entre KNN e kernel com o MSE da regressão linear, levando em conta o erro-padrão associado ao melhor suavizador.

A lógica implementada é:

- se o MSE da reta for menor ou igual ao MSE mínimo do suavizador mais seu erro-padrão, o código recomenda ficar com a reta;
- caso contrário, informa que o suavizador supera a reta por mais de um erro-padrão.

Essa regra procura evitar a adoção de um modelo mais flexível quando a melhora observada é pequena em relação à incerteza da comparação. A decisão é orientada pela validação cruzada e deve ser lida em conjunto com o RMSE de teste, a estabilidade dos parâmetros e a interpretabilidade.

A mensagem produzida pelo código é uma regra operacional para esta análise, não uma prova definitiva de que a relação real seja linear ou não linear.

## 12. Pontos técnicos a verificar antes de executar

### 12.1 Definição das dobras

No trecho fornecido, `K_folds` e `folds` são utilizados, mas não são definidos nas células anteriores. É necessário criá-los antes de executar a validação cruzada. Como o texto do código informa “CV(5)”, a intenção é utilizar cinco dobras.

Exemplo de definição compatível com essa intenção:

```r
K_folds <- 5
set.seed(123)
folds <- sample(rep(seq_len(K_folds), length.out = nrow(treino_sub)))
```

Essa definição deve ser executada depois da criação de `treino_sub` e antes da célula de validação cruzada. Ela atribui cada observação a uma das cinco dobras, com tamanhos tão equilibrados quanto possível.

### 12.2 Função de kernel definida, mas não utilizada no fluxo principal

A célula 5 define `kernel_univariado_pred`, enquanto a validação e a avaliação final utilizam `nw_pred`. Ambas implementam uma suavização por kernel gaussiano univariado, mas a função efetivamente conectada ao processo de validação é `nw_pred`. Para evitar confusão, convém documentar que `kernel_univariado_pred` é uma definição auxiliar não usada no fluxo principal mostrado.

### 12.3 Tratamento de valores ausentes e pesos

Na validação, `cv_metodo` substitui previsões ausentes pela média da parte de ajuste. Já `nw_pred` calcula a soma dos pesos e a soma ponderada usando os valores de treino. O código pressupõe que os vetores de treino estejam preparados adequadamente. Se existirem valores ausentes em `ESCOLARIDADE_NUM` ou `IDADE` que escapem à limpeza, isso pode afetar os pesos e as previsões. Recomenda-se conferir essas colunas antes de ajustar os modelos.

### 12.4 Separação treino/teste e subamostra

A subamostra usada para seleção de parâmetros é extraída somente do treino, o que preserva o conjunto de teste para avaliação final. Esse é um ponto metodológico positivo. É importante não utilizar o resultado do teste para voltar a escolher `k` ou `h`, pois isso faria o teste deixar de ser uma avaliação independente.

### 12.5 Critério de duplicidade

A deduplicação por data de óbito, idade e causa básica pode remover registros legítimos que coincidam nesses três campos. A definição de duplicidade deve, idealmente, ser conferida com a documentação do banco e com os identificadores disponíveis. O notebook informa o critério usado, mas não apresenta evidência suficiente para garantir que ele corresponda à definição correta de duplicata.

### 12.6 Escala ordinal da escolaridade

A codificação numérica transforma faixas categóricas em uma escala ordenada. Regressão linear, KNN e kernel passam a calcular distâncias ou tendências nessa escala. Os resultados dependem, portanto, das escolhas numéricas feitas para representar as categorias. Uma análise de sensibilidade com codificações alternativas pode avaliar se as conclusões são robustas, mas essa análise adicional não está implementada no notebook fornecido.

### 12.7 Variáveis disponíveis versus variáveis usadas

Apesar de preparar várias variáveis demográficas e contextuais, o modelo comparativo usa somente escolaridade. Não se deve afirmar que a análise controla por sexo, município, raça, local de morte ou ano. Para incluir essas variáveis, seria necessário especificar e avaliar outros modelos, o que está fora do fluxo de comparação apresentado.

## 13. Como executar e reproduzir a análise

1. Coloque `dataset_final.xlsx` no diretório de trabalho do R ou do ambiente compatível.
2. Instale e carregue os pacotes necessários.
3. Execute a leitura, a deduplicação e a limpeza dos dados.
4. Gere `ESCOLARIDADE_NUM` e confira a distribuição dos níveis e das categorias que receberam o valor padrão 6.
5. Faça a separação treino/teste e gere `treino_sub`.
6. Defina `K_folds <- 5` e crie `folds` com uma atribuição por observação de `treino_sub`.
7. Defina a função RMSE e as funções de previsão.
8. Execute a validação cruzada para a média, a reta, o KNN e o kernel.
9. Examine as tabelas de MSE, erro-padrão e contagem de previsões ausentes.
10. Verifique os parâmetros escolhidos e os avisos de borda da grade.
11. Avalie os modelos no conjunto de teste e compare `RMSE_teste`.
12. Gere os gráficos e interprete a tendência junto dos resultados numéricos.

Para reprodutibilidade, mantenha registradas as sementes aleatórias, a proporção treino/teste, o tamanho da subamostra, a definição das dobras e as grades de parâmetros utilizadas.

## 14. Síntese comparativa

| Método | Ideia central | Parâmetro principal | Principal vantagem | Principal limitação |
|---|---|---|---|---|
| Média do treino | Prevê uma constante igual à média das idades | Nenhum | Referência simples | Não representa associação com escolaridade |
| Regressão linear | Ajusta uma reta entre escolaridade e idade | Coeficientes estimados | Interpretação direta e simplicidade | Não captura curvatura |
| KNN | Usa a média das idades dos vizinhos mais próximos | \(k\) | Flexibilidade local | Sensível à escolha de \(k\) e à distribuição dos níveis |
| Kernel gaussiano | Faz média ponderada por proximidade | \(h\) | Curva suave e flexível | Sensível à largura de banda e à escala do preditor |

## 15. Conclusão

O notebook constrói um fluxo de comparação entre um modelo de referência, uma regressão linear e dois métodos de suavização não paramétrica. A limpeza dos dados e a separação treino/teste estabelecem a base da análise; a validação cruzada de cinco dobras é planejada para escolher os parâmetros de KNN e kernel; e o RMSE de teste permite comparar a capacidade preditiva dos modelos em observações reservadas.

A principal questão metodológica é verificar se a flexibilidade dos suavizadores reduz o erro de maneira relevante em comparação com a reta. A regra de um erro-padrão favorece parâmetros mais suavizados e pode favorecer a regressão linear quando a diferença de desempenho é pequena. Os gráficos complementam a avaliação numérica ao revelar a forma da relação observada.

Antes de interpretar resultados substantivos, devem ser verificados os seguintes pontos: a definição de `K_folds` e `folds`, a validade do critério de remoção de duplicatas, as categorias de escolaridade codificadas como 6 e a ausência de valores inválidos nas colunas usadas. Por fim, como os números de saída não acompanham o código analisado, a escolha do melhor modelo e os valores efetivos de erro só podem ser informados após executar o notebook.

**Escopo da conclusão:** esta análise avalia associação e desempenho preditivo entre escolaridade codificada e idade ao óbito. Por si só, ela não permite concluir que escolaridade cause maior ou menor idade ao óbito, nem que as diferenças observadas estejam isoladas de outras variáveis que não entram nos modelos comparados.
