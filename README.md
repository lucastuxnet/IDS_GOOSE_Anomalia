# IDS por Anomalia para GOOSE (IEC 61850) com MLP e Random Forest

### Módulo de anomalia de um IDS híbrido: rede neural + regras geradas por LLM (SBRC 2026)

**Autor:** Lucas A. Martins — PPGEELT/UFU · `lucas.martins@ufu.br`

Este repositório traz a ferramenta (notebook Jupyter), os resultados e os artigos (modelo SBC) do **módulo de detecção por anomalia** de um IDS híbrido para o protocolo GOOSE:

- O **módulo de assinatura** do IDS é o pipeline LLM publicado no SBRC 2026 ([lucastuxnet/SBRC_2026](https://github.com/lucastuxnet/SBRC_2026)).
- Este repositório treina e compara um **MLP (Multilayer Perceptron)** e um **Random Forest** no dataset ERENO e depois combina o modelo com as regras LLM.

```
Tráfego GOOSE ─▶ Extração de atributos ─┬─▶ Regras LLM (SBRC 2026) ──┐
                                         └─▶ MLP / Random Forest ─────┴─▶ Fusão OU seletiva ─▶ Alerta
```

## Resultados principais (teste: 60.000 mensagens)

| Modelo | Acurácia | F1-macro | TPR (ataque) | FPR | Inferência unitária |
|---|---:|---:|---:|---:|---:|
| MLP | 89,98% | 91,09% | 93,53% | 16,12% | 0,20 ms |
| Random Forest | **92,41%** | **93,49%** | **93,95%** | **13,11%** | 5,3 ms |
| Híbrido seletivo (RF + 16 regras LLM) | — | — | 94,04% | 14,26% | — |

- **Falhas bem cobertas.** Os dois modelos detectam mais de 99,7% dos ataques que exploram três falhas do GOOSE:
  - falta de autenticação de origem (*masquerade*);
  - falta de integridade (*injection*);
  - falta de validação de `stNum`/`sqNum` (*high_StNum*, *poisoned_high_rate*).
- **Falhas em aberto.**
  - Falta de garantia de entrega (*grayhole*): 63–71% de detecção.
  - Falta de proteção anti-replay: 89–94%.
- **Medidas elétricas importam.** Só com os 19 atributos de protocolo, o F1-macro do RF cai para 91,36% e o FPR sobe para 16,37% (ablação na §10.1).
- **Fusão com as regras LLM.** A fusão OU com todas as regras aumenta o FPR em cerca de 4,8 pontos sem ganho real de detecção. Selecionar as regras pelo FPR medido no treino (≤ 0,5%) reduz esse custo a cerca de 1,1 ponto.

## Estrutura

```
.
├── IDS_GOOSE_Anomalia_MLP_RF.ipynb   # ferramenta principal (já executada, com saídas)
├── rules_llm_sbrc2026.py             # 22 regras geradas por LLM no SBRC 2026 (módulo de assinatura)
├── requirements.txt
├── PROMPT.md                         # prompt-especificação reutilizável do experimento e dos artigos
├── dataset/
│   └── ERENO-2.0-100K.zip            # ERENO compactado (34 MB); o notebook lê o .zip direto
├── resultados/
│   ├── figuras/                      # fig01–fig10 em PNG (300 dpi) e PDF
│   └── dados/                        # métricas, matrizes de confusão, detecção por classe/falha, híbrido
└── artigos/
    ├── sbc_overleaf/                 # artigo curto (8 páginas) — main.tex + main.pdf
    └── sbc_completo/                # versão completa (20 páginas, sem capa) — main.tex + main.pdf
```

## Como executar

```bash
git clone https://github.com/lucastuxnet/IDS_GOOSE_Anomalia.git
cd IDS_GOOSE_Anomalia
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook IDS_GOOSE_Anomalia_MLP_RF.ipynb
```

Na **§1 — Configuração**, escolha:

| Variável | Valores | Efeito |
|---|---|---|
| `DATASET` | chave de `DATASETS` (padrão `"ERENO-2.0-100K"`) | base usada; aceita `.csv` ou `.zip` em `dataset/` |
| `ALGORITMO` | `"MLP"`, `"RF"`, `"AMBOS"` | algoritmo treinado; `AMBOS` gera o comparativo |
| `CONJUNTO_ATRIBUTOS` | `"protocolo_eletrico"` (37) ou `"protocolo"` (19) | com ou sem as medidas elétricas |
| `USAR_REGRAS_LLM` | `True` / `False` | executa a §11 (IDS híbrido) |
| `LIMIAR_FPR_REGRA` | padrão `0.005` | limiar da fusão seletiva |

A execução completa (os dois algoritmos e a fusão) leva cerca de 2,5 minutos em 2 vCPUs.

## Seções do notebook

| § | Conteúdo |
|---|---|
| 1 | Configuração (dataset, algoritmo, atributos, hiperparâmetros) |
| 2–4 | Setup, ingestão e limpeza do ERENO, divisão 70/30 e padronização |
| 5 | Treinamento e curvas da MLP |
| 6 | Teste do IDS: métricas multiclasse e binárias, latência |
| 7 | Matrizes de confusão (multiclasse e binária) |
| 8 | Tráfego anômalo real × detectado por classe |
| 9 | Falhas de segurança do GOOSE exploradas e detectadas |
| 10 | Comparativo MLP × RF, F1 por classe, importância de atributos, ablação de atributos |
| 11 | IDS híbrido: regras LLM, seleção pelo treino, fusões |
| 12 | Exportação de CSVs, figuras e metadados |

## Artigos (Overleaf)

Cada pasta em `artigos/` é um projeto LaTeX completo: `main.tex`, `referencias.bib`, `arquitetura.tex` (TikZ), `figuras/`, `sbc-template.sty` e `sbc.bst`. Para usar no Overleaf, faça o upload da pasta como `.zip`: **New Project → Upload Project**.

> **Atenção ao estilo:** o `sbc-template.sty` e o `sbc.bst` incluídos são versões **compatíveis**, feitas para compilação local. Antes de submeter, substitua-os pelos arquivos oficiais do modelo *SBC Conferences Template* (Overleaf/site da SBC). Nenhum comando do `main.tex` precisa mudar. A contagem de páginas (8 e 20) foi conferida com o estilo compatível; confira de novo depois da troca.

Os e-mails dos coautores no `main.tex` estão marcados com `[CONFIRMAR]`.

## Reprodutibilidade e limitações

- Sementes fixas (`random_state=42`); todos os números dos artigos saem de `resultados/dados/`.
- O ERENO é sintético, e a divisão é aleatória por mensagem. As regras LLM foram geradas a partir do mesmo dataset. Esses pontos estão discutidos como ameaças à validade no artigo longo.
- A latência foi medida em contêiner compartilhado. Repita a medição no equipamento de implantação.

## Referência ao trabalho anterior

> L. A. Martins, C. B. Quincozes, G. Siervo, S. E. Quincozes, M. C. Luizelli. *From Red Flags to Detection Rules: An LLM-driven Pipeline for Real-Time GOOSE Intrusion Detection and Prevention.* Anais Estendidos do SBRC 2026, pp. 57–65. DOI: 10.5753/sbrc_estendido.2026.23263.

## Licença

MIT.
