# Dados — procedência e termos de uso

## Origem

`bisnode_2012_preprocessado_v3.csv` é um derivado do painel **bisnode-firms**,
distribuído como material de apoio do livro *Data Analysis for Business,
Economics, and Policy*, de Gábor Békés e Gábor Kézdi (Cambridge University
Press).

- Repositório original: https://osf.io/b2ft9/
- Portal do livro: https://gabors-data-analysis.com

O painel bruto (`cs_bisnode_panel.csv`, ~98 MB, 287.830 registros × 48
variáveis) **não está incluído** neste repositório. Baixe-o do OSF caso queira
rodar o pré-processamento do zero.

## O que este arquivo contém

Recorte do ano-base 2012, após limpeza, tratamento de ausentes e engenharia de
atributos: **21.715 empresas × 49 variáveis**, incluindo a resposta `default`.

O pipeline que o produz está em `../preprocessamento.ipynb`.

## Termos de uso

**Os dados originais são disponibilizados para uso educacional, e a Bisnode
retém todos os demais direitos.**

Isto é mais restritivo do que a licença MIT que cobre o código deste
repositório. A licença do código **não** se estende aos dados.

Se você pretende reutilizar este arquivo, especialmente em contexto comercial,
verifique os termos na fonte original antes. Em caso de dúvida, baixe os dados
direto do OSF e aplique o pré-processamento por conta própria, em vez de
reutilizar este derivado.

A inclusão deste recorte neste repositório foi feita para fins acadêmicos e de
demonstração de trabalho.
