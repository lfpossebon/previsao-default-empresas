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
| Random Forest | 0.34 | 0.430 | 0.587 | 0.508 | 0.830 | 0.271 | 93 | 1224 |
| Elastic Net | 0.13 | 0.407 | 0.564 | 0.483 | 0.760 | 0.277 | 131 | 1086 |
| Regressão Logística | 0.14 | 0.405 | 0.566 | 0.484 | 0.735 | 0.294 | 145 | 963 |
| XGBoost | 0.14 | 0.418 | 0.553 | 0.464 | 0.722 | 0.285 | 152 | 990 |

O Random Forest apresenta o melhor desempenho em cinco das seis métricas,
identificando 83,0% dos defaults contra 72,2% do XGBoost, ao custo de 1.224
falsos positivos contra 990.

A seleção do modelo, contudo, depende da parametrização de custo adotada.
Assumindo que um default equivale à receita de três contratos adimplentes —
premissa enunciada na Seção 4.1 do documento — o custo total `FN x 3 + FP`
inverte o ranking:

| Modelo | FN | FP | Custo total |
|---|---:|---:|---:|
| Reg. Logística | 145 | 963 | 1.398 |
| XGBoost | 152 | 990 | 1.446 |
| Elastic Net | 131 | 1.086 | 1.479 |
| Random Forest | 93 | 1.224 | 1.503 |

Sob 3:1 a Regressão Logística minimiza o custo total. O ponto de indiferença
entre os dois modelos ocorre em uma razão de 5,02: abaixo dela a Regressão
Logística é preferível; acima, o Random Forest.

Registre-se que o XGBoost, usualmente competitivo nesse tipo de tarefa,
apresentou o menor recall do conjunto. Sob classes desbalanceadas e preditoras
financeiras correlacionadas, o *bagging* mostrou desempenho superior ao
*boosting* neste caso.

## Interpretabilidade

A Seção 5 do documento aplica as ferramentas de interpretabilidade sobre o
XGBoost otimizado:

- **VIP** — ranking de importância das variáveis
- **PDP** via DALEX — efeito marginal das quatro variáveis mais importantes
  sobre a probabilidade prevista, revelando não-linearidades e limiares
- **Curvas Precision-Recall** e **estatística KS** por modelo
- **Matrizes de confusão** nos thresholds calibrados

## Reproduzindo

Requer R e [Quarto](https://quarto.org). As dependências estão declaradas no
início de `modelagem.qmd` — entre elas `tidyverse`, `glmnet`, `ranger`,
`xgboost`, `pROC`, `yardstick`, `vip` e `DALEX`.

```bash
quarto render modelagem.qmd
```

O documento lê `data/bisnode_2012_preprocessado_v3.csv`, já incluído no
repositório.

## Estrutura

```
modelagem.qmd          # análise completa em R — modelos, métricas, interpretabilidade
preprocessamento.ipynb # pipeline de preparação em Python (limpeza, feature engineering)
data/                  # base pré-processada + procedência
docs/index.html        # análise renderizada, publicada via GitHub Pages
```

### Nota sobre o pré-processamento

`preprocessamento.ipynb` é a iteração mais recente do pipeline de preparação e
gera 50 colunas. A base versionada em `data/` tem 49 e corresponde à versão
efetivamente consumida pela modelagem. Rodar o notebook do zero produz um
arquivo ligeiramente diferente do que está aqui — a diferença está em uma
variável adicionada depois que a modelagem já havia sido fechada.

## Autoria

Trabalho em grupo do PADS Insper, publicado com o consentimento dos
participantes. Os nomes foram omitidos a pedido.

## Licença

O código está sob licença MIT (ver [LICENSE](LICENSE)). **A base de dados tem
termos próprios e mais restritivos** — ver [`data/README.md`](data/README.md)
antes de qualquer reutilização.
