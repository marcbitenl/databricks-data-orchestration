# **Earthquake Data Pipeline - Databricks & Azure Data Lake** 

## 📌 Visão Geral

Este repositório contém uma **pipeline de dados distribuída e escalável** construída no **Azure Databricks**, projetada para coletar, transformar e organizar dados sísmicos da **USGS Earthquake API**.

A solução segue o modelo **Medallion Architecture** (**Bronze, Silver e Gold**), permitindo ingestão eficiente, tratamento de dados e armazenamento estruturado para análise avançada.

O pipeline é **totalmente orquestrado dentro do Databricks**, sem necessidade de ferramentas externas como Data Factory ou Synapse Analytics.

---

## 🏠 Arquitetura da Pipeline

A execução do fluxo de dados é gerenciada por um **workflow no Databricks**, que segue a seguinte sequência:

### 🔹 **Origem dos Dados (Azure External Locations)**

- Os dados são extraídos a partir da funcionalidade **Dados Externos** no Databricks.
- Diferente da abordagem tradicional com **Azure Data Factory**, utilizamos **credenciais gerenciadas** no próprio Databricks.
- A credencial foi criada e configurada para acessar **containers específicos no Azure Data Lake**, garantindo controle granular de permissões.
- As localizações externas foram configuradas para as camadas **Bronze, Silver e Gold**, permitindo que os dados sejam acessados diretamente no Databricks sem necessidade de cópias adicionais.

### 🔹 **Controle de Acesso via IAM**

- O acesso ao **Unity Catalog** foi configurado via **IAM (Identity and Access Management)** no Azure.
- A identidade gerenciada `unity-catalog-access-connector` foi adicionada como **Colaboradora de Dados do Storage Blob**, garantindo que os notebooks e workflows no Databricks possam acessar os containers do Data Lake de forma segura.
- Esse método reduz a necessidade de armazenar credenciais dentro do código e melhora a governança dos dados.

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

- **Normalização dos dados**, aplicando ajustes estruturais e padronizações.
- **Enriquecimento das informações**, incluindo junções e manipulações adicionais.
- **Criação de colunas estruturadas** para facilitar análises futuras.
- **Escrita dos dados na camada Silver** dentro do **Azure Data Lake**.

### 🏆 **Gold Layer (Dados Prontos para Consumo)**

- Agregação de dados e cálculos estatísticos relevantes.
- Conversão dos dados para um formato otimizado para consultas.
- Exportação dos dados refinados para a camada **Gold** no **Azure Data Lake**.

---

## ⚙️ Tecnologias Utilizadas

✅ **Azure Databricks** → Orquestração e processamento distribuído com **Apache Spark**  
✅ **Azure Data Lake Storage (ADLS)** → Armazenamento estruturado em camadas  
✅ **Unity Catalog** → Governança e controle de acesso centralizado  
✅ **Parquet Format** → Arquivos otimizados para leitura e performance  
✅ **PySpark** → Manipulação eficiente de grandes volumes de dados

---

## 🔄 Fluxo de Execução

A pipeline é executada via **workflow no Databricks**, conforme o JSON do job:

1️⃣ **Bronze** → Coleta e armazena os dados brutos da API.  
2️⃣ **Silver** → Limpeza, transformação e normalização dos dados.  
3️⃣ **Gold** → Agregação e otimização dos dados para análise.

Cada etapa é executada apenas **se a anterior for bem-sucedida** (`run_if: ALL_SUCCESS`), garantindo a integridade do pipeline.

---

## 🚀 Benefícios da Solução

✔️ **Automatização Completa** → Fluxo 100% gerenciado dentro do Databricks.  
✔️ **Escalabilidade** → Capacidade de processar grandes volumes de dados sísmicos.  
✔️ **Eficiência e Organização** → Dados estruturados em camadas para fácil consumo.  
✔️ **Alto Desempenho** → Uso otimizado do Apache Spark para processamento distribuído.  
✔️ **Segurança Aprimorada** → Uso de IAM e Unity Catalog para controle de acesso seguro.

---

## 💡 Contribuições

Sinta-se à vontade para sugerir melhorias, abrir issues ou contribuir para otimizar a pipeline!

📩 Para dúvidas ou sugestões, entre em contato.


