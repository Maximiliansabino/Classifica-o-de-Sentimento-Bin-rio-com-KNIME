# Guia Passo a Passo — Construção do Workflow no KNIME

Este guia detalha como reconstruir o workflow manualmente no KNIME Analytics Platform.

---

## Pré-requisitos

1. Instale o **KNIME Analytics Platform 5.x**: https://www.knime.com/downloads
2. Instale a extensão **KNIME Textprocessing**:
   - Menu: `File` → `Install KNIME Extensions`
   - Busque por "Textprocessing" e instale

---

## Etapa 1: Leitura dos Dados

### 1.1 — Adicione 3 nós **File Reader**
- Arraste o nó `File Reader` do painel de nós para a área de trabalho (3 vezes)
- Configure cada um:
  - **File Reader #1**: aponte para `data/amazon_cells_labelled.txt`
  - **File Reader #2**: aponte para `data/yelp_labelled.txt`
  - **File Reader #3**: aponte para `data/imdb_labelled.txt`
- Em cada File Reader, configure:
  - Separador: **Tab** (tabulação)
  - Desmarcar "Has Column Header" (os arquivos NÃO têm cabeçalho)
  - Colunas resultantes: `Col0` (String) e `Col1` (Integer)

### 1.2 — Adicione o nó **Concatenate**
- Busque "Concatenate" no painel de nós
- Conecte as 3 saídas dos File Readers nas entradas do Concatenate
- **Nota:** O nó Concatenate padrão tem 2 entradas. Use o nó `Concatenate (Optional in)` que permite múltiplas entradas, OU use 2 nós Concatenate em cascata

### 1.3 — Adicione o nó **Column Rename**
- Conecte a saída do Concatenate ao Column Rename
- Configure:
  - `Col0` → renomear para `sentence`
  - `Col1` → renomear para `label`

### 1.4 — Adicione o nó **Number to String**
- Conecte a saída do Column Rename
- Selecione a coluna `label` para converter
- Isso transforma os valores 0 e 1 em Strings "0" e "1"

---

## Etapa 2: Pré-processamento de Texto

### 2.1 — **Strings to Document**
- Conecte a saída do Number to String
- Configure:
  - **Document column:** `sentence`
  - **Title column:** `sentence` (ou Row ID)
  - **Authors column:** (nenhum)
  - **Category column:** `label`  ← **IMPORTANTE!**
  - **Source column:** (nenhum)

### 2.2 — **Punctuation Erasure**
- Conecte a saída do Strings to Document
- Configuração: padrão (default)
- Selecione a coluna `Document`

### 2.3 — **Case Converter**
- Conecte a saída do Punctuation Erasure
- Configure: **Lowercase**
- Selecione a coluna `Document`

### 2.4 — **Stop Word Filter**
- Conecte a saída do Case Converter
- Configure:
  - **Built-in list:** English
  - Selecione a coluna `Document`

### 2.5 — **Snowball Stemmer**
- Conecte a saída do Stop Word Filter
- Configure:
  - **Language:** English
  - Selecione a coluna `Document`

---

## Etapa 3: Representação Vetorial (TF-IDF)

### 3.1 — **Bag of Words Creator**
- Conecte a saída do Snowball Stemmer
- Configuração: padrão
- Isso cria a matriz documento-termo

### 3.2 — **TF** (Term Frequency)
- Conecte a saída do Bag of Words Creator
- Configure:
  - **TF computation:** Relative frequency of term in document

### 3.3 — **IDF** (Inverse Document Frequency)
- Conecte a saída do TF
- Configuração: padrão

### 3.4 — **Document Data Extractor**
- Conecte a saída do IDF
- Configure:
  - Marque **Extract category** ← **ESSENCIAL!** (isso recupera a coluna `label`)
  - A coluna extraída será chamada `Document Category`

---

## Etapa 4: Divisão dos Dados

### 4.1 — **Partitioning**
- Conecte a saída do Document Data Extractor
- Configure:
  - **Relative (%):** 80
  - **Sampling method:** Stratified sampling
  - **Column:** `Document Category` (a coluna do label)
  - **Random seed:** (defina um valor fixo para reprodutibilidade, ex: 42)

---

## Etapa 5: Treinamento dos Modelos

A partir do Partitioning, crie 3 ramos paralelos. A **1ª saída** (porta superior) do Partitioning é o **treino** e a **2ª saída** (porta inferior) é o **teste**.

### 5.1 — Naive Bayes

- **Naive Bayes Learner:**
  - Conecte a **1ª saída** do Partitioning (treino)
  - Configure: **Class column:** `Document Category`

- **Naive Bayes Predictor:**
  - Entrada 1 (modelo): saída do Naive Bayes Learner
  - Entrada 2 (dados): **2ª saída** do Partitioning (teste)

### 5.2 — Logistic Regression

- **Logistic Regression Learner:**
  - Conecte a **1ª saída** do Partitioning (treino)
  - Configure:
    - **Target column:** `Document Category`
    - **Reference category:** `0`
    - **Max iterations:** 100
    - **Regularization:** Ridge (L2)

- **Logistic Regression Predictor:**
  - Entrada 1 (modelo): saída do Logistic Regression Learner
  - Entrada 2 (dados): **2ª saída** do Partitioning (teste)

### 5.3 — Decision Tree

- **Decision Tree Learner:**
  - Conecte a **1ª saída** do Partitioning (treino)
  - Configure:
    - **Class column:** `Document Category`
    - **Quality measure:** Gini Index
    - **Max depth:** 10
    - **Min records per node:** 5

- **Decision Tree Predictor:**
  - Entrada 1 (modelo): saída do Decision Tree Learner
  - Entrada 2 (dados): **2ª saída** do Partitioning (teste)

---

## Etapa 6: Avaliação

### Para cada modelo, adicione um nó **Scorer**:

- **Scorer (Naive Bayes):**
  - Conecte a saída do Naive Bayes Predictor
  - Configure:
    - **First column (real):** `Document Category`
    - **Second column (predito):** `Prediction (Document Category)`

- **Scorer (Logistic Regression):**
  - Conecte a saída do Logistic Regression Predictor
  - Mesma configuração

- **Scorer (Decision Tree):**
  - Conecte a saída do Decision Tree Predictor
  - Mesma configuração

---

## Execução

1. Selecione todos os nós: `Ctrl+A`
2. Execute: `Shift+F7` ou botão verde "Execute All"
3. Clique com botão direito em cada **Scorer** → "View: Confusion Matrix" para ver resultados

---

## Diagrama Resumido do Fluxo

```
File Reader (Amazon) ──┐
File Reader (Yelp)  ───┤──→ Concatenate → Column Rename → Number to String
File Reader (IMDb)  ───┘
                                                              │
                                                              ▼
Strings to Document → Punctuation Erasure → Case Converter → Stop Word Filter → Snowball Stemmer
                                                                                       │
                                                                                       ▼
                                              Bag of Words Creator → TF → IDF → Doc Data Extractor
                                                                                       │
                                                                                       ▼
                                                                                  Partitioning
                                                                                  ┌────┴────┐
                                                                               Treino     Teste
                                                                              ┌───┼───┐      │
                                                                              │   │   │      │
                                                                              ▼   ▼   ▼      │
                                                                            NB   LR  DT      │
                                                                             │   │   │       │
                                                                             ▼   ▼   ▼      │
                                                                     NB Pred LR Pred DT Pred ◄┘
                                                                        │      │      │
                                                                        ▼      ▼      ▼
                                                                    Scorer  Scorer  Scorer
```

---

## Dica: Exportar o Workflow

Para compartilhar no GitHub:
1. Clique com botão direito no workflow no KNIME Explorer
2. `Export KNIME Workflow` → salve como `.knwf`
3. Adicione o arquivo `.knwf` ao repositório
