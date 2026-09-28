# GovSpender: Auditoria Inteligente de Despesas Públicas

> Projeto Final de Conclusão do AI Talent Academy (Trilha Machine Learning) — WhiteCube

O GovSpender é um sistema inteligente voltado à fiscalização e auditoria de gastos públicos realizados via Cartão de Pagamento do Governo Federal (CPGF). Utilizando técnicas avançadas de Aprendizado de Máquina Não Supervisionado (Machine Learning), o projeto analisa transações corporativas para priorizar a atuação de auditores humanos na identificação de desvios e fraudes.

---

## O Problema e A Solução

* O Problema: A Lei da Transparência (LC nº 131/2009) determina a divulgação de despesas públicas. Contudo, devido ao massivo volume de dados do CPGF, a fiscalização manual por auditores torna-se uma tarefa lenta, custosa e pouco eficiente.
* A Solução: O GovSpender atua como um funil inteligente de auditoria[cite: 1]. Através do cruzamento entre Segmentação Comportamental e Detecção Estatística de Anomalias, a ferramenta reduz o ruído dos dados e gera um ranking priorizado com as transações e portadores de maior risco de fraude[cite: 1].

---

## Arquitetura de Dados e Governança (Medallion Architecture)

Para atender às diretrizes do DAMA-DMBOK, Privacy by Design, LGPD e ao Princípio da Impessoalidade (Art. 37, CF/88), o pipeline adota uma Arquitetura Medalhão estruturada em três camadas[cite: 1]:

+-----------------+       +-----------------+       +-----------------+
|  Camada Bronze  |  ---> |   Camada Silver |  ---> |   Camada Gold   |
|   (Dados Brutos)|       |(Limpeza & LGPD) |       |(Feature Engin.) |
+-----------------+       +-----------------+       +-----------------+

1. Camada Bronze (Raw): Armazenamento do acervo bruto extraído diretamente do Portal da Transparência (21.891 registros e 15 colunas)[cite: 1].
2. Camada Silver (Clean & Privacy): 
   * Tratamento de valores nulos e conversão de tipos de dados (moeda e datas)[cite: 1].
   * Isolamento de transações sob sigilo legal para auditoria segregada[cite: 1].
   * Privacy by Design: Anonimização/pseudonimização via hashing criptográfico (SHA-256) sobre CPFs e CNPJs de portadores e favorecidos para prevenção de vazamentos e remoção de vieses no modelo[cite: 1].
3. Camada Gold (Analytical):
   * Agrupamento por HASH_CPF_PORTADOR gerando métricas comportamentais (volume total, ticket médio, volatilidade, % de saques, % de compras e sigilo)[cite: 1].
   * Aplicação de transformação logarítmica (np.log1p) e escala robusta (RobustScaler)[cite: 1].

---

## Modelagem de Machine Learning

O pipeline combina dois modelos complementares não supervisionados para refinar o sinal de fraude[cite: 1]:

 Base de Dados (CPGF)
          |
          v
   Pré-processamento (Medalhão)
          |
 +--------+------------------------+
 v                                 v
1. Segmentação (K-Means)          2. Detecção (Isolation Forest)
   - Agrupa perfis de gasto          - Isola desvios atípicos
   - Define Cluster de Alto Risco    - Taxa de contaminação: 1%
 +--------+------------------------+
          |
          v
 Cruzamento Lógico (AND) ---> Top 10 Transações Suspeitas

1. Segmentação Comportamental (K-Means):
   * Divide os portadores em 3 perfis de consumo: Gasta Pouco (0), Gastos Moderados (1) e Gasta Muito / Alto Consumo (2)[cite: 1].
   * Hiperparâmetros validados pelos métodos do Cotovelo (Elbow Method) e Score de Silhueta[cite: 1].
2. Detecção de Anomalias (Isolation Forest):
   * Isola portadores com comportamentos estatisticamente distantes do padrão[cite: 1].
   * Ajustado com taxa de contaminação de 1%, mapeando o início da cauda longa de risco atípico[cite: 1].
3. Cruzamento de Resultados:
   * A intersecção lógica (Cluster K-Means == 2 AND Alerta Isolation Forest == True) elimina falsos positivos[cite: 1].
   * Resultado: Redução de 2.810 portadores para 11 casos de altíssima prioridade (0,39% da base)[cite: 1].

---

## Estrutura do Repositório

```text
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
```[cite: 1]

---

## Como Executar o Projeto

1. Clone este repositório:
   ```bash
   git clone [https://github.com/joaohppenha/govspendertest.git](https://github.com/joaohppenha/govspendertest.git)
   cd govspendertest
