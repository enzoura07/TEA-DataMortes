# Consolidado — PI III — Aula 08 — Tema 1 (proximidade e janela) — 26/11/2026
*guia versão 1 · tutora: Gemini · motor09*
**Aluno:** Enzo Shinji Sugano Ura

## 1. O que foi passado
- M1 — janela par a par: entendi que a janela $w$ define a maior distância válida e que apenas a menor distância entra na soma ($\frac{1}{p_j - p_i}$). Nos exemplos didáticos, `d3` resulta em $0$ para $w = 1$ e $\frac{1}{2}$ de $w = 2$ a $5$; em `"modelo de texto"` com $w = 5$, o escore totaliza $1{,}7$.
- M2 — grade janela $\times$ $k$: analisei o impacto no ranking e verifiquei que, na consulta canônica, o AP permanece igual nas 5 janelas, enquanto $k \geq 3$ eleva o AP de $0{,}833$ para $1$. Em `"vetores de documentos"`, ampliar a janela ($w = 3$ a $5$) puxa pares frouxos e derruba o documento relevante `d2`.
- M3 — ordem literal: comparei a proximidade ordenada com a versão livre ($\text{livre}_w(q, d) = \text{prox}_w(q, d) + \text{prox}_w(\text{rev}(q), d)$). A versão livre corrige consultas invertidas como `"documentos busca"`, mas reduz o AP para $0{,}583$ na consulta do gabarito ao promover `d1` ("recuperação de informação"), continuando sujeita ao limite $w$.
- M4 — no corpus do grupo: analisei os resultados do corpus do grupo (29 documentos, 10 consultas divididas em 6 de dev e 4 de teste, limiar $\geq 2$). Com $k = 3$ e $w = 3$ definidos no dev, o BM25 e a proximidade livre atingiram MAP $1{,}000$ no dev (enquanto a ordenada caiu para $0{,}972$ devido à queda de AP na `q03`). No teste, o MAP passou de $0{,}393$ (BM25 e ordenada) para $0{,}455$ na proximidade livre ($+15{,}7\%$), destacando a melhora na `q08`.

## 2. Como foi o aprendizado — opinião da tutora
O aluno demonstrou excelente domínio conceitual da reordenação por proximidade, compreendendo com precisão o funcionamento do limite da janela $w$, a regra da menor distância entre ocorrências e a razão pela qual a ordem livre ainda depende de $w$. Além disso, soube interpretar criticamente os trade-offs entre ordem literal e livre tanto nos exemplos didáticos quanto nos dados reais do grupo, mantendo rigor metodológico na separação entre desenvolvimento e teste.
**Teste final:** acertou M1, M2, M3 e M4; a revisar nenhum módulo — domínio completo dos conceitos.

## 3. Observações para a frente
- **Revisar:** atenção à convenção de distância entre posições ($p_j - p_i$, e não span inclusivo) ao implementar fórmulas manualmente · **Perguntas guardadas:** nenhuma · **Para o grupo:** apresentar o contraste entre o teto de desempenho no dev (MAP $1{,}000$) e o ganho descritivo de $+15{,}7\%$ no teste com a proximidade livre (usando o estudo de caso da `q08`), deixando claro que a escolha de $k = 3$ e $w = 3$ respeitou o conjunto de desenvolvimento.
