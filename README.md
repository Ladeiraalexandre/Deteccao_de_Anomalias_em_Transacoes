# Detecção de Fraudes em Cartões de Crédito

## 1. O Problema e o Impacto do Desbalanceamento
A detecção de fraude é caracterizada por um desequilíbrio severo de classes (neste caso, as fraudes representam apenas ~0,17% das transações). Em cenários altamente desbalanceados, a métrica de **Acurácia** torna-se ilusória (um modelo ingênuo que classifique tudo como legítimo alcançaria 99.83% de acurácia, mas detectaria 0% das fraudes).

Para avaliar de forma realista, priorizamos:
* **Recall (Sensibilidade)**: A proporção de fraudes reais que conseguimos identificar (essencial para minimizar prejuízos).
* **Precisão**: A proporção de alarmes que são realmente fraudes (essencial para evitar fricção desnecessária com bons clientes).
* **Curva Precision-Recall (PR)**: Mais informativa do que a curva ROC para este cenário, pois foca diretamente no comportamento da classe minoritária.

---

## 2. Preparação de Dados
Para preparar as variáveis antes do treinamento, aplicamos as seguintes transformações:
* **Log1p (`np.log1p`)**: Aplicado ao atributo de valor (`Amount`) para reduzir a variabilidade extrema e aproximar sua distribuição de uma curva normal.
* **StandardScaler**: Padronização dos dados para garantir que todas as variáveis fiquem na mesma escala, evitando distorções em algoritmos sensíveis como Regressão Logística.
* **Divisão Estratificada (`stratify=y`)**: Divisão em 80% treino e 20% teste garantindo que a proporção exata de fraudes seja mantida em ambas as partições.

---

## 3. Comparação de Modelos (Classe 1 - Fraude)
Após o treinamento dos modelos com técnicas de ponderação de classe, obtivemos os seguintes resultados aproximados para a classe de fraude (**Classe 1**):

| Modelo | Precisão | Recall | F1-Score |
| :--- | :---: | :---: | :---: |
| **Regressão Logística** | 0.06 | 0.92 | 0.11 |
| **Random Forest (Balanced)** | 0.79 | 0.83 | 0.81 |
| **XGBoost (Weighted)** | 0.87 | 0.83 | 0.85 |

*O XGBoost e o Random Forest apresentaram um equilíbrio de F1-Score muito superior ao baseline de Regressão Logística, mantendo um Recall robusto com muito menos falsos positivos.*

---

## 4. Otimização e Explicabilidade (SHAP)
**Pipeline & GridSearch**: Criamos um pipeline integrado com `StandardScaler` e `RandomForestClassifier` otimizando hiperparâmetros diretamente pelo objetivo de **Recall**.
 **Limiar de Decisão Customizado (0.3)**: Reduzimos o limiar de decisão de 0.5 para 0.3 para priorizar a captura de fraudes, maximizando o Recall às custas de uma pequena redução na Precisão.
 **Explicabilidade Global com SHAP**: A análise indicou quais variáveis latentes (como as componentes V14, V12 e V4) possuem o maior peso na contribuição do cálculo do risco de fraude das transações.

---

## 5. Melhorias em Relação ao Pipeline Anterior
* **Sem Vazamento de Dados (Data Leakage)**: Introdução do `Pipeline` do scikit-learn garantindo que a escala e o ajuste (`fit`) ocorram estritamente no conjunto de treino.
* **Estratificação na Divisão**: Garante estabilidade estatística em partições de dados desbalanceados.
* **Otimização Focada**: Uso do `GridSearchCV` focado em maximizar o `recall` ao invés de acurácia padrão.
