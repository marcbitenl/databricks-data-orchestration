# 🌍 **Earthquake Data Pipeline - Databricks & Azure Data Lake** ⚡

<img width="962" alt="image" src="https://github.com/user-attachments/assets/3572d185-d1aa-4304-92c1-900c22ad9908" />


## 📌 Visão Geral

Este repositório contém uma **pipeline de dados distribuída e escalável** construída no **Azure Databricks**, projetada para coletar, transformar e organizar dados sísmicos da **USGS Earthquake API**.

A solução segue o modelo **Medallion Architecture** (**Bronze, Silver e Gold**), permitindo ingestão eficiente, tratamento de dados e armazenamento estruturado para análise avançada.

O pipeline é **totalmente orquestrado dentro do Databricks**, sem necessidade de ferramentas externas como **Data Factory** ou **Synapse Analytics**.

---

## 🏠 Arquitetura da Pipeline

A execução do fluxo de dados é gerenciada por um **workflow no Databricks**, que segue a seguinte sequência:

### 🔹 **Uma Abordagem Moderna: Tudo no Databricks**

Esta pipeline adota uma **abordagem moderna baseada inteiramente no Databricks**, dispensando ferramentas externas como **Azure Data Factory e Synapse Analytics**. Essa estratégia se alinha com a tendência atual do mercado, onde muitas empresas estão migrando para um modelo **Lakehouse**.

✅ **Governança centralizada** → Uso de **IAM e External Locations** para controle seguro dos dados.  
✅ **Menos dependências** → Tudo é gerenciado dentro do **Databricks**, reduzindo a necessidade de integrações externas.  
✅ **Melhor performance** → A utilização do **Delta Lake e Parquet** garante **baixo custo e alta eficiência**.  
✅ **Flexibilidade** → Permite **ajustes rápidos** e mais controle sobre a orquestração sem precisar de pipelines separados no ADF.  
✅ **Custo otimizado** → Redução de custos ao evitar execução desnecessária de pipelines no Azure Data Factory.

---

### 🔹 **Origem dos Dados (Azure External Locations)**

- Os dados são extraídos a partir da funcionalidade **Dados Externos** no Databricks.
- Diferente da abordagem tradicional com **Azure Data Factory**, utilizamos **credenciais gerenciadas** no próprio Databricks.
- A credencial foi criada e configurada para acessar **containers específicos no Azure Data Lake**, garantindo controle granular de permissões.
- As localizações externas foram configuradas para as camadas **Bronze, Silver e Gold**, permitindo que os dados sejam acessados diretamente no Databricks sem necessidade de cópias adicionais.

### 🔹 **Controle de Acesso via IAM**

- O acesso ao **Azure Data Lake** foi configurado via **IAM (Identity and Access Management)** no Azure.
- A identidade gerenciada `unity-catalog-access-connector` foi adicionada como **Colaboradora de Dados do Storage Blob**, garantindo que os notebooks e workflows no Databricks possam acessar os containers do Data Lake de forma segura.
- Esse método reduz a necessidade de armazenar credenciais dentro do código e melhora a governança dos dados.

### 🔹 **Bibliotecas Externas Utilizadas**

- Durante a execução do pipeline, foi necessária a instalação da biblioteca **`reverse_geocoder`** diretamente no **cluster do Databricks**.
- A instalação foi feita via **PyPI**, garantindo suporte a funcionalidades de geocodificação reversa utilizadas no processamento dos dados sísmicos.
- Exemplo de instalação manual:
  ```python
  %pip install reverse_geocoder
  ```
  Ou, via interface do Databricks:
  - Navegue até o cluster e vá para a aba **Bibliotecas**.
  - Clique em **Instalar novo** e selecione **PyPI**.
  - Digite `reverse_geocoder` e clique em **Instalar**.

### 🔸 **Bronze Layer (Ingestão de Dados Brutos)**

- **Conexão com a API**: A API da **USGS Earthquake** é acessada utilizando o método `GET` do módulo `requests`, permitindo a obtenção de dados sísmicos em tempo real.
- **Parâmetros Configurados**: A data inicial e final são definidas dinamicamente e passadas como query parameters.
- **Leitura e Conversão de Dados**: O retorno da API (JSON) é convertido para um **DataFrame PySpark** para manipulação eficiente.
- **Armazenamento no Azure Data Lake**: Os dados são salvos no formato **Parquet**, garantindo eficiência e compatibilidade com as próximas camadas.
- **Definição de Variáveis para Próximas Etapas**:
  ```python
  start_date = "2025-02-10"
  bronze_adls = "abfss://bronze@yourdatalake.dfs.core.windows.net/earthquake_data"
  silver_adls = "abfss://silver@yourdatalake.dfs.core.windows.net/earthquake_data"
  dbutils.jobs.taskValues.set(
      key="bronze_output",
      value={"start_date": start_date, "bronze_adls": bronze_adls, "silver_adls": silver_adls}
  )
  ```

### 🔸 **Silver Layer (Transformação e Enriquecimento)**

- **Recuperação dos valores da Bronze Layer**:
  ```python
  bronze_output = dbutils.jobs.taskValues.get(
      taskKey="Bronze",
      key="bronze_output"
  )
  start_date = bronze_output.get("start_date", "")
  bronze_adls = bronze_output.get("bronze_adls", "")
  silver_adls = bronze_output.get("silver_adls", "")
  ```
- **Normalização dos dados**, aplicando ajustes estruturais e padronizações.
- **Enriquecimento das informações**, incluindo junções e manipulações adicionais.
- **Criação de colunas estruturadas** para facilitar análises futuras.
- **Escrita dos dados na camada Silver** dentro do **Azure Data Lake**.

### 🏆 **Gold Layer (Dados Prontos para Consumo)**

- **Recuperação dos valores da Silver Layer**:
  ```python
  silver_output = dbutils.jobs.taskValues.get(
      taskKey="Silver",
      key="silver_output"
  )
  ```
- Agregação de dados e cálculos estatísticos relevantes.
- Conversão dos dados para um formato otimizado para consultas.
- Exportação dos dados refinados para a camada **Gold** no **Azure Data Lake**.

---

## 🚀 Benefícios da Solução

✔️ **Automatização Completa** → Fluxo 100% gerenciado dentro do Databricks.  
✔️ **Escalabilidade** → Capacidade de processar grandes volumes de dados sísmicos.  
✔️ **Eficiência e Organização** → Dados estruturados em camadas para fácil consumo.  
✔️ **Alto Desempenho** → Uso otimizado do Apache Spark para processamento distribuído.  
✔️ **Segurança Aprimorada** → Uso de IAM e External Locations para controle de acesso seguro.

---

## 💡 Contribuições

Sinta-se à vontade para sugerir melhorias, abrir issues ou contribuir para otimizar a pipeline!

📩 Para dúvidas ou sugestões, entre em contato.

