# Dynamic Graph Anomaly Detection in Financial Markets 📊🕸️

Este projeto consiste em um motor de ponta a ponta (End-to-End) de **Engenharia de Dados e Machine Learning** desenvolvido para identificar padrões anômalos e riscos sistêmicos em fundos de investimento no Brasil. 

Utilizando dados públicos da Comissão de Valores Mobiliários (CVM), o pipeline transforma milhões de registros estáticos de portfólios financeiros em **Redes Complexas Temporais (Grafos Direcionados)**, extraindo features topológicas para alimentar algoritmos não supervisionados de detecção de anomalias.

---

## 🎯 Objetivos do Projeto

O foco principal desta aplicação é monitorar o fluxo de capital e detectar preventivamente três grandes riscos de mercado:

*   **Risco de Concentração (Efeito Manada):** Identificar quando uma grande parcela do mercado passa a aportar capital massivamente em um mesmo ativo ou fundo simultaneamente.
*   **Risco de Contágio (Vulnerabilidade em Cascata):** Mapear a rede de Fundos de Investimento em Cotas (FICs) para encontrar "nós pontes" (fundos centrais) que, em caso de iliquidez, poderiam quebrar múltiplos outros fundos dependentes.
*   **Strategy Drift (Desvio de Estratégia):** Detectar mudanças bruscas e não padronizadas no comportamento de alocação de um fundo específico ao longo do tempo.

---

## ⚙️ Arquitetura e Etapas do Pipeline

O projeto está estruturado em 3 fases principais, garantindo escalabilidade e otimização de memória:

### 1. Ingestão e Engenharia de Dados (ETL)
*   **Extração:** Leitura em lote (`pathlib.glob`) do histórico mensal de composição de carteiras (Bloco 2) do portal de Dados Abertos da CVM.
*   **Transformação e Limpeza:** 
    *   Filtragem de colunas para reter apenas o núcleo da rede (`DT_COMPTC`, Source, Target e Peso Financeiro).
    *   Aplicação de threshold de materialidade financeira (ex: `>= R$ 100.000,00`) para expurgo de resíduos contábeis e ruídos.
*   **Consolidação:** Concatenação otimizada dos meses em um *DataFrame* temporal leve e estruturado.

### 2. Processamento Topológico (Teoria dos Grafos)
*   **Fatiamento Temporal:** Agrupamento iterativo dos dados por mês para evitar a sobreposição irreal do tempo (*Dynamic Graphs*).
*   **Construção da Rede:** Geração mensal de Grafos Direcionados (`nx.DiGraph`), onde:
    *   **Nós (Nodes):** Representam os Fundos de Investimento e as Cotas alvo.
    *   **Arestas (Edges):** Representam a direção do investimento (Source $\rightarrow$ Target).
    *   **Pesos (Weights):** Representam o Valor de Mercado da posição.
*   **Extração de Features:** Cálculo de métricas de centralidade (ex: *In-Degree Centrality*, *Betweenness Centrality*) para quantificar o grau de influência e popularidade de cada ativo na rede naquele mês específico.

### 3. Machine Learning (Detecção de Anomalias)
*   **Matriz de Features:** Transformação dos scores matemáticos do grafo em uma base tabular cronológica.
*   **Modelagem Não Supervisionada:** Aplicação de algoritmos (como *Isolation Forest*) para analisar a evolução temporal das centralidades e apontar *outliers* de comportamento.

---

## 🛠️ Stack Tecnológico

*   **Python:** Linguagem base do pipeline.
*   **Pandas:** Ingestão de arquivos pesados, limpeza e manipulação tabular de alta performance.
*   **NetworkX:** Construção das redes direcionadas e cálculo de matemática de grafos.
*   **Scikit-Learn:** Algoritmos de Machine Learning para a detecção das anomalias na etapa final.
