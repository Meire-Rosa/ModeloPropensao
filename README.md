# Modelo de Propensão à Contratação em CRM Bancário

## 1.Contexto e objetivo de negócio
Este projeto foi desenvolvido no contexto de um desafio para a posição de Pesquisador de Dados em CRM bancário.
O objetivo é construir um modelo de propensão capaz de estimar a probabilidade de um cliente aceitar uma oferta bancária, utilizando informações sobre seu perfil e histórico de relacionamento, de modo a apoiar a priorização de campanhas comerciais.
Para este estudo, foi utilizado um conjunto de dados de campanhas de telemarketing bancário, cujo objetivo comercial era a contratação de um depósito a prazo (term deposit).


### Pergunta de negócio

Como estimar, antes do contato comercial, a probabilidade de um cliente contratar um depósito a prazo, utilizando informações disponíveis sobre seu perfil e histórico de relacionamento, para apoiar a priorização das campanhas de CRM?


A proposta é ordenar os registros conforme a propensão estimada e avaliar a capacidade do modelo de concentrar contratações observadas em uma lista de tamanho limitado.
A seleção de 2.000 registros foi utilizada como cenário experimental de capacidade de campanha. A definição de um limite operacional real dependeria de informações adicionais, como custo por contato, retorno financeiro esperado e capacidade da equipe comercial.

## 2. Base de dados
### 2.1. Fonte
Foi utilizado o conjunto de dados Bank Marketing, associado ao artigo A data-driven approach to predict the success of bank telemarketing.

•	Fonte: [UCI Machine Learning Repository — Bank Marketing](https://archive.ics.uci.edu/dataset/222/bank+marketing)

•	Origem: campanhas de marketing direto de uma instituição bancária portuguesa.

•	Período analisado: maio de 2008 a novembro de 2010.

•	Quantidade de registros: 45.211.

•	Variáveis: 16 variáveis preditoras e uma variável-alvo.

•	Tipo de problema: classificação binária.

Os clientes foram contatados principalmente por telefone para a oferta de um depósito a prazo.

A variável-alvo y indica se houve contratação (yes) ou não (no).

### 2.2 Dicionário de Dados 

<img width="574" height="585" alt="image" src="https://github.com/user-attachments/assets/3f7ca69a-9d2d-4b74-bd74-0bd76404b140" />

### 2.3. Considerações sobre a qualidade dos dados
Na análise inicial, a base foi considerada estruturada para análise, sem identificação de valores nulos ou duplicatas na verificação realizada.

Ainda assim, a ausência de nulos e duplicatas não garante, por si só, a ausência de inconsistências semânticas, valores extremos ou vieses. Essas questões devem ser consideradas ao longo da análise exploratória e da preparação dos dados.

A base não possui um identificador explícito de cliente. Para rastrear os registros durante o experimento, foi criado o campo técnico id_registro. Esse identificador não representa um código real de cliente bancário.

## 3. Metodologia

O trabalho foi organizado em cinco etapas:

E1.	Definição da pergunta de negócio.

E2.	Análise exploratória dos dados (EDA).

E3.	Seleção e preparação das variáveis.

E4.	Treinamento e avaliação dos modelos.

E5.	Interpretação dos resultados e implicações para o CRM.

## 4. Etapa 1 — Definição do problema

A variável-alvo escolhida foi y, que representa a contratação do depósito a prazo.

Para a modelagem, a variável foi codificada da seguinte forma:

•	1: houve contratação;

•	0: não houve contratação.

A tarefa foi formulada como um problema de classificação binária, com estimativa de probabilidade para cada registro.

O propósito não é apenas classificar os clientes em duas categorias, mas também ordenar os registros por propensão estimada, permitindo avaliar diferentes estratégias de priorização de campanhas.

## 5. Etapa 2 — Análise exploratória dos dados (EDA)

### 5.1. Perfil demográfico e profissional

A base contém 45.211 registros, com idades entre 18 e 95 anos. Foram identificados 487 registros de pessoas com mais de 70 anos, aproximadamente 1,07% da base.

As ocupações estão distribuídas em 12 categorias. As três categorias mais frequentes são:

•	Operários (blue-collar): 21,53%;

•	Gerentes (management): 20,92%;

•	Técnicos (technician): 16,80%.

Em conjunto, representam aproximadamente 59,25% dos registros.

Quanto à escolaridade, 51,32% possuem ensino médio (secondary) e 29,42% possuem ensino superior (tertiary). Os demais registros correspondem a clientes com ensino fundamental ou escolaridade desconhecida.

Em relação ao estado civil, a distribuição inicial foi:

•	Casados: 60,2%;

•	Solteiros: 28,3%;

•	Divorciados ou viúvos: 11,5%, conforme a categorização adotada pela base.

### 5.2. Características financeiras

Na análise inicial, observou-se que:

•	55,6% dos registros indicam empréstimo habitacional;

•	16% indicam empréstimo pessoal;

•	1,8% indicam inadimplência de crédito.

Essas variáveis ajudam a caracterizar o conjunto de dados, mas não devem ser interpretadas isoladamente como medidas completas da capacidade de pagamento ou da intenção de contratação.

### 5.3. Distribuição da variável-alvo

Aproximadamente 11,7% dos registros correspondem a clientes que contrataram o depósito a prazo.

A predominância de respostas negativas caracteriza um problema de classes desbalanceadas. Nesse cenário, a acurácia isolada pode ser enganosa: um modelo que classifique quase todos os registros como não contratantes pode apresentar acurácia elevada sem ser útil para identificar oportunidades comerciais.

Por isso, foram consideradas métricas como AUC-ROC, AUC-PR, precisão e recall, além da avaliação de uma lista de tamanho fixo.

## 6. Etapa 3 — Seleção e preparação das variáveis

### 6.1. Estratégia de seleção

O modelo inicial priorizou variáveis de perfil e relacionamento que poderiam estar disponíveis antes do contato comercial.

<img width="514" height="605" alt="image" src="https://github.com/user-attachments/assets/51d7cc7e-f1c8-4911-aba4-3535b679be8c" />

### 6.2. Codificação das variáveis

As variáveis binárias default, housing, loan e y foram transformadas em valores numéricos:

•	yes → 1;

•	no → 0.

As variáveis categóricas job, marital, education e contact foram processadas utilizando componentes do PySpark ML:

•	StringIndexer: transformação das categorias em índices;

•	OneHotEncoder: representação das categorias em vetores binários.

O pipeline de preparação foi aplicado às variáveis categóricas antes da montagem do vetor de características.

A classe VectorAssembler foi utilizada para reunir as variáveis selecionadas em uma coluna features, exigida pelos algoritmos de classificação do PySpark ML.

O vetor resultante apresentou 36 posições no experimento com o identificador técnico preservado. O identificador id_registro foi mantido separadamente e não foi utilizado como variável preditora.

### 6.3. Identificador técnico

Como a base original não disponibiliza um identificador único de cliente, foi criada a coluna id_registro utilizando monotonically_increasing_id().

Esse campo foi utilizado para rastrear os registros selecionados e recuperar suas características originais durante a análise dos resultados.

O identificador é exclusivamente técnico e não deve ser interpretado como uma identificação real de cliente. Ele também não deve ser utilizado como variável de entrada do modelo.

## 7. Etapa 4 — Modelagem

### 7.1. Divisão dos dados
Os dados foram divididos em conjuntos de treinamento e teste, utilizando a função randomSplit do PySpark:

•	80% para treinamento;

•	20% para teste;

•	semente aleatória: 42.

A divisão permite treinar os modelos em uma parte dos dados e avaliar seu desempenho em registros não utilizados diretamente no ajuste dos parâmetros.

Para uma versão de produção, recomenda-se avaliar também uma divisão estratificada e uma validação temporal, considerando que os registros representam campanhas realizadas em diferentes períodos.

### 7.2. Modelos selecionados
Foram avaliados dois algoritmos de classificação.

#### 7.2.1 Regressão Logística

Modelo probabilístico utilizado como primeira abordagem para estimar a propensão à contratação.

Configuração utilizada:

•	featuresCol="features";

•	labelCol="y";

•	maxIter=100.
#### 7.2.2 Árvore de Decisão

Modelo baseado em regras de divisão dos dados, útil para explorar relações não lineares e facilitar a interpretação de segmentos.
Configuração utilizada:

•	featuresCol="features";

•	labelCol="y";

•	maxDepth=5;

•	seed=42.

Os modelos foram treinados novamente após a inclusão do identificador técnico no fluxo de preparação, preservando a correspondência entre os registros de teste e suas previsões.


### 7.3. Estratégia de avaliação
Além das métricas tradicionais de classificação, foi utilizada uma estratégia orientada ao objetivo de negócio: ordenar os registros pela probabilidade estimada de contratação e selecionar os 2.000 primeiros.

Para essa avaliação:

•	a probabilidade de contratação corresponde à probabilidade estimada da classe 1;

•	a precisão da seleção é a proporção de registros selecionados que efetivamente contrataram o produto;

•	o tamanho da lista foi mantido igual para os dois modelos.

A precisão da seleção pode ser expressa por:

[
\text{Precisão@2000} =
\frac{\text{Contratações observadas entre os 2.000 selecionados}}{2000}
]

Essa métrica ajuda a responder quantas contratações foram observadas entre os registros priorizados, mas não mede, por si só, o efeito causal da campanha.

### 7.4. Resultados do experimento atual

<img width="516" height="126" alt="image" src="https://github.com/user-attachments/assets/ac97004c-abff-457b-ae60-c89e5902849e" />

A regressão logística apresentou o melhor resultado nesse experimento.

Entre os 2.000 registros selecionados por esse modelo, 505 correspondiam a contratações observadas, enquanto a árvore de decisão identificou 426 contratações no grupo selecionado.

A diferença foi de 79 contratações observadas, equivalente a 3,95 pontos percentuais de precisão.

Em termos relativos, a quantidade de contratações observadas entre os selecionados pela regressão logística foi aproximadamente 18,5% maior que a da árvore de decisão.

Interpretação: para a mesma quantidade de registros priorizados, a regressão logística apresentou maior concentração de contratações históricas no conjunto de teste.

Esse resultado favorece a regressão logística como candidata para as próximas etapas de avaliação. Contudo, ainda não é suficiente para concluir que ela produzirá maior retorno financeiro ou melhor resultado em campanhas futuras.

## 8. Etapa 5 — Interpretação dos resultados para CRM

### 8.1. Perfil dos 2.000 registros selecionados pela regressão logística
A análise exploratória da lista selecionada identificou a seguinte distribuição profissional:

<img width="515" height="280" alt="image" src="https://github.com/user-attachments/assets/535c7738-c2bb-4814-bbc5-f3df4d394104" />

As cinco categorias profissionais mais frequentes representam 81,15% dos registros selecionados.

Quanto à escolaridade:

<img width="537" height="123" alt="image" src="https://github.com/user-attachments/assets/50f0aebf-5e9e-4e96-a005-3a8c6a1f93ed" />

Quanto ao estado civil: 

<img width="533" height="106" alt="image" src="https://github.com/user-attachments/assets/5ca31caf-f559-4a74-b491-1670583d8a80" />

### 8.2. Interpretação inicial
A seleção apresenta maior frequência de registros classificados como management, seguida por retired, technician e admin.. 

A escolaridade terciária corresponde a pouco mais da metade da seleção, enquanto os registros de pessoas solteiras representam a categoria mais frequente de estado civil.

Esses resultados descrevem a composição da lista selecionada pelo modelo. Não demonstram, isoladamente, que essas categorias apresentam maior taxa de contratação.

Para identificar quais grupos apresentam maior propensão observada, é necessário comparar as taxas de contratação por categoria no conjunto de teste completo e na lista selecionada.

Essa análise comparativa é a próxima etapa da investigação.

### 8.3. Aplicações potenciais para o CRM
Caso os resultados sejam confirmados em validações adicionais, o modelo poderá apoiar:

•	priorização de registros em campanhas de capacidade limitada;

•	comparação de estratégias de seleção de público;

•	identificação de segmentos para análises comerciais complementares;

•	acompanhamento de métricas de contratação por campanha;

•	avaliação do equilíbrio entre volume de contatos e retorno esperado.

A utilização de características demográficas e financeiras em decisões comerciais requer avaliação de privacidade, conformidade regulatória, equidade e adequação à finalidade do produto. A presença de associação estatística não deve ser interpretada como justificativa automática para excluir grupos de clientes de uma oferta.

## 9. Tecnologias utilizadas

•	Python

•	Apache Spark / PySpark

•	PySpark ML

•	Regressão Logística

•	Árvore de Decisão

•	Git e GitHub para versionamento e documentação do projeto.

## 10. Conclusão
O estudo demonstrou a aplicação de técnicas de preparação de dados e classificação para um problema de propensão à contratação em CRM bancário.

Foram desenvolvidos dois modelos, Regressão Logística e Árvore de Decisão, e avaliada a capacidade de cada um de priorizar 2.000 registros do conjunto de teste.

No experimento atual, a regressão logística apresentou precisão de seleção de 25,25%, contra 21,30% da árvore de decisão. O resultado indica maior concentração de contratações históricas entre os registros priorizados pela regressão logística, para o mesmo tamanho de lista.

A análise do perfil dos registros selecionados também revelou a composição profissional, educacional e de estado civil desse grupo. Essa caracterização constitui um ponto de partida para investigações adicionais, mas precisa ser complementada por comparações de taxas de contratação entre grupos.

O próximo passo é concluir a análise comparativa dos perfis, consolidar as métricas e investigar a estabilidade dos resultados. Somente após essas etapas será possível avaliar com maior segurança a utilidade do modelo para apoiar decisões de priorização de campanhas de CRM.



