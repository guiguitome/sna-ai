# 📊 Detecção de Risco Sistêmico no Mercado de Fundos (CVM) via Machine Learning e Teoria dos Grafos

Este projeto desenvolve uma pipeline de **Engenharia de Dados e Machine Learning (Aprendizado Não-Supervisionado)** para monitorar e detectar anomalias comportamentais no mercado brasileiro de fundos de investimento (dados da CVM). 

Ao invés de analisar fotografias estáticas do mercado, o modelo avalia o comportamento temporal (*Strategy Drift*) dos ativos ao longo de 12 meses (Ago/2025 a Jul/2026), cruzando topologia de redes com um comitê de algoritmos de detecção de *outliers*.

## 🎯 Objetivos Analíticos

O monitoramento centralizado da rede foi dividido em duas esteiras independentes de risco sistêmico:

1.  **Efeito Manada (In-Degree Centrality):** Identifica fundos que atraíram ou perderam capital de forma abrupta e anômala em relação ao mercado, agindo como potenciais "Cisnes Negros" de atração de liquidez.
2.  **Risco de Contágio (Out-Degree Centrality):** Mapeia distribuidores de capital (*Sources*) que alteraram drasticamente suas teses de alocação, gerando choques de liquidez de primeira ordem em fundos dependentes.

## ⚙️ Arquitetura e Pipeline de Dados

A base original de ~30.000 fundos ativos passou por um tratamento rigoroso para evitar ruídos estatísticos e o paradoxo de variância zero na geometria dos modelos espaciais:

*   **Bifurcação de Escopo:** Separação das matrizes de 12 meses em `df_manada` (apenas entradas) e `df_contagio` (apenas saídas).
*   **Limpeza de Matrizes Esparsas:** Remoção de nós isolados direcionalmente (*Target-only* e *Source-only*), reduzindo a base de Manada para 15.306 fundos e a de Contágio para 24.354 fundos. Isso garantiu a estabilização do cálculo de distância multivariável.

## 🤖 O Comitê de Decisão (*Ensemble Learning*)

Para evitar o alto índice de falsos positivos comum em matrizes financeiras altamente esparsas, foi implementado um sistema de **Votação por Maioria (Majority Vote >= 2)** utilizando três "juízes" com perspectivas matemáticas distintas:

*   **Isolation Forest (`contamination=0.01`):** Avaliação macro. Isola anomalias cortando o espaço de dados de forma hierárquica e lidando bem com mudanças temporais bruscas (*Strategy Drift*).
*   **Local Outlier Factor (LOF) (`n_neighbors=20`):** Avaliação micro. Foca na densidade local, detectando fundos que se descolaram exclusivamente do comportamento do seu próprio nicho/vizinhança.
*   **One-Class SVM (`nu=0.01`, `kernel='rbf'`, `gamma='scale'`):** Avaliação de fronteira. Desenha limites geométricos não-lineares ao redor do comportamento "normal". Atuou como ponte de validação, sendo calibrado pelo comitê para ignorar flutuações de baixa variância.

## 📈 Resultados e Validação

A aplicação da urna de apuração (consenso cruzado) filtrou os alarmes falsos de instabilidade topológica, revelando o quadro real de risco sistêmico do período:

*   **Fundos validados com Efeito Manada:** 54 alvos críticos.
*   **Fundos validados com Risco de Contágio:** 174 distribuidores anômalos.

O repositório inclui visualizações dessas apurações através de **Diagramas de Venn** (comprovando a eficiência da interseção do comitê) e gráficos de série temporal que ilustram o abismo matemático de um *Strategy Drift* comparado ao comportamento de um fundo estável.

## 🚀 Limitações Técnicas e Próximos Passos (Future Work)

O pipeline atual foca com sucesso na detecção de Risco de Contágio Direto (choques de liquidez de primeira ordem mapeados via *Out-Degree Centrality*). Para a evolução da arquitetura, planeja-se:

*   **Implementação de *Betweenness Centrality*:** Adição de métricas de centralidade de intermediação para mapear fundos que atuam como "pontes" entre *clusters* distintos (ex: varejo e fundos imobiliários). Isso permitirá ao modelo rastrear não apenas quem engatilha o choque financeiro, mas prever colapsos estruturais e fragmentação da rede em caso de quebra dessas pontes.
*   **Processamento Distribuído:** Escalar o cálculo de *Betweenness* (que exige roteamento completo entre todos os pares de nós) utilizando *clusters* com PySpark para suportar o processamento longitudinal sem gargalos de *hardware*.