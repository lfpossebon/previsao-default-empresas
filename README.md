# Previsão de default de empresas

Modelo preditivo para estimar se uma empresa encerrará operações em até dois
anos, a partir de indicadores financeiros e cadastrais. Projeto Integrador da
disciplina de Aprendizagem Estatística de Máquina I — PADS Insper.

**[Ver a análise completa renderizada →](https://lfpossebon.github.io/previsao-default-empresas/)**

---

## O problema

A previsão de encerramento de operações é um problema de classificação com
duas características que condicionam as decisões de modelagem.

**Desbalanceamento de classes.** A maioria das empresas permanece em operação,
de modo que um classificador que atribua a classe majoritária a todas as
observações obtém acurácia elevada sem capacidade discriminativa.

**Assimetria de custo entre os erros.** Um falso negativo corresponde a crédito
concedido a uma empresa que encerra operações; um falso positivo corresponde à
recusa de uma operação viável. O primeiro representa perda direta, o segundo,
custo de oportunidade.

Ambas as características tornam a acurácia inadequada como métrica e limitam a
interpretação da AUC-ROC sob forte desbalanceamento. A avaliação adota AUC-PR,
F2-Score, estatística KS e a contagem direta de falsos negativos.

## Dados

Painel Bisnode de empresas europeias, ano-base 2012, com dados financeiros e
cadastrais. Após pré-processamento e engenharia de atributos: **21.715
empresas × 49 variáveis**.

A variável resposta `default` marca 1 quando a empresa encerrou operações em
até dois anos, 0 quando continuou operando.

Detalhes de origem e licença em [`data/README.md`](data/README.md).

## Modelos

Quatro famílias, todas com threshold calibrado em vez do corte padrão de 0.5:

| Modelo | Pacote | Hiperparâmetros |
|---|---|---|
| Regressão Logística | `stats::glm` | — |
| Elastic Net | `glmnet` | λ via CV de 5 folds |
| Random Forest | `ranger` | `ntrees`, `mtry`, `nodesize` via CV de 5 folds |
| XGBoost | `xgboost` | `nrounds`, `eta`, `subsample`, `max_depth` via CV de 5 folds |

## Resultados

| Modelo | Corte | AUC-PR | F2 | KS | Recall | Precisão | FN | FP |
|---|---|---|---|---|---|---|---|---|
| Random Forest | 0.33 | 0.558 | 0.676 | 0.484 | 0.883 | 0.348 | 107 | 1507 |
| XGBoost | 0.38 | 0.548 | 0.675 | 0.473 | 0.907 | 0.334 | 85 | 1651 |
| Elastic Net | 0.16 | 0.517 | 0.658 | 0.448 | 0.836 | 0.356 | 150 | 1379 |
| Reg. Logística | 0.16 | 0.517 | 0.656 | 0.441 | 0.827 | 0.358 | 158 | 1352 |

Por F2-Score, Random Forest (0,676) e XGBoost (0,675) são indistinguíveis. O
Random Forest lidera em AUC-PR; o XGBoost apresenta o maior recall e o menor
número de falsos negativos.

Aplicando a razão de custo de 3:1 enunciada no documento, o custo total
`FN x 3 + FP` produz:

| Modelo | FN | FP | Custo total |
|---|---:|---:|---:|
| Reg. Logística | 158 | 1.352 | 1.826 |
| Random Forest | 107 | 1.507 | 1.828 |
| Elastic Net | 150 | 1.379 | 1.829 |
| XGBoost | 85 | 1.651 | 1.906 |

Os três primeiros ficam contidos em 0,2% de diferença. O ponto de indiferença
entre Random Forest e Regressão Logística ocorre em uma razão de 3,04 —
praticamente sobre a premissa adotada, de modo que a ordenação entre os dois
não é robusta à estimativa de custo.

A conclusão do trabalho é que, com as métricas disponíveis, a seleção do modelo
não está determinada pelos dados e depende de uma estimativa mais precisa do
custo relativo entre os dois tipos de erro.

## Interpretabilidade

A Seção 5 do documento aplica as ferramentas de interpretabilidade sobre o
XGBoost otimizado:

- **VIP** — ranking de importância das variáveis
- **PDP** via DALEX — efeito marginal das quatro variáveis mais importantes
  sobre a probabilidade prevista, revelando não-linearidades e limiares
- **Curvas Precision-Recall** e **estatística KS** por modelo
- **Matrizes de confusão** nos thresholds calibrados

## Reproduzindo

O projeto é um site Quarto com duas partes. Requer [Quarto](https://quarto.org),
Python e R.

```bash
quarto render
```

Gera as três páginas em `docs/`.

**Parte 1** requer `pandas`, `numpy`, `altair`, `seaborn`, `matplotlib`,
`dfply` e `missingno`, além do painel bruto `cs_bisnode_panel.csv` em `data/`,
que não é versionado — ver [`data/README.md`](data/README.md) para a origem.

**Parte 2** requer as bibliotecas declaradas no início de `02-modelagem.qmd`,
entre elas `tidyverse`, `glmnet`, `ranger`, `xgboost`, `pROC`, `yardstick`,
`vip` e `DALEX`. Lê `data/bisnode_2012_preprocessado_v3.csv`, incluído no
repositório, e portanto roda sem o painel bruto.

> A versão do `vip` precisa ser a 0.4.1. Em builds de desenvolvimento
> posteriores, `vip()` retorna um `data.frame` em vez de um objeto `ggplot`, e
> os gráficos de importância não são gerados.

## Estrutura

```
index.qmd                  # visão geral e encadeamento das duas partes
01-preparacao-dados.ipynb  # parte 1 — engenharia de dados em Python
02-modelagem.qmd           # parte 2 — modelagem e avaliação em R
data/                      # base analítica + procedência e licença
docs/                      # site renderizado, publicado via GitHub Pages
_quarto.yml                # configuração do site
```

### Nota sobre as duas bases

A parte 1 gera `bisnode_2012_preprocessado_vf.csv`, com 50 colunas. A base
versionada em `data/` é a `v3`, com 49 colunas, que é a efetivamente consumida
pela parte 2. A diferença corresponde a uma variável acrescentada ao pipeline
depois que a modelagem já havia sido fechada, e por isso executar a parte 1 do
zero produz um arquivo distinto do que está versionado.

### Nota sobre os resultados

Os números publicados foram gerados pela execução atual do documento. Uma
execução anterior, em ambiente com versões diferentes de R e dos pacotes,
produziu métricas distintas para os quatro modelos. As conclusões qualitativas
— proximidade entre os modelos e sensibilidade da ordenação à razão de custo —
se mantêm, mas os valores absolutos dependem do ambiente de execução.

## Autoria

Trabalho em grupo do PADS Insper, publicado com o consentimento dos
participantes. Os nomes foram omitidos a pedido.

## Licença

O código está sob licença MIT (ver [LICENSE](LICENSE)). **A base de dados tem
termos próprios e mais restritivos** — ver [`data/README.md`](data/README.md)
antes de qualquer reutilização.
