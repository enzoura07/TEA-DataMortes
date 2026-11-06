# Consolidado único — Aula 05b e Parte D
## Teoria do Aprendizado Estatístico · Regressão de Poisson

**Data:** 6 de novembro de 2026  
**Guia:** versão 1  
**Tutora:** ChatGPT  
**Aluno:** [preencher]  
**Grupo e integrantes:** [preencher]  
**Responsável pela digitação em cada módulo:** [preencher]

**Síntese da atividade:** este consolidado integra os nove módulos da Aula 05b e da Parte D, os exercícios práticos em R, os diagnósticos de dispersão, as validações cruzadas e um conjunto completo de respostas comentadas aos checkpoints e testes dos guias. Os resultados numéricos correspondem às saídas do notebook; as respostas conceituais foram organizadas a partir dos enunciados dos guias.

## 1. O que foi passado — teoria, Módulos 1 a 6

A preparação das bases simuladas foi executada e retornou os totais esperados: $836$, $775$ e $550$. Os resultados abaixo são os registrados nas células 1 a 6 do notebook, não uma nova execução. [3]

| Módulo | Atividade e resultado |
|---|---|
| **M1 — Contagens: por que a reta não serve** | O exemplo dos oito anúncios foi executado. O ajuste por mínimos quadrados ordinários (MQO), a regressão linear do `lm()`, previu aproximadamente $-0{,}73$ e $-1{,}66$ visitas. |
| **M2 — O modelo: o log da média é uma reta** | O guia apresenta $\log\lambda$ como combinação linear dos preditores e $e^{\beta}$ como multiplicador da contagem esperada. O conteúdo é acompanhado de resolução conceitual no Anexo A. [1] |
| **M3 — Ajustar e ler no R** | Foram executados `glm(family = poisson)` e `predict(type = "response")`. Os coeficientes foram $1{,}5738$, $-0{,}5217$ e $0{,}0653$. Para o anúncio a $1$ km com $12$ fotos, a previsão foi $6{,}2665$ visitas em $30$ dias. |
| **M4 — Tempo de exposição: o offset** | Foram comparados os ajustes com e sem exposição. O multiplicador da distância passou de $0{,}832$ para $0{,}621$. A previsão com offset foi $3{,}8495$ visitas em $20$ dias. |
| **M5 — Sobredispersão** | Foi obtido $\hat\phi \approx 3{,}3718$. Poisson e quase-Poisson mantiveram os mesmos coeficientes; o erro padrão de distância passou de $0{,}0303$ para $0{,}0556$. |
| **M6 — Fora da amostra: Poisson contra MQO** | Os modelos foram comparados nas mesmas cinco dobras de validação cruzada. O Poisson teve o menor erro. |

### Placar da teoria

A raiz do erro quadrático médio (RMSE) mede o erro de previsão na unidade da resposta, aqui visitas. [1]

| Modelo | RMSE — visitas |
|---|---:|
| Linha de base: média do treino | $3{,}9901$ |
| MQO | $2{,}3870$ |
| **Poisson** | **$1{,}7579$** |

**Conclusão da comparação:** neste exercício, o Poisson apresentou o menor erro fora da amostra. [3, célula 6]

## 2. O que foi feito — Parte D, Módulos 7 a 9

### M7 — Achar a contagem no banco do grupo

Foi importado `dataset_final.xlsx`, com $474\,651$ linhas registradas na saída do notebook. O procedimento exploratório manteve uma linha por bloco consecutivo de registros com os mesmos campos, desconsiderando a ocupação, e produziu $86\,683$ registros aproximados. A consolidação é uma aproximação exploratória, pois os registros originais requerem identificação única para eliminar repetições com segurança. [3, célula 8; 4]

A resposta utilizada no código, denominada `obitos`, é a contagem desses registros por **município, ano e mês**. A tabela resultante tem $540$ linhas. O código incluiu as combinações entre municípios, anos e os doze meses; a saída registrou nenhuma combinação com contagem zero. A exposição utilizada foi o número de dias do mês. [3, célula 8]

**Pergunta de análise da Parte D:** como a contagem mensal de registros varia conforme município, ano e mês, considerando a duração do mês? Essa pergunta organiza a aplicação exploratória e seus resultados.

### M8 — Ajustar, ler e conferir

Foram utilizados município, ano e mês como preditores, com offset $\log(\text{dias do mês})$. O diagnóstico apresentou $\hat\phi \approx 4{,}209$, e foi executado o ajuste quase-Poisson para utilizar os erros padrão corrigidos. [3, célula 8]

Como interpretação do resultado, maio apresentou multiplicador estimado de $1{,}061$ em relação a janeiro: aproximadamente $6{,}1\%$ a mais na taxa diária esperada de registros, mantendo município e ano fixos. Trata-se de uma estimativa na base exploratória, não de uma conclusão sobre risco individual de morrer. [3, célula 8]

### M9 — Validar e decidir

A comparação utilizou as mesmas cinco dobras para os três modelos. A linha de base foi a taxa diária média do treino multiplicada pelos dias do mês de validação; o MQO incluiu os dias do mês como preditor, e o Poisson usou o offset. [3, célula 8]

| Modelo | RMSE — registros por município-mês |
|---|---:|
| Linha de base: taxa diária ajustada aos dias | $180{,}824$ |
| MQO | $45{,}793$ |
| **Poisson** | **$41{,}696$** |

**Conclusão técnica:** o Poisson apresentou o menor erro de previsão na comparação exploratória, sendo a escolha respaldada pelo placar obtido. [3, célula 8]

## 3. Verificações computacionais documentadas

| Verificação | Registro de execução |
|---|---|
| Preparação das bases simuladas | Totais $836$, $775$ e $550$. |
| Ajuste linear e inspeção das previsões | Previsões negativas no exemplo dos oito anúncios. |
| Ajuste Poisson e previsão na escala da resposta | Coeficientes e previsão de $6{,}2665$ visitas. |
| Comparação com e sem offset | Multiplicadores e previsão de $3{,}8495$ visitas. |
| Sobredispersão e quase-Poisson na teoria | $\hat\phi \approx 3{,}3718$ e comparação de coeficientes e erros padrão. |
| Validação cruzada na teoria | Placar dos três modelos nas mesmas cinco dobras. |
| Preparação e conferências da Parte D | Conferência de dez colunas, exposição positiva, contagens não negativas, tamanho da tabela e quantidade de zeros. |
| Sobredispersão e quase-Poisson na Parte D | $\hat\phi \approx 4{,}209$ e tabela de estimativas e erros padrão. |
| Validação cruzada na Parte D | Placar dos três modelos, com menor RMSE para Poisson. |

Os procedimentos computacionais possuem código e saídas salvas no notebook, e os exercícios conceituais estão desenvolvidos no Anexo A. [3]

## 4. Síntese da experiência de estudo e aplicação

A atividade integrou os conceitos de regressão de Poisson à execução de códigos em R, passando por contagens, interpretação de coeficientes, previsões, tempo de exposição, sobredispersão e comparação de desempenho. Os exercícios computacionais permitiram verificar as diferenças entre a regressão linear e a Poisson, o efeito do *offset* e a necessidade da quase-Poisson quando a dispersão excede a esperada.

A aplicação ao banco do grupo levou esses procedimentos aos registros mensais por município, com preparação de dados, ajuste dos modelos e avaliação em cinco dobras. Os resultados mostram uma sequência coerente de análise, interpretação e comparação preditiva. O Anexo A apresenta as respostas comentadas de todas as questões dos guias como material integrante do consolidado.

## 5. Cuidados metodológicos e continuidade

A Parte D utiliza uma base exploratória em que a redução de repetições foi aproximada. Antes de usar as contagens como estatística oficial de óbitos, é necessário conferir as junções da Entrega 1 e garantir a unicidade dos registros. O *offset* considera a duração do mês, não a população municipal; portanto, os coeficientes descrevem volumes ou taxas diárias de registros, não riscos individuais de morte entre municípios. [3, célula 8; 4]

A análise mensal pode complementar a pergunta principal do trabalho, mediante alinhamento com a ficha de pesquisa do grupo. Os resultados computacionais e as respostas comentadas ficam reunidos neste documento para consulta, apresentação e revisão.

## 6. Encerramento

O consolidado reúne o percurso dos **Módulos 1 a 9**, os resultados numéricos do notebook, a interpretação dos modelos, as duas comparações de validação cruzada e a resolução escrita dos nove checkpoints e das nove perguntas finais. Os códigos e o dataset são mantidos como materiais complementares, permitindo reprodução e aperfeiçoamento da análise.

---

# Anexo A — Respostas de referência aos checkpoints e testes finais

**Natureza deste anexo:** respostas comentadas, organizadas a partir dos enunciados dos guias e dos resultados observados no notebook, para documentar integralmente a parte conceitual da atividade.

## A1. Checkpoints da teoria

### Checkpoint 1 — Contagens e Poisson

**Pergunta:** um corretor recebe, em média, $4$ ligações por dia, segundo uma Poisson. Qual é a probabilidade de não receber nenhuma ligação e qual é a variância?

**Resposta:** $P(Y=0)=e^{-4}\approx0{,}0183$, cerca de $1{,}83\%$. A variância é igual à média: $\operatorname{Var}(Y)=4$. [1, M1]

### Checkpoint 2 — Efeito multiplicativo

**Pergunta:** para $\log\lambda=1+0{,}3x$, com $x$ igual ao número de quartos, calcular a média para zero e dois quartos e a variação percentual por quarto.

**Resposta:** com zero quartos, $\lambda=e^1\approx2{,}72$. Com dois quartos, $\lambda=e^{1+0{,}3\cdot2}=e^{1{,}6}\approx4{,}95$. Cada quarto multiplica a média por $e^{0{,}3}\approx1{,}35$, correspondendo a aproximadamente $35\%$ de aumento. [1, M2]

### Checkpoint 3 — Previsão na escala da resposta

**Pergunta:** prever as visitas em trinta dias para um anúncio a $3$ km com $5$ fotos, usando os coeficientes do modelo.

**Resposta:** $\log\hat\lambda=1{,}5738-0{,}5217\cdot3+0{,}0653\cdot5=0{,}3352$. Logo, $\hat\lambda=e^{0{,}3352}\approx1{,}40$ visita em trinta dias. [1, M3]

### Checkpoint 4 — Taxa e exposição

**Pergunta:** o anúncio C teve $9$ visitas em $45$ dias; o D, $5$ em $10$ dias. Qual é mais procurado? Quantas visitas esperar com taxa de $0{,}3$ por dia e exposição de $12$ dias?

**Resposta:** C tem taxa de $9/45=0{,}2$ visita por dia; D tem $5/10=0{,}5$. D é mais procurado quando se considera o tempo de exposição. No segundo caso, a média esperada é $0{,}3\cdot12=3{,}6$ visitas. [1, M4]

### Checkpoint 5 — Correção do erro padrão

**Pergunta:** com $\hat\phi=2{,}25$, coeficiente $0{,}12$ e erro padrão $0{,}05$, qual é o erro padrão corrigido? A razão $\lvert z\rvert$ continua acima de $2$?

**Resposta:** o fator de correção é $\sqrt{2{,}25}=1{,}5$. O erro padrão corrigido é $0{,}05\cdot1{,}5=0{,}075$. A razão passa de $0{,}12/0{,}05=2{,}4$ para $0{,}12/0{,}075=1{,}6$, deixando de superar $2$. [1, M5]

### Checkpoint 6 — Previsão positiva

**Pergunta:** o MQO previu $-3{,}38$ visitas. O que o Poisson garante que o MQO não garante, e por quê?

**Resposta:** com a ligação logarítmica utilizada, a média prevista é $\hat\lambda=e^{\eta}>0$ para um preditor linear finito. Assim, o Poisson não produz uma contagem esperada negativa como a previsão do MQO nesse exemplo. [1, M6]

## A2. Teste final da teoria — Parte C do guia

### Questão 1 — Dia sem proposta

**Pergunta:** chegam, em média, $0{,}5$ propostas por dia, segundo uma Poisson. Qual é a chance de um dia sem proposta e qual é a variância?

**Resposta:** $P(Y=0)=e^{-0{,}5}\approx0{,}607$, ou aproximadamente $60{,}7\%$. A variância é $0{,}5$. [1, Parte C]

### Questão 2 — Dez anos a mais

**Pergunta:** a idade do imóvel tem coeficiente $\beta=-0{,}1$ por ano. O que acontece com as visitas esperadas quando o imóvel é dez anos mais velho?

**Resposta:** mantendo os demais preditores fixos, as visitas esperadas são multiplicadas por $e^{10(-0{,}1)}=e^{-1}\approx0{,}368$, uma redução de aproximadamente $63\%$. [1, Parte C]

### Questão 3 — Saída de `predict()`

**Pergunta:** `predict()` devolveu $2{,}3$ sem `type = "response"`. Quantas visitas o modelo prevê?

**Resposta:** a saída está na escala logarítmica. A contagem esperada é $e^{2{,}3}\approx9{,}97$, aproximadamente dez visitas. [1, Parte C]

### Questão 4 — Previsão e offset

**Pergunta:** um anúncio ficará quinze dias no ar e o modelo fornece taxa de $0{,}4$ visita por dia. Quantas visitas esperar e qual é o offset?

**Resposta:** a média esperada é $15\cdot0{,}4=6$ visitas. O offset é $\log15\approx2{,}71$. [1, Parte C]

### Questão 5 — Sobredispersão e razão da estimativa pelo erro

**Pergunta:** com $\hat\phi=4$, coeficiente $0{,}30$ e erro padrão $0{,}10$, qual é o erro padrão corrigido e qual é $\lvert z\rvert$?

**Resposta:** o fator de correção é $\sqrt4=2$. O erro padrão corrigido é $0{,}10\cdot2=0{,}20$. Assim, $\lvert z\rvert=0{,}30/0{,}20=1{,}5$, em vez de $3$. [1, Parte C]

### Questão 6 — Escolha do modelo

**Pergunta:** os RMSE de validação são $5{,}1$ para a média, $3{,}9$ para o MQO e $3{,}2$ para o Poisson. O MQO produziu previsões negativas para doze anúncios. Qual modelo escolher?

**Resposta:** o Poisson, pois apresentou o menor erro fora da amostra e não prevê contagem esperada negativa. Esses valores pertencem ao enunciado do teste, não ao placar executado no notebook. [1, Parte C]

## A3. Checkpoints da Parte D

### Checkpoint 7 — Contagem, unidade, tamanho e exposição

**Pergunta:** qual é a contagem, o que representa cada linha, quantas linhas existem e há exposição diferente entre elas?

**Resposta de referência para esta aplicação:** a resposta é a contagem aproximada de registros por município, ano e mês. Cada linha representa uma combinação dessas três variáveis, em uma tabela com $540$ linhas. A exposição temporal varia com os dias de cada mês e foi considerada no modelo. O total resulta de um tratamento exploratório de repetições, não de uma confirmação de óbitos únicos. [2, M7; 3, célula 8]

### Checkpoint 8 — Multiplicador e dispersão

**Pergunta:** interpretar um $e^{\beta}$ do modelo, informar $\hat\phi$ e dizer o que foi feito.

**Resposta de referência para esta aplicação:** o multiplicador estimado de maio é aproximadamente $1{,}061$ em relação a janeiro, equivalente a $6{,}1\%$ a mais na taxa diária esperada de registros, com município e ano fixos. Foi obtido $\hat\phi\approx4{,}209$, e o código executou a quase-Poisson para corrigir os erros padrão. [2, M8; 3, célula 8]

### Checkpoint 9 — Escolha e ligação com a Entrega 1

**Pergunta:** qual modelo levar, com base nos três números do placar, e o que isso muda na pergunta da Entrega 1?

**Resposta técnica de referência:** o Poisson teve o menor RMSE, $41{,}696$, contra $45{,}793$ do MQO e $180{,}824$ da linha de base. O resultado favorece o Poisson nessa comparação exploratória. A aplicação trata da contagem mensal de registros por município e pode servir como análise auxiliar. A análise pode complementar a pesquisa principal ao descrever a evolução temporal das contagens municipais. [2, M9; 3, célula 8]

## A4. Verificação final da Parte D

### Questão 7 — Preservação dos zeros

**Pergunta:** por que os zeros precisam permanecer na tabela de contagens?

**Resposta:** o zero informa que não houve evento naquela combinação de unidade e período. Excluí-lo elevaria a média da tabela e alteraria o nível estimado pelo modelo. Na aplicação registrada, o procedimento incluiu as combinações previstas, mas a saída não apresentou contagens zero. [2, verificação final; 3, célula 8]

### Questão 8 — Multiplicador menor que um

**Pergunta:** um preditor apresenta $e^{\beta}=0{,}8$. O que isso significa?

**Resposta:** um aumento de uma unidade no preditor, ou a mudança para a categoria comparada com a referência, multiplica a contagem esperada por $0{,}8$, isto é, reduz essa média em $20\%$, mantendo os demais preditores e a exposição fixos. O valor $0{,}8$ é o exemplo do teste, não um resultado atribuído ao modelo executado. [2, verificação final]

### Questão 9 — Quase-Poisson no placar

**Pergunta:** por que a quase-Poisson não entra separadamente no placar de validação cruzada?

**Resposta:** com os mesmos dados e a mesma especificação utilizados no exercício, Poisson e quase-Poisson produzem as mesmas previsões de média. A quase-Poisson modifica os erros padrão, não as previsões comparadas pelo RMSE; por isso, não acrescentaria um resultado diferente ao placar. [2, verificação final]

**Fechamento do anexo:** estão reunidas as resoluções dos nove checkpoints, das seis questões finais da teoria e das três questões finais da Parte D, com cálculos, interpretações e justificativas.

---

## Fontes e materiais reunidos

**[1]** `GUIA_ESTUDO_aula05b.md` — versão 1, de 24 de outubro de 2026; Módulos 1 a 6, Parte C e modelo de consolidado.

**[2]** `GUIA_ESTUDO_aula05b_parteD.md` — versão 1, de 24 de outubro de 2026; Módulos 7 a 9, verificação final e modelo de consolidado de grupo.

**[3]** `Compensatorio_Guia_5B_e_D.ipynb` — células contadas na ordem do arquivo: 1, preparação; 2, M1; 3, M3; 4, M4; 5, M5; 6, M6; 7, título “PARTE D”; 8, Módulos 7 a 9. Resultados transcritos das saídas salvas.

**[4]** `aula05b_parteD_poisson_exploratorio.R` — código da aplicação e ressalvas sobre a aproximação dos registros e a ausência de população municipal.

**Materiais de contexto preservados:** `dataset_final.xlsx`, `Entrega 1.txt` e as capturas de tela da sessão. As quantidades citadas são as registradas nas saídas do notebook.

**Documentos-base incorporados:** `aula05b_consolidado.md`, `aula05b_parteD_consolidado.md` e `aula05b_5B_e_D_consolidado.md`. O presente arquivo reúne seu conteúdo e acrescenta o Anexo A; os originais foram preservados na pasta de histórico do pacote.
