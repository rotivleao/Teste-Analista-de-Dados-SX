Critérios avaliadas:
- Docker;
- SQL;
- Python;
- Organização do Código
- Documentação
- ETL
- Modelagem dos dados

### Desejáveis
- PySpark
- Esquema Estrela


### Steps:

1. Realizar um Fork desse projeto
2. Realizar a modelagem dimensional da base
    - A base está disponível para download [clicando aqui](https://download.inep.gov.br/microdados/microdados_enem_2020.zip).
    - Após descompactar a paste, o Arquivo com a base encontra-se no diretório microdados_enem_2020/DADOS/MICRODADOS_ENEM_2020.csv
    - A documentação necessária sobre os campos da base está disponível nos demais diretórios dentro da pasta descompactada.
3. Realizar o ETL dessa base em Python para o MySQL no container
4. Disponibilizar o link do seu repositório para posterior avaliação


### Levantar Indicadores
#### Responder às seguintes perguntas:
1. Qual a escola com a maior média de notas?
Resposta: Privado
+----------------------+------------------+
|TP_DEPENDENCIA_ADM_ESC|     Average_Score|
+----------------------+------------------+
|                  null| 522.4011138732146|
|         Não respondeu|  522.407730440292|
|               Privada| 521.8136657875808|
|               Externa| 604.8483753579226|
|               Pública|502.31427247886506|
+----------------------+------------------+

2. Qual o aluno com a maior média de notas e o valor dessa média?
Resposta: 
Número de inscrição = 200005996961
Média = 858.58
+------------+-----------------+
|NU_INSCRICAO|Max_Average_Score|
+------------+-----------------+
|200005996961|858.5799999999999|
|200004812135|852.8599999999999|
|200001357436|           852.76|
|200004002694|            851.5|
|200001850224|851.3400000000001|
|200006051511|850.7400000000001|
|200003688741|849.3399999999999|
|200006619550|           847.82|
|200001993301|           847.68|
|200004146848|            847.6|
|200006712724|847.1200000000001|
|200005145247|            846.8|
|200004974610|           846.72|
|200001392605|            846.6|
|200003864472|846.5200000000001|
|200002272979|            845.8|
|200002467280|845.1200000000001|
|200001423299|            845.0|
|200001622477|            844.9|
|200005414259|            844.3|
+------------+-----------------+

3. Qual a média geral?
+-----------------+
|Average_Score_All|
+-----------------+
| 526.580680276944|
+-----------------+

4. Qual o % de Ausentes?
55%

+--------+-------+
|presenca|   QTDE|
+--------+-------+
|       1|2597440|
|       2|   1426|
|       0|3184243|
+--------+-------+

5. Qual o número total de Inscritos?
+-------+
|   QTDE|
+-------+
|5783109|
+-------+

6. Qual a média por disciplina?
**OBS:**Acredito que seja média de notas.
+-----------------+-----------------+-----------------+-----------------+-----------------+
|               CN|               CH|               LC|               MT|          REDACAO|
+-----------------+-----------------+-----------------+-----------------+-----------------+
|490.4097924879862|511.1522016309976|523.8009359364437|520.5783348219793|573.4127241171473|
+-----------------+-----------------+-----------------+-----------------+-----------------+

7. Qual a média por Sexo?
**OBS:**Acredito que seja média de notas, de qualquer forma, estou colocando o total de inscritos e as médias por nota.
+----+-------+   
|sexo|   QTDE|
+----+-------+
|   F|3468805|
|   M|2314304|
+----+-------+

+-------+-----------------+
|TP_SEXO|    Average_Score|
+-------+-----------------+
|      F|521.2449112081383|
|      M|534.7254885958833|
+-------+-----------------+

8. Qual a média por Etnia?
**OBS:**Acredito que seja média de notas, de qualquer forma, estou colocando o total de inscritos e as médias por nota.
+-----------------+-----------+
|      Etnia      |   QTDE    |
+--------------+--------------+
|          Branca | 2007633   |
|           Parda | 2720485   |
|        Indígena |  37846    |
|         Amarela | 128522    |
|           Preta | 771740    |
|   Não declarado | 116883    |
+-----------------+-----------+
 
+-----------------+--------------------+
|      Etnia      |   Average_Score    |
+--------------+-----------------------+
|          Branca | 556.526722139788   |
|           Parda | 508.65750169968993 |
|        Indígena | 472.84366774078063 |
|         Amarela | 524.9384386151195  |
|           Preta | 500.8683019963931  |
|   Não declarado | 534.0503395999999  |
+-----------------+-------------------+


### Levantar Visões
1. Gere visualizações gráficas que demonstrem a nota como indicador, trazendo as dimensões e os gráficos que melhor possam representar 
a informação para avaliação da performance.
2. Analisar correlações de variáveis que identificar dentro do dataset com a variável dependente nota total (NU_NOTA_CN
NU_NOTA_CH, NU_NOTA_LC, NU_NOTA_M.T).
3. Gerar visualizações (Data viz) que melhor estratifiquem e demonstremos dados do bloco de DADOS DA REDAÇÃO, verificando o comportamento
das notas 4 provas vs. estes dados.
4. Gerar visualizações (Data viz) que melhor estratifiquem e demonstremos dados do bloco de DADOS DO QUESTIONÁRIO SOCIOECONÔMICO, verificando
o comportamento das notas 4 provas vs. estes dados.
5. Faça um resumo em 10 bullets de Conclusões e Insights.

### sugestões
1. Tableau.
2. Power BI.
3. Qlik.
4. Power Point.
5. Excel.
6. Colab.


###Comentários Finais:
- Arquivo do colab está sendo compartilhado neste link -> https://colab.research.google.com/drive/1bAkp7qA5j0yH7XTq3mzpVVoyDNqjsm8g?usp=sharing
 
- Conexão com bando de dados:
  * Infelizmente não consegui conectar fazer com que os dados CSV da planilha MICRODADOS_ENEM_2020 fosse transferidos banda o banco de dados via **SPARK** devido ao um problem de conexão do drive JAR. Tentei fazer de tudo para corrigir mas não consegui.
tentei seguir para Mysql.connector, porém ele não suporta o volume da base.
  * Para não deixar de concluir o desafio eu criei uma amostragem com 10.000 linhas do arquivo MICRODADOS_ENEM_2020 e importei ele para o banco de dados na nuvem que criei pela **[RailWay](https://railway.app/)**.
Essa núvem eu criei 1 base fato e 6 dimensões para fazer o diagrama em formato de estrela solicitado.
Anexo da foto do diagrama aqui -> ![Diagrama Estrela](https://github.com/rotivleao/Teste-Analista-de-Dados-SX/assets/158123640/7dee4e9a-65cc-416a-a7e6-01ea6cc91188)
O Database encontra-se funcionando perfeitamente dentro RailWay e caso deseje acessar, segue abaixo a rota:
    host='roundhouse.proxy.rlwy.net',
    database='railway',
    port='17830',
    user='root',
    password='Ec-h6GBafBE335CDAdhDaebA6cb-F6Gg'
Seguindo as orienteções, tudo baseado em MySQL.

- Os pontos de **"Levantar Indicadores"** foram todos respondendido fazendo scripts através do python por spark ou sql.
- Os pontos de "Levantar Visões" eu não consegui fazer devido o problema com a migração da base pata DB. Eu ia fazer uma amostra desses dados seguindo a lágica que eu montei de 10.000 linhas mas meu prazo acabou.



