# GovSpender: Auditoria Inteligente de Despesas Públicas

> Projeto Final de Conclusão do AI Talent Academy (Trilha Machine Learning) — WhiteCube

O GovSpender é um sistema inteligente voltado à fiscalização e auditoria de gastos públicos realizados via Cartão de Pagamento do Governo Federal (CPGF). Utilizando técnicas avançadas de Aprendizado de Máquina Não Supervisionado (Machine Learning), o projeto analisa transações corporativas para priorizar a atuação de auditores humanos na identificação de desvios e fraudes.

---

## O Problema e A Solução

* **O Problema:** A Lei da Transparência (LC nº 131/2009) determina a divulgação de despesas públicas. Contudo, devido ao massivo volume de dados do CPGF, a fiscalização manual por auditores torna-se uma tarefa lenta, custosa e pouco eficiente.
* **A Solução:** O GovSpender atua como um funil inteligente de auditoria. Através do cruzamento entre Segmentação Comportamental e Detecção Estatística de Anomalias, a ferramenta reduz o ruído dos dados e gera um ranking priorizado com as transações e portadores de maior risco de fraude.

---

## Arquitetura de Dados e Governança (Medallion Architecture)

Para atender às diretrizes do DAMA-DMBOK, Privacy by Design, LGPD e ao Princípio da Impessoalidade (Art. 37, CF/88), o pipeline adota uma Arquitetura Medalhão estruturada em três camadas:

```
+-----------------+       +-----------------+       +-----------------+
|  Camada Bronze  |  ---> |   Camada Silver |  ---> |   Camada Gold   |
|  (Dados Brutos) |       |(Limpeza & LGPD) |       |(Feature Engin.) |
+-----------------+       +-----------------+       +-----------------+
```

1. **Camada Bronze (Raw):** Armazenamento do acervo bruto extraído diretamente do Portal da Transparência (21.891 registros e 15 colunas).
2. **Camada Silver (Clean & Privacy):**
   * Tratamento de valores nulos e conversão de tipos de dados (moeda e datas).
   * Isolamento de transações sob sigilo legal para auditoria segregada.
   * **Privacy by Design:** Anonimização/pseudonimização via hashing criptográfico (SHA-256) sobre CPFs e CNPJs de portadores e favorecidos para prevenção de vazamentos e remoção de vieses no modelo.
3. **Camada Gold (Analytical):**
   * Agrupamento por `HASH_CPF_PORTADOR` gerando métricas comportamentais (volume total, ticket médio, volatilidade, % de saques, % de compras e sigilo).
   * Aplicação de transformação logarítmica (`np.log1p`) e escala robusta (`RobustScaler`).

---

## Modelagem de Machine Learning

O pipeline combina dois modelos complementares não supervisionados para refinar o sinal de fraude:

```
                 Base de Dados (CPGF)
                          |
                          v
             Pré-processamento (Medalhão)
                          |
      +-------------------+-------------------+
      |                                       |
      v                                       v
1. Segmentação (K-Means)            2. Detecção (Isolation Forest)
   - Agrupa perfis de gasto            - Isola desvios atípicos
   - Define Cluster de Alto Risco      - Taxa de contaminação: 1%
      |                                       |
      +-------------------+-------------------+
                          |
                          v
            Cruzamento Lógico (AND) 
                          |
                          v
            Top 10 Transações Suspeitas
```

1. **Segmentação Comportamental (K-Means):**
   * Divide os portadores em 3 perfis de consumo: Gasta Pouco (0), Gastos Moderados (1) e Gasta Muito / Alto Consumo (2).
   * Hiperparâmetros validados pelos métodos do Cotovelo (Elbow Method) e Score de Silhueta.
2. **Detecção de Anomalias (Isolation Forest):**
   * Isola portadores com comportamentos estatisticamente distantes do padrão.
   * Ajustado com taxa de contaminação de 1%, mapeando o início da cauda longa de risco atípico.
3. **Cruzamento de Resultados:**
   * A intersecção lógica (`Cluster K-Means == 2` AND `Alerta Isolation Forest == True`) elimina falsos positivos.
   * **Resultado:** Redução de 2.810 portadores para 11 casos de altíssima prioridade (0,39% da base).

---

## Estrutura do Repositório

```
.
├── Documentacao/
│   ├── 1. Dicionario_de_Dados_Camada_Bronze.pdf
│   ├── 2. Dicionario_de_Dados_Camada_Silver.pdf
│   ├── 3. Dicionario_de_Dados_Camada_Gold.pdf
│   ├── 4. Relatorio_de_Arquitetura.pdf
│   └── relatorio_transacoes_suspeitas.pdf
├── bronze/
│   ├── camada_bronze.csv
│   └── camada_bronze.parquet
├── silver/
│   └── camada_silver.parquet
├── gold/
│   └── camada_gold.parquet
├── notebooks/
│   └── pipeline_govspender.ipynb
├── requirements.txt
└── README.md
```

---

## Como Executar o Projeto

1. **Clone este repositório:**
   ```bash
   git clone [https://github.com/joaohppenha/govspendertest.git](https://github.com/joaohppenha/govspendertest.git)
   cd govspendertest
   ```

2. **Crie e ative um ambiente virtual:**
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Linux/macOS
   # ou: .venv\Scripts\activate  # Windows
   ```

3. **Instale as dependências:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Execute os Notebooks:**
   Navegue até a pasta `notebooks/` e execute o fluxo em ordem para rodar o pipeline de ETL, treinamento dos modelos e geração automática do relatório PDF.

---

## Relatório Final e Human-in-the-Loop

O sistema gera automaticamente um Relatório Executivo em PDF contendo a lista dos casos mais críticos.

> **Aviso Importante:** Os resultados do GovSpender representam indicativos e alertas analíticos, cabendo estritamente ao auditor humano a tomada de decisão e abertura dos procedimentos investigativos cabíveis (Human-in-the-Loop).

---

## Autor

Desenvolvido por João Penha como projeto de encerramento do AI Talent Academy — WhiteCube.
