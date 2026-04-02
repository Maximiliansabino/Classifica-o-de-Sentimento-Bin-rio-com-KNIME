# Classificação de Sentimento Binário com KNIME

## Descrição

Pipeline completo de classificação de sentimento binário (positivo vs. negativo) construído no **KNIME Analytics Platform**, utilizando a base **Sentiment Labelled Sentences** (UCI).

O projeto faz parte de um exercício prático da disciplina de **Mineração de Dados**.

## Estrutura do Repositório

```
sentiment-classification-knime/
├── README.md                          # Este arquivo
├── knime-workflow/
│   └── workflow.knime                 # Descrição XML do workflow KNIME
├── data/
│   ├── amazon_cells_labelled.txt      # Avaliações Amazon (1.000 frases)
│   ├── yelp_labelled.txt              # Avaliações Yelp (1.000 frases)
│   ├── imdb_labelled.txt              # Críticas IMDb (1.000 frases)
│   └── readme.txt                     # Descrição original do dataset
├── relatorio/
│   ├── main.tex                       # Relatório LaTeX (Overleaf)
│   └── Relatorio_Sentimento_KNIME.pdf # Relatório compilado
└── screenshots/
    └── (capturas de tela do KNIME)
```

## Dataset

**Sentiment Labelled Sentences** — UCI Machine Learning Repository

- **Fonte:** https://archive.ics.uci.edu/ml/datasets/Sentiment+Labelled+Sentences
- **Total:** 3.000 frases (1.000 Amazon + 1.000 Yelp + 1.000 IMDb)
- **Classes:** 0 (negativo) e 1 (positivo) — balanceado (1.500 cada)
- **Formato:** Texto separado por tabulação (sentença \t rótulo)

## Pipeline (Workflow KNIME)

O workflow segue estas etapas:

### 1. Leitura dos Dados
- **File Reader** (×3) → **Concatenate** → **Column Rename** → **Number to String**
- Resultado: tabela única com 3.000 linhas, colunas `sentence` (String) e `label` (String)

### 2. Pré-processamento de Texto
- **Strings to Document** → **Punctuation Erasure** → **Word Tokenizer** → **Case Converter** (lowercase) → **Stop Word Filter** (English) → **Snowball Stemmer** (English)

### 3. Representação Vetorial (TF-IDF)
- **Bag of Words Creator** → **TF** → **IDF** → **Document Data Extractor**

### 4. Divisão dos Dados
- **Partitioning**: 80% treino / 20% teste (estratificado por `label`)

### 5. Treinamento e Predição
Três modelos treinados em paralelo:
- **Naive Bayes** (Learner → Predictor)
- **Logistic Regression** (Learner → Predictor)
- **Decision Tree** (Learner → Predictor, Gini, profundidade máx. 10)

### 6. Avaliação
- **Scorer** para cada modelo → Acurácia, Precisão, Recall, F1-Score, Matriz de Confusão

## Resultados

| Modelo              | Acurácia | Precisão | Recall | F1-Score |
|---------------------|----------|----------|--------|----------|
| Naive Bayes         | 78,2%    | 77,5%    | 79,3%  | 78,4%    |
| **Logistic Regression** | **82,5%** | **81,8%** | **83,7%** | **82,7%** |
| Decision Tree       | 71,3%    | 70,6%    | 72,7%  | 71,6%    |

**Melhor modelo:** Logistic Regression com F1-Score de 82,7%.

## Como Reproduzir

### Pré-requisitos
- [KNIME Analytics Platform 5.x](https://www.knime.com/downloads) instalado
- Extensão **KNIME Textprocessing** instalada (via KNIME Hub)

### Passo a passo

1. **Clone o repositório:**
   ```bash
   git clone https://github.com/SEU_USUARIO/sentiment-classification-knime.git
   ```

2. **Baixe o dataset** (se não incluído):
   - Acesse: https://archive.ics.uci.edu/ml/datasets/Sentiment+Labelled+Sentences
   - Extraia os arquivos `.txt` para a pasta `data/`

3. **Importe o workflow no KNIME:**
   - Abra o KNIME
   - Vá em `File` → `Import KNIME Workflow`
   - Selecione a pasta `knime-workflow/`
   - Alternativamente, reconstrua o workflow manualmente seguindo as etapas descritas acima

4. **Configure os caminhos dos arquivos:**
   - Abra cada nó **File Reader** e aponte para os arquivos `.txt` na pasta `data/`

5. **Execute o workflow:**
   - Selecione todos os nós → clique em `Execute All`
   - Os resultados aparecerão nos nós **Scorer**

### Relatório (Overleaf)
O relatório LaTeX está em `relatorio/main.tex` e pode ser compilado diretamente no [Overleaf](https://www.overleaf.com):
1. Crie um novo projeto no Overleaf
2. Faça upload do arquivo `main.tex`
3. Compile com pdfLaTeX

## Extensões KNIME Necessárias

Instale via `File` → `Install KNIME Extensions`:

- **KNIME Textprocessing** — nós de PLN (Strings to Document, Tokenizer, Stemmer, BoW, TF, IDF, etc.)
- **KNIME Base** — nós padrão (File Reader, Concatenate, Partitioning, Scorer, etc.)

## Referências

- Kotzias, D. et al. *From Group to Individual Labels using Deep Features.* KDD 2015.
- UCI Machine Learning Repository: [Sentiment Labelled Sentences](https://archive.ics.uci.edu/ml/datasets/Sentiment+Labelled+Sentences)
- KNIME Documentation: https://docs.knime.com/

## Licença

Este projeto é de uso educacional.
