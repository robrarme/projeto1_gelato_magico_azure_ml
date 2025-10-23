## Projeto 1: Gelato Mágico no Azure ML Studio


<br>


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Projeto de *machine learning* para prever vendas de sorvete conforme a temperatura do dia mediante regressão preditiva, usando o ML Studio da Azure.



&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	O objetivo é treinar um modelo usando o Automated ML e um *pipeline* de componentes, registrá-lo e gerenciá-lo usando o MLFLow e implementá-lo para previsões em tempo real num ambiente de computação em nuvem.


<br>


#### 1- OBTENÇÃO DOS DADOS




&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Obtive os dados num *dataset* do Kraggle: https://www.kaggle.com/datasets/sakshisatre/ice-cream-sales-dataset 
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	[Figura 1.1](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%201.1%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Kaggle%20-%20Ice-Cream%20Sales%20Dataset.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Escolhi-os por causa do tamanho da amostra, 500 observações, supostamente cada observação equivale a um dia observável, a temperatura média e o correspondente valor de venda diários.

<br>

#### 2- CRIAÇÃO DE UMA INSTÂNCIA DE COMPUTAÇÃO E CRIAÇÃO DE UM CLUSTER DE COMPUTAÇÃO

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	No AzureML, naveguei até a seção "Computação" e selecionei "Instâncias de computação". Após definir parâmetros como nome, tipo de máquina virtual e autenticação, a instância é provisionada e pode ser iniciada sob demanda ([Figura 2.1](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%202.1%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Compute%20-%20Inst%C3%A2ncia%20de%20computa%C3%A7%C3%A3o.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Criei o *cluster* na mesma seção "Computação", na aba "Clusters de computação", na qual defini o nome, o tipo de VM, o número mínimo e máximo de nós e o tempo de inatividade ([Figura 2.2](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%202.2%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Compute%20-%20Cluster%20de%20computa%C3%A7%C3%A3o.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Assim, foi possível garantir controle de recursos durante a fase de desenvolvimento e o treinamento dos trabalhos (*jobs*).

<br>

#### 3- CRIAÇÃO DE UM *DATASET* TABULAR 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Acessei a seção "Dados" e selecionei "Criar dataset". Em seguida, escolhi a opção "Tabular", define a origem dos dados (como um armazenamento do Azure ou *upload* local), configurei o esquema de colunas e tipos de dados, e finalizei o processo com a validação e registro do *dataset* ([Figura 3.1](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%203.1%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Data%20-%20Perfil%20dos%20dados.jpg)).

<br>

#### 4- EXECUÇÃO DO AUTOMATED ML



&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Para iniciar, acessei a seção "Automated ML", selecionei um *dataset* tabular previamente registrado, define a variável-alvo, escolhe o tipo de tarefa e configura o ambiente de execução ([Figura 4.1](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%204.1%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Automated%20ML%20-%20Cria%C3%A7%C3%A3o%20de%20um%20trabalho%20em%20AutoML.jpeg)).

- Alvo: Vendas
- Métrica primária: NormalizedRootMeanSquareError
- Modelo de regressão: foi selecionado apenas o XGBoostRegressor, excluídos os demais, a fim de executar mais rapidamente e economizar recursos de minha conta na Azure
- _Timeout_ de experimento: 15 minutos
- _Timeout_ de iteração: 15 minutos
- "Early termination" habilitado
- Tipo de validação: automático
- Dados de teste: automático
- Ambiente de execução: *cluster* de computação, até 3 nós e 0 nós para economizar recursos.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	O sistema então testa múltiplos algoritmos e hiperparâmetros, retornando o modelo com melhor desempenho baseado em métricas específicas 
([Figura 4.2](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%204.2%20-%20Projeto1-%20Gelato%20M%C3%A1gico%20-%20Jobs%20-%20Automated%20ML%20-%20All%20jobs.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	O algoritmo mais preciso (menor erro) foi o "VotingEnsemble", embora tenha durado mais de 3 minutos, aproximadamente 6x mais do que a média da maioria dos modelos ([Figura 4.3](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%204.3%20-%20Projeto1-%20Gelato%20M%C3%A1gico%20-%20Jobs%20-%20Automated%20ML%20-%20Models%20%26%20Child%20Jobs%20-%20Melhor%20modelo.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	A correlação ficou em 98%, e o R2 e a variância explicada ficou em 97%, indicando uma alta de precisão do modelo ([Figura 4.4](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%204.4%20-%20Projeto1-%20Gelato%20M%C3%A1gico%20-%20Jobs%20-%20Automated%20ML%20-%20Overview%20-%20Melhor%20modelo%20-%20Other%20Metrics.jpg)).

- Variância explicada (parâmetro EXPLAINED_VARIANCE): 97,4%
- Coeficiente de determinação (parâmetro R2_SCORE): 97,4%
- Coeficiente de correlação de Spearman (parâmetro SPEARMAN_CORRELATION): 98,4%

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Os gráficos de valores previstos e realizados ("Predicted vs. True") estão bem aderentes um ao outro ([Figura 4.5](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%204.5%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Jobs%20-%20Automated%20ML%20-%20Metrics%20-%20Predicted%20x%20True%20%26%20Histogram.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Os mecanismos de controle de qualidade dos dados (*data guardrails*) foram realizados com sucesso: os dados de validação foram separados, valores faltantes e alta cardinalidade não foram detectados ([Figura 4.6](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%204.6%20-%20Projeto1-%20Gelato%20M%C3%A1gico%20-%20Jobs%20-%20Automated%20ML%20-%20Data%20guardrails.jpg)).

<br>

#### 5- DESENVOLVIMENTO DE *PIPELINE* NO DESIGNER

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Para criar um *pipeline* de ML, acessei a seção "Designer" no Azure ML Studio e iniciei um novo projeto. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Selecionei os módulos de entrada de dados, transformação, algoritmos de aprendizado e métricas de avaliação, conectando-os numa seqüência lógica. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Após configurar os parâmetros e vincular o *pipeline* a um *cluster* de computação, executei o fluxo e acompanhar os resultados em tempo real. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Seqüência dos componentes do *pipeline*:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		Database >> Select Columns >> Split Data >> Train Model >> Score Model >> Evaluate Model

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Parâmetros usados dos componentes:

- "Select Colums": colunas "Temperatura" e "Vendas"
- "Split Data": "Fraction of rows in the first output dataset" = 0,2
- "Untrained model" = "Linear Regression" ([Figura 5.1](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%205.1%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Designer%20-%20Configura%C3%A7%C3%A3o%20da%20Regress%C3%A3o%20Linear.jpg))
- "Train Model": "Model Explanations" = false (para não onerar mais recursos) ([Figura 5.2](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%205.2%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Designer%20-%20Configura%C3%A7%C3%A3o%20do%20componente%20Train%20Model.jpg))

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Configurei o trabalho (*job*) do *pipeline* ([Figura 5.3](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%205.3%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Designer%20-%20Configure%20%26%20Submit%20-%20Trabalho%20(job)%20do%20pipeline%20-%20Runtime%20Settings.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Por fim, o *pipeline* foi:
- criado ([Figura 5.4](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%205.4%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Designer%20-%20Pipeline.jpg)), 
- executado ([Figura 5.5](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%205.5%20-%20Projeto1%20-%20Projeto1-%20Gelato%20M%C3%A1gico%20-%20Jobs%20-%20Designer%20-%20Experimento.jpg))
- e foram apresentados seus resultados ([Figura 5.6](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%205.6%20-%20Projeto1%20-%20Projeto1-%20Gelato%20M%C3%A1gico%20-%20Jobs%20-%20Designer%20-%20Score%20Model%20component%20-%20Scored%20Labels.jpg)).

<br>

#### 6- DESENVOLVIMENTO NO MLFLOW

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Gerei o *scripts* e o *notebook* do melhor modelo: algoritmo "VotingEnsemble" ([Figura 6.1](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%206.1%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20MLFLow%20-%20Notebook%20Gerado.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Executei os comandos do *notebook* ([Figura 6.2](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%206.2%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20MLFLow%20-%20Jobs%20-%20Trabalho%20(job)%20Executado.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Registrei automaticamente o modelo ([Figura 6.3](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%206.3%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20MLFLow%20-%20Registro%20autom%C3%A1tico%20do%20modelo.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Gerei as métricas correspondentes ([Figura 6.4](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%206.4%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20MLFLow%20-%20Gera%C3%A7%C3%A3o%20das%20m%C3%A9tricas%20do%20modelo.jpg), 
 [Figura 6.5](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%206.5%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20MLFLow%20-%20M%C3%A9tricas%20do%20modelo%20geradas.jpg)).

<br>

#### 7- IMPLANTAÇÃO DO MODELO

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Efetuei a implantação de um *endpoint* para previsões em tempo real em ambiente de nuvem, definindo o nome, a máquina virtual e habilitando a opção para gravar a coleção de dados de inferência, para aprimorar o modelo e monitorar o desvio de dados ([Figura 7.1](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%207.1%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20Jobs%20-%20Automated%20ML%20-%20Melhor%20modelo%20-%20Model%20-%20Deploy.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	*Endpoint* criado ([Figura 7.2](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%207.2%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20MLFLow%20-%20Models%20-%20Endpoint%20criado.jpg)).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Lista de *endpoints* criados ([Figura 7.3](https://github.com/robrarme/projeto1_gelato_magico_azure_ml/blob/main/Figura%207.3%20-%20Projeto1%20-%20Gelato%20M%C3%A1gico%20-%20MLFLow%20-%20Models%20-%20Lista%20dos%20Endpoints.jpg)).

<br>

#### 8- COMPREENSÃO DO ESTUDO DE CASO E POSSIBILIDADES

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Ante a variável Temperatura, observa-se que o gráfico de Volume de Vendas é praticamente uma reta ascendente.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Para um vendedor de sorvetes, seria interessante aprofundar os estudos acrescentando mais variáveis, por exemplo dias da semana, para saber se há aumento de vendas durante os fins-de-semana, ou se há uma queda de vendas num dia chuvoso, embora quente. Nestes casos, podem ser testados outros algoritmos para verificar se há uma maior aderência ao modelo reformulado. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Ademais, em regiões muito quentes, seria bom testar se há um ponto de reversão no gráfico, uma vez que a população talvez evite se expor ao sol graças ao desconforto térmico, assim como em regiões frias descobrir o ponto em que não valha a pena venderou se mantenha uma estrutura mínima possível do estabelecimento.
