# Previsão de Preços no Airbnb · Rio de Janeiro

Modelo de regressão que estima o **preço da diária** de um imóvel no Airbnb do Rio de Janeiro a partir das suas características. O objetivo é ajudar:
- **quem anuncia** a saber quanto cobrar;
- **quem procura** a saber se um imóvel está com preço atrativo em comparação com imóveis parecidos.

![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?logo=pandas&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?logo=scikitlearn&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-3F4F75?logo=plotly&logoColor=white)

---

## Resultados

Comparação de três modelos no conjunto de teste (25% dos dados):

| Modelo | R² | MAE (R$) | RMSE (R$) |
|---|---|---|---|
| Regressão Linear (baseline) | 32,7% | 159,63 | 217,60 |
| Random Forest | 97,3% | 14,50 | 44,01 |
| **Extra Trees** | **97,5%** | **11,54** | **41,97** |

O **Extra Trees** foi o modelo escolhido. Ele erra, em média, cerca de R$ 11,50 no preço da diária, e foi salvo com `joblib` para uso em produção.

### O que mais influencia o preço
Importância das variáveis no Extra Trees:
1. Número de quartos (`bedrooms`)
2. Localização (`latitude` e `longitude`)
3. Quantidade de comodidades (`n_amenities`)
4. Taxa por hóspede extra (`extra_people`)
5. Banheiros e capacidade de hóspedes

A expectativa inicial de que a **localização** pesaria muito se confirmou. Já a **sazonalidade** (`mes`) teve peso baixo.

---

## Dados
- **Fonte:** [Airbnb Rio de Janeiro · Kaggle](https://www.kaggle.com/datasets/allanbruno/airbnb-rio-de-janeiro).
- **Período:** bases mensais de abril de 2018 a maio de 2020 (sem junho de 2018).
- **Tamanho:** 902.210 registros e 108 colunas antes do tratamento; **557.640 registros e 20 variáveis** depois.

## Etapas do projeto

1. **Consolidação:** junção das bases mensais, com colunas de ano e mês.
2. **Seleção de colunas:** análise qualitativa dos primeiros registros. Saíram IDs, links, textos livres, colunas repetidas e colunas quase constantes.
3. **Valores faltantes:**
   - colunas com mais de 300 mil valores nulos foram excluídas;
   - nas demais, as poucas linhas com nulos foram removidas.
4. **Outliers:** removidos com o critério do intervalo interquartil (1,5 × IQR) em preço, hóspedes extras, imóveis por anfitrião, capacidade, banheiros, quartos, camas, noites mínimas e comodidades.
   - **Decisão de negócio:** o foco são imóveis comuns e anfitriões pessoa física. Imóveis de altíssimo luxo e anfitriões com mais de 6 anúncios (imobiliárias) ficaram fora do público-alvo.
5. **Engenharia de variáveis:**
   - a lista de comodidades virou a contagem `n_amenities`;
   - categorias raras foram agrupadas (tipo de imóvel, tipo de cama, política de cancelamento).
6. **Visualização:** mapa de densidade de preços com Plotly.
7. **Encoding:** variáveis binárias em 0/1 e one-hot encoding nas categóricas.
8. **Modelagem:** comparação de Regressão Linear, Random Forest e Extra Trees com R², MAE e RMSE.
9. **Ajuste:** remoção de variáveis com importância quase nula, simplificando o modelo sem perder desempenho.
10. **Deploy:** exportação do modelo (`modelo.joblib`) e da base tratada.

## Limitações e próximos passos
- **Vazamento entre meses:** o mesmo imóvel aparece em várias bases mensais, e a divisão treino/teste foi aleatória. Então um anúncio pode estar no treino e no teste ao mesmo tempo, o que tende a inflar o R². O próximo passo é dividir por anúncio (ex.: `GroupKFold`) ou por período (treinar no passado e testar nos meses seguintes).
- **Estabilidade:** usar validação cruzada para reportar média e desvio padrão, em vez de uma única divisão.
- **Comodidades:** avaliar o impacto de comodidades específicas (piscina, ar-condicionado, vista), e não só a quantidade.

## Como executar

```bash
pip install pandas numpy scikit-learn matplotlib seaborn plotly joblib
```

Baixe as bases mensais do Kaggle, coloque-as em uma pasta `dataset/` e execute o notebook `Solução Airbnb Rio.ipynb`.

## Créditos
Projeto desenvolvido como estudo guiado. A solução de referência, citada no notebook, é o trabalho de [Allan Bruno no Kaggle](https://www.kaggle.com/allanbruno/helping-regular-people-price-listings-on-airbnb).
