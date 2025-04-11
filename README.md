# 🎧 MVP - Engenharia de Dados | PUC-Rio

Este repositório contém o MVP desenvolvido para a disciplina de Engenharia de Dados da Pós-graduação Lato Sensu em Ciência de Dados e Analytics da PUC-Rio.  
O projeto tem como foco a análise das métricas da artista **Taylor Swift** no Spotify, com o objetivo de verificar a consistência, coerência e integridade dos dados disponibilizados publicamente, através das seguintes perguntas:

- A música com o maior número de streams é consistentemente a que alcançou a maior posição no ranking mundial?
- Poderia uma música com menos streams ter tido um pico maior devido a um momento específico ou viralização?
- Álbuns com maiores vendas possuem consistentemente músicas com maior número de streams em comparação com álbuns de menor venda física?
- A popularidade no streaming sempre se alinha com o sucesso de vendas físicas?
- Faixas com um alto número de views no YouTube também apresentam consistentemente um elevado número de views streams?
- A popularidade das plataformas de vídeo se traduz diretamente em engajamento de streaming?
- Há inconsistências na classificação de gênero em relação aos atributos musicais das faixas?
- Faixas marcadas como feat (com participação de outros artistas) apresentam um padrão consistente de aumento ou diminuição de stream ou ranking em comparação com suas músicas solo?
- A colaboração impacta as métricas de forma previsível?

---

## 🧪 Tecnologias e Ferramentas Utilizadas

- **Databricks Community Edition**
- **SQL (Spark SQL)**
- **Python (parcial)**
- **Spotify Web API** (documentação)
- **Spotipy** (para conexão com a API)

---

## 📂 1. Busca pelos Dados

### Fonte inicial

- Coleta realizada inicialmente através da **Spotify Web API**, via biblioteca **Spotipy**.
- Foram utilizados endpoints para coleta de álbuns, faixas, características acústicas e rankings da artista.

### Autenticação

- A autenticação foi realizada com as credenciais obtidas no [Spotify Developer Dashboard](https://developer.spotify.com/dashboard/applications).
- Redirecionamento configurado para `http://127.0.0.1:8888/callback`, conforme exigência da API.
- Fluxo utilizado: `Authorization Code Flow`.

![Configuração da API](https://github.com/user-attachments/assets/ae1a370c-c667-4dba-affa-0a2619dc2400)
![Exemplo de token](https://github.com/user-attachments/assets/b2eb93c6-bc39-44cb-8efc-485896849e84)

### Abandono da API e troca de fonte

- A coleta via API apresentou falhas contínuas por **timeout de resposta**, especialmente ao usar `sp.track(...)`.
- Erro comum: `ReadTimeoutError: HTTPSConnectionPool(...)`
- Por esse motivo, foi decidido seguir com uma base alternativa pública do **Kaggle**.

---

## 📥 2. Coleta

### Dataset alternativo

- **Fonte**: Kaggle  
- **Link direto**: [Taylor Swift Discography Dataset - Kaggle](https://www.kaggle.com/datasets/delfinaoliva/taylor-swift-discography?select=taylor_swift_discography.csv)
- **Formato**: `.csv`
- **Usuário**: Delfina Oliva
- **Origem dos dados**: Spotify Public API
- **Licença/API Terms**:  
  - [Spotify API Documentation](https://developer.spotify.com/documentation/web-api)  
  - [Spotify API Terms of Use](https://developer.spotify.com/terms)

---

## 🛠️ 3. Modelagem

- O dataset foi carregado inicialmente com todos os campos como `STRING` no Databricks.
- Após a ingestão, foi aplicada normalização de tipos de dados e criação de uma nova tabela refinada.
- **Modelo utilizado**: Flat table
- **Camada criada**: `hive_metastore.default.TB_METRICS_SPOTIFY_TAYLOR_SWIFT`

### 🧾 Catálogo de Dados Normalizados
![Catálogo de Dados](https://github.com/user-attachments/assets/4ff927fb-ca40-4198-85aa-06fa8e88c032)


---

## ⚙️ 4. Carga

### Etapas da ingestão e transformação

1. **Plataforma**: Databricks Community Edition
2. **Passos realizados**:
   - Acesso ao ambiente e criação de cluster.
   - Upload do `.csv` no DBFS.
   - Ingestão bruta via UI em `taylor_swift_discography_updated_csv`.
   - Criação de tabela refinada com casting correto dos tipos de dados.
   - Criação de notebook: `MVP - ENGENHARIA DE DADOS | PUC RIO`.
   - Nome da fonte computacional: `SPOTIFY PUBLIC DATASET | TAYLOR SWIFT`.

### Evidências visuais

- ![Tabela Refinada](https://github.com/user-attachments/assets/ad057064-cfaf-4f09-aaf4-4bfd4fcffce3)
- ![Upload CSV](https://github.com/user-attachments/assets/f8f6bead-ae4a-448b-addb-bf8b2a19bb9e)
- ![Create Table](https://github.com/user-attachments/assets/26fa0d96-267f-44ec-bff5-13d00de9ce58)
- ![Preview Table](https://github.com/user-attachments/assets/2255d109-0523-445d-8a49-5ee44145795f)
- ![Cluster Ativo](https://github.com/user-attachments/assets/0051c9c1-2e9f-40e7-b36e-530465345523)
- ![Catálogo no Databricks](https://github.com/user-attachments/assets/ac42838c-f1ea-4964-9f6a-b2de6588850f)
  


## 📊 5. Análise

### a. Qualidade dos Dados

Durante a análise da base original e a ingestão no Databricks, foram observados os seguintes pontos:

- **Ausência de valores nulos em colunas-chave**, como `track_name`, `spotify_streams` e `spotify_global_peak`.
- **Inconsistência entre colunas relacionadas**, como `album_physical_sales` e `spotify_streams`, onde os registros estavam majoritariamente incompletos ou desbalanceados.
- Todos os campos foram inicialmente lidos como `string`, o que exigiu transformação manual para tipos apropriados (`int`, `float`, `date`, etc.).
- **Problemas de normalização** nos gêneros musicais, com variações de grafia e espaçamento, foram tratados com funções de `regexp_replace`.

---

### b. Solução do Problema

As análises foram realizadas utilizando **Spark SQL** no Databricks, baseando-se na tabela refinada `TB_METRICS_SPOTIFY_TAYLOR_SWIFT`.

---

#### 📌 Pergunta 1:
**A música com o maior número de streams é consistentemente a que alcançou a maior posição no ranking mundial?**  
✅ **Sim**. A música com maior número de streams também é a que possui melhor posição no ranking (mais próxima de 1).
![image](https://github.com/user-attachments/assets/dc492205-4374-4fb2-9299-1b9b59ae8707)

---

#### 📌 Pergunta 2:
**Poderia uma música com menos streams ter tido um pico maior devido a um momento específico ou viralização?**  
✅ **Sim**. Há casos em que músicas com menos streams atingiram melhores picos de ranking, possivelmente por eventos virais, lançamentos de clipe, ou repercussões momentâneas.
![image](https://github.com/user-attachments/assets/5acff1db-eb42-46e0-9309-7267fdfaf3d3)

---

#### 📌 Pergunta 3:
**Álbuns com maiores vendas possuem consistentemente músicas com maior número de streams em comparação com álbuns de menor venda física?**  
⚠️ **Não foi possível afirmar**. A base apresenta ausência de dados cruzados entre `album_physical_sales` e `spotify_streams` – ou há um, ou outro, dificultando comparações diretas. Isso pode indicar uma estratégia deliberada de distribuição segmentada.
![image](https://github.com/user-attachments/assets/2b0f630a-65d3-4045-9194-ec44069f07f2)
![image](https://github.com/user-attachments/assets/1ae253e0-267f-4779-af4e-bd3780da6e94)

---

#### 📌 Pergunta 4:
**A popularidade no streaming sempre se alinha com o sucesso de vendas físicas?**  
⚠️ **Também não**. Novamente, a ausência de registros simultâneos de vendas e streams impede correlações diretas.
![image](https://github.com/user-attachments/assets/2b0f630a-65d3-4045-9194-ec44069f07f2)
![image](https://github.com/user-attachments/assets/1ae253e0-267f-4779-af4e-bd3780da6e94)
---

#### 📌 Pergunta 5:
**Faixas com um alto número de views no YouTube também apresentam consistentemente um elevado número de streams?**  
🔎 **Não há dados quantitativos diretos de YouTube views**, apenas um campo binário (`track_videoclip`). Contudo, as faixas com videoclipe tendem a ter mais streams, sugerindo uma possível correlação indireta.
![image](https://github.com/user-attachments/assets/d4fcaa47-e257-4b83-8a99-737bfbb7cc07)

---

#### 📌 Pergunta 6:
**A popularidade das plataformas de vídeo se traduz diretamente em engajamento de streaming?**  
📈 **Sim, em parte**. Músicas com videoclipe apresentaram streams. Embora não haja contagem explícita de views, o videoclipe pode funcionar como reforço de engajamento.
![image](https://github.com/user-attachments/assets/28511493-9f42-4da7-be93-8c9d441ef424)

---

#### 📌 Pergunta 7:
**Há inconsistências na classificação de gênero em relação aos atributos musicais das faixas?**  
⚠️ **Sim, parcialmente**. Algumas faixas de gêneros como *IndieFolk* ou *DreamPop* apresentaram métricas como `energy` e `danceability` elevadas, o que foge ao padrão esperado. Isso indica uma classificação pouco padronizada ou subgêneros híbridos.
![image](https://github.com/user-attachments/assets/f92f7560-cf9d-436e-92a3-f397bd64b161)
![image](https://github.com/user-attachments/assets/e5d5172c-176b-4ac5-a934-6b5c284ec970)

---

#### 📌 Pergunta 8:
**Faixas marcadas como feat (com participação de outros artistas) apresentam um padrão consistente de aumento ou diminuição de stream ou ranking em comparação com suas músicas solo?**  
📉 **Não há padrão claro**.  
- A faixa com **maior número de streams** é um **feat.**, com 40% a mais que a música solo mais ouvida.
  ![image](https://github.com/user-attachments/assets/077ca3f5-4762-456a-80be-e328277cd3e0)
- Por outro lado, os maiores picos de ranking pertencem a músicas solo.
  ![image](https://github.com/user-attachments/assets/bd8fb50f-efb0-4216-9c65-d25f33819657)

---

#### 📌 Pergunta 9:
**A colaboração impacta as métricas de forma previsível?**  
❌ **Não de forma consistente**. O impacto parece depender do artista participante. Feats. com grandes nomes têm melhor desempenho, mas não é regra geral.
  ![image](https://github.com/user-attachments/assets/077ca3f5-4762-456a-80be-e328277cd3e0)
  ![image](https://github.com/user-attachments/assets/bd8fb50f-efb0-4216-9c65-d25f33819657)
---

## ✅ Conclusão Parcial

As análises demonstraram que embora existam tendências observáveis, os dados apresentam lacunas que impedem algumas correlações diretas. Ainda assim:

- O projeto provou que é possível identificar **relações relevantes** entre métricas musicais e comerciais.
- O uso de **Spark SQL** mostrou-se eficaz para realizar consultas analíticas de alta performance.
- Fica evidente que, em alguns casos, o **sucesso de uma música não depende exclusivamente de sua popularidade nos rankings**.

## 📦 6. Entrega

Todo o código, dados e documentação estão disponíveis neste repositório público do GitHub. A estrutura inclui:

Além disso:

- A **interface visual do Databricks** foi usada para demonstrar a criação de tabelas e clusters.
- Os scripts de tratamento e modelagem foram executados e validados diretamente via notebook.

✅ Evidências visuais da execução estão disponíveis via screenshots dentro do repositório.

📎 Link para o notebook em execução no Databricks (visualização HTML):  
**https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/637213097784843/1487169712647734/1809110672226766/latest.html**

---

## 🧠 7. Autoavaliação

Este MVP cumpriu com sucesso o objetivo de analisar criticamente os dados musicais da artista Taylor Swift no Spotify. Algumas observações pessoais sobre o desenvolvimento:

### ✅ Pontos Fortes
- Utilização do **Databricks** e **Spark SQL** para consultas de alto desempenho.
- Boa **estruturação da análise** com foco em perguntas investigativas relevantes.
- Elaboração de **catálogo de dados completo**, incluindo normalização dos campos com transformações explícitas.

### ⚠️ Desafios Encontrados
- A conexão com a **API do Spotify falhou** repetidamente por problemas de timeout, mesmo utilizando bibliotecas como `Spotipy`.
- Foi necessário abandonar a coleta via API e partir para um dataset já estruturado no **Kaggle**, o que limitou a atualização em tempo real dos dados.
- Houve **dificuldade de cruzamento entre atributos de venda física e streaming**, o que impossibilitou análises mais profundas nesse aspecto.

### 💡 Próximos Passos
- Aplicar **modelos de clusterização ou classificação** para prever popularidade com base em características musicais.
- Avaliar a **evolução temporal** de métricas, aplicando filtros por data de lançamento.
- Implementar **visualizações interativas** com ferramentas como Power BI, Tableau ou Dash.
- Comparar dados de **outras artistas** com Taylor Swift para análise de mercado musical.

---

## 🏁 8. Como Executar

> Requisitos:
> - Conta gratuita na [Databricks Community Edition](https://community.cloud.databricks.com/login.html)
> - Importação do arquivo CSV e notebook

### Passos:

1. Acesse sua conta Databricks.
2. Crie um novo cluster em **Compute** e o inicie.
3. Vá até **Workspace** e importe o notebook `.dbc` disponível neste repositório.
4. Faça o upload do arquivo `taylor_swift_discography_updated.csv` para o **DBFS**.
5. Execute as células do notebook passo a passo.
6. Analise os resultados diretamente via queries em Spark SQL.

---


## ✍️ Autor

**Juliano Faiolo Terto de Oliveira**  
Aluno da Pós-graduação em Ciência de Dados e Analytics – PUC-Rio  
GitHub: [@julianofaiolo](https://github.com/Crlhju)  



---



