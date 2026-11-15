# Prompt para gerar o experimento e os artigos (reutilizável)

Prompt usado como especificação deste repositório. Ele pode ser reaplicado a outro dataset, a outros algoritmos ou a uma nova versão das regras LLM: basta trocar os campos entre `<< >>`.

---

## Papel

Você é um pesquisador em segurança da informação para sistemas de energia (IEC 61850). Domina detecção de intrusões, aprendizado de máquina e escrita científica no padrão da Sociedade Brasileira de Computação (SBC).

## Contexto

- No SBRC 2026 publiquei o pipeline *From Red Flags to Detection Rules*. Nele, um LLM extrai *red flags* de amostras rotuladas de tráfego GOOSE e as converte em regras Python de detecção (`<<rules_llm_sbrc2026.py>>`).
- Esse pipeline é o **módulo de assinatura** de um IDS híbrido.
- Agora quero construir o **módulo de anomalia**, baseado em redes neurais e aprendizado de máquina, e avaliar a combinação dos dois módulos.
- Dataset: `<<ERENO-2.0-100K.csv>>` (ERENO, 9 classes: `normal` + 8 ataques GOOSE), na pasta `dataset/`.
- Notebook de referência de estilo: `<<SBRC_2026_LLM_IDS_GOOSE_v1.ipynb>>`. Seguir a mesma estrutura: seções numeradas, sementes fixas, saídas exportadas e gráfico de barras empilhadas por classe de ataque.
- Referências: `<<referencias.bib>>` e `<<export.bib>>`. Usar somente obras desses arquivos ou clássicas e verificáveis.

## Tarefa 1: ferramenta (notebook Jupyter)

Crie `IDS_GOOSE_Anomalia_MLP_RF.ipynb` com:

1. **Configuração.** Variável `DATASET`, que escolhe a base num dicionário `DATASETS` e aceita `.csv` ou `.zip`. Variável `ALGORITMO` = `"MLP"`, `"RF"` ou `"AMBOS"`. Conjunto de atributos e hiperparâmetros editáveis. Menus opcionais com `ipywidgets`.
2. **Ingestão e limpeza.**
   - Remover linhas de cabeçalho ARFF embutidas.
   - Descartar colunas categóricas constantes.
   - Remover carimbos de tempo absolutos (`Time`, `t`, `GooseTimestamp`), para que o modelo não aprenda a ordem da simulação.
3. **Pré-processamento.** Divisão estratificada 70/30 com `random_state=42`. Padronização ajustada só no treino (necessária para a MLP).
4. **Treinamento.**
   - MLP Perceptron: (128, 64), ReLU, Adam, L2, *early stopping*.
   - Random Forest: 100 árvores.
   - Gráfico das curvas de perda e validação da MLP.
5. **Teste do IDS.** Métricas multiclasse (acurácia, P/R/F1 macro) e binárias normal × ataque (TPR, FPR, F1). Latência em lote e unitária, esta com `n_jobs=1`.
6. **Matrizes de confusão** multiclasse (% por linha) e binária, por modelo.
7. **Tráfego anômalo real × detectado** por classe de ataque.
8. **Falhas de segurança do GOOSE** exploradas e detectadas, em barras horizontais empilhadas por ataque:
   - F1: autenticação de origem
   - F2: integridade
   - F3: anti-replay
   - F4: validação de stNum/sqNum e taxa
   - F5: garantia de entrega
9. **Comparativo MLP × RF.** Métricas, latência frente ao limite de 4 ms, F1 por classe e importância de atributos. Incluir uma ablação do RF com e sem as medidas elétricas (19 × 37 atributos), na mesma divisão.
10. **IDS híbrido.** Aplicar as regras LLM ao mesmo teste e avaliar:
    - regras isoladas;
    - fusão OU com todas as regras;
    - fusão OU **seletiva**, só com regras de FPR ≤ 0,5% em tráfego normal **medido no treino**.
11. **Exportação.** Figuras em PNG e PDF, CSVs e JSON de metadados.

Requisitos visuais: paleta categórica fixa (uma cor por ataque, `normal` em cinza), escala sequencial de um só matiz nas matrizes, um único eixo por gráfico e rótulos em português.

## Tarefa 2: artigos (LaTeX, modelo SBC)

Usar **apenas números produzidos pela execução do notebook**. Nunca inventar resultados.

- **Artigo A (Overleaf, ~8 páginas).** Resumo/Abstract, Introdução, Trabalhos Relacionados, Proposta (diagrama TikZ da arquitetura híbrida), Metodologia, Resultados (tabela global, matriz de confusão, falhas GOOSE, fusão híbrida) e Conclusão.
- **Artigo B (versão completa, 20 páginas, sem capa: artigo para simpósio da SBC).** Resumo/Abstract, Introdução com contribuições, Fundamentação (GOOSE, falhas F1–F5, IDS, MLP, RF, regras LLM), Trabalhos Relacionados com tabela comparativa, Proposta com formalização da fusão seletiva, Metodologia (dataset, atributos, hiperparâmetros, métricas, ferramenta), Resultados com todas as figuras, Discussão (escolha do modelo, por que grayhole/replay são difíceis, papel das regras, ameaças à validade) e Conclusão com trabalhos futuros.
- Compilar os dois PDFs e conferir número de páginas, citações indefinidas e figuras sobrepostas.
- Auditar por script: todo número do texto deve existir nos CSVs de `resultados/dados/`. Latências variam entre execuções; reportar a execução do notebook e a faixa observada.

## Restrições

- Português do Brasil, tom acadêmico, sem exageros. Números com vírgula decimal.
- Descrever honestamente as limitações. Exemplos: dataset sintético; divisão aleatória por mensagem; regras geradas sobre o mesmo dataset; latência medida em contêiner.
- Marcar com `[VERIFICAR]` no `.bib` os metadados incertos.
