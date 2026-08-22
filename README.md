# Previsão de default de empresas

Modelo preditivo para estimar se uma empresa encerrará operações em até dois
anos, a partir de indicadores financeiros e cadastrais. Projeto Integrador da
disciplina de Aprendizagem Estatística de Máquina I — PADS Insper.

**[Ver a análise completa renderizada →](https://lfpossebon.github.io/previsao-default-empresas/)**

---

## O problema

Prever falência é um problema de classificação com duas características que
condicionam todas as decisões de modelagem:

**As classes são desbalanceadas.** A maioria das empresas continua operando, e
um classificador que responde "não vai quebrar" para todo mundo acerta a maior
parte das vezes sem ter aprendido nada.

**Os erros custam valores diferentes.** Deixar de identificar uma empresa que
vai quebrar — falso negativo — significa crédito concedido que não volta.
Sinalizar uma empresa saudável como risco — falso positivo — significa negócio
perdido. Em crédito, o primeiro erro é caro; o segundo é um custo de
oportunidade.

Isso orienta a escolha de métricas: acurácia é inútil aqui, e mesmo a AUC-ROC
padrão engana sob desbalanceamento. A avaliação usa **AUC-PR**, **F2** (que
pesa recall acima de precisão), **KS** e a contagem direta de falsos negativos.

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
| **Random Forest** | 0.34 | **0.430** | **0.587** | **0.508** | **0.830** | 0.271 | **93** | 1224 |
| Elastic Net | 0.13 | 0.407 | 0.564 | 0.483 | 0.760 | 0.277 | 131 | 1086 |
| Regressão Logística | 0.14 | 0.405 | 0.566 | 0.484 | 0.735 | **0.294** | 145 | 963 |
| XGBoost | 0.14 | 0.418 | 0.553 | 0.464 | 0.722 | 0.285 | 152 | 990 |

**Random Forest vence em cinco das seis métricas relevantes** e captura 83% dos
defaults, contra 72% do XGBoost. O preço é explícito: 1.224 falsos positivos
contra 990 do XGBoost.

Essa é a decisão central do trabalho. O Random Forest deixa passar 93 empresas
que quebraram, o XGBoost deixa passar 152 — 59 a mais. Em troca, o Random
Forest sinaliza 234 empresas saudáveis a mais como risco. Se o custo de um
falso negativo supera o de um falso positivo em mais de ~4:1, o Random Forest é
a escolha economicamente correta mesmo perdendo em precisão.

Vale notar que o XGBoost, geralmente o favorito nesse tipo de tarefa, ficou em
último em recall. Sob classes desbalanceadas e preditoras financeiras
fortemente correlacionadas, a estabilidade do *bagging* superou o *boosting*.

## Interpretabilidade

O modelo vencedor não é tratado como caixa-preta:

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
