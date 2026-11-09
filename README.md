# Detecção de anomalias em transações de cartão

Projeto educacional de **Caio Siqueira** para o bootcamp **Bradesco Agentes de IA do Zero à Prática**, na DIO.

O notebook treina classificadores para sinalizar fraudes em transações reais anonimizadas. Na base original, apenas **492 de 284.807 transações (0,172%)** são fraudes. Liberar tudo daria cerca de 99,83% de acurácia, com recall zero na classe que importa.

- [Notebook executado com tabelas, curvas e explicações](deteccao_anomalias.ipynb)
- [Dependências para reprodução](requirements.txt)

## Preparação e avaliação

1. Carregamento do [dataset público da ULB e Worldline](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud), com [espelho no Zenodo](https://zenodo.org/records/7395559).
2. Conferência do esquema, remoção de linhas integralmente repetidas e criação de `LogAmount = log(1 + Amount)`. A base após esse tratamento contém **283,726 registros e 473 fraudes**.
3. Divisão estratificada 60/20/20 para treino, validação e teste. A padronização da regressão logística é ajustada no treino. Pesos e subamostragem também são definidos apenas no treino.
4. O limiar é escolhido na validação para maximizar recall com precisão de pelo menos 60%. F1 é o primeiro desempate. O teste é reservado para medir o desempenho após congelar a escolha.

## Comparação dos modelos

Resultados no teste, com limiares escolhidos na validação. **Precisão, recall e F1 são da classe fraude.** AP é a precisão média que resume a curva precisão–recall. FP é o número de transações legítimas sinalizadas.

| Modelo | Limiar | Precisão | Recall | F1 | AP | FP |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Random Forest ponderada | 0.217030 | 77.78% | 81.05% | 0.7938 | 0.8424 | 22 |
| XGBoost ponderado | 0.488272 | 72.73% | 84.21% | 0.7805 | 0.8429 | 30 |
| Logística baseline | 0.022919 | 64.71% | 81.05% | 0.7196 | 0.7454 | 42 |
| Logística ponderada | 0.999852 | 78.12% | 78.95% | 0.7853 | 0.7253 | 21 |
| Logística subamostrada | 0.764867 | 65.55% | 82.11% | 0.7290 | 0.7192 | 41 |

**Modelo selecionado na validação: Random Forest ponderada.** Com limiar **0.217030**, ele sinalizou **77 de 95 fraudes** e gerou **22 falsos alertas** em 56,746 transações de teste. No corte 0,5, o mesmo modelo sinalizou 72 fraudes e gerou 2 falsos alertas. A meta de precisão foi usada na validação e não representa garantia fora da amostra.

## O que o SHAP mostrou

O notebook calcula valores TreeSHAP do **XGBoost**, inclusive quando outro modelo é escolhido para o teste principal. Os componentes com maior importância absoluta média na amostra foram `V14`, `V4`, `V12`, `V3`, `V10`. Para a fraude sinalizada, o escore foi **0.999929** com corte do XGBoost **0.488272**. O maior efeito positivo foi `V14` (+3.3365) e o maior efeito negativo foi `V8` (-0.4294), em log-odds. A soma das contribuições com o viés reconstitui a margem do modelo. Componentes PCA explicam a decisão matemática e têm significado comercial limitado.

## Decisões próprias em relação ao roteiro da aula

A implementação segue os modelos indicados no roteiro e acrescenta uma validação separada para congelar modelo e limiar antes do teste, um piso explícito de precisão, remoção de duplicatas integrais antes da divisão, comparação com um corte de 0,5 e uma experiência controlada de subamostragem. A decomposição local do TreeSHAP tem uma verificação numérica de aditividade.

## Como reproduzir

Com Python 3.12, instale `requirements.txt` em um ambiente virtual e execute todas as células do notebook no JupyterLab ou no VS Code. O download é feito pelo notebook. Para trabalhar offline, defina `CREDITCARD_CSV` com o caminho de uma cópia local da mesma base. A base bruta fica fora do repositório.

A avaliação é retrospectiva. Dois dias de transações e uma quantidade pequena de fraudes limitam a generalização. Uma aplicação real exige validação temporal e dados atuais, avaliação do custo de revisão e monitoramento de mudanças na prevalência.

[Enunciado da DIO](https://web.dio.me/lab/deteccao-de-anomalias-em-transacoes/learning/e38d0c1c-a7e9-4d74-b79c-9a5584578581?back=/track/bradesco-agentes-de-ia-do-zero-a-pratica)
