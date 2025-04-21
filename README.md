# product-catalog-streamlit-azure
Aplicação web para cadastro e visualização de produtos, desenvolvida com Streamlit. Utiliza armazenamento de imagens no Azure Blob Storage e persistência de dados em SQL Server. Ideal para portfólios, vitrines digitais ou sistemas simples de gerenciamento de produtos.

Durante o curso prático Microsoft Azure Cloud Native, promovido pela DIO em parceria com especialistas do setor, tive a oportunidade de construir uma aplicação completa em nuvem, explorando desde os fundamentos até práticas modernas de deploy, monitoramento e observabilidade.

- Etapas e Tecnologias Exploradas
Fundamentos do Azure e Serviços PaaS

Compreensão da arquitetura da nuvem Microsoft Azure, incluindo conceitos de recursos, grupos de recursos, escalabilidade e segurança.

Criação e configuração de ambientes via Azure Portal e Azure CLI.

Desenvolvimento e Deploy com Azure App Services

Criação de uma aplicação web com interface em Streamlit e backend conectado a um banco de dados SQL Server.

Deploy da aplicação utilizando o Azure App Service, aproveitando os recursos de escalabilidade automática e integração contínua.

Armazenamento em Nuvem

Utilização do Azure Blob Storage para persistência de arquivos (como imagens) com links públicos e privados.

Aplicação de boas práticas para organização de containers e blobs, incluindo segurança e versionamento.

Contêineres e Orquestração

Empacotamento da aplicação em Docker e publicação em Azure Container Registry (ACR).

Deploy utilizando Azure Container Apps e conceitos iniciais de Azure Kubernetes Service (AKS).

Compreensão de estratégias de escalonamento, balanceamento e resiliência.

Monitoramento e Observabilidade

Implementação de métricas e logs com Application Insights e Log Analytics.

Coleta e visualização de dados em tempo real para diagnóstico e melhoria contínua.

Configuração de alertas e painéis de monitoramento.

- Principais Insights e Possibilidades
A abstração oferecida pelos serviços PaaS da Azure facilita a criação e manutenção de aplicações modernas sem necessidade de infraestrutura dedicada.

O uso de contêineres amplia a portabilidade e modularização, tornando os ambientes mais flexíveis e escaláveis.

Ferramentas de observabilidade como Application Insights e Log Analytics são fundamentais para garantir a saúde da aplicação e tomada de decisões baseadas em dados.

A combinação entre armazenamento em nuvem, deploy automatizado e monitoramento ativo proporciona um fluxo de desenvolvimento robusto, seguro e orientado à performance.

- Resultados Alcançados
Desenvolvimento de uma aplicação web funcional com upload de arquivos, persistência em banco de dados e interface responsiva.

Realização de deploy completo em ambiente cloud-native, utilizando práticas modernas de desenvolvimento em nuvem.

Criação de uma solução end-to-end, com backend em contêiner, armazenamento cloud e monitoramento integrado.

![image](https://github.com/user-attachments/assets/2b8cbddb-bf21-4c1b-a538-d48fe6ae2261)
![image](https://github.com/user-attachments/assets/8277413d-cef5-44e4-9699-059067ebad0a)
![image](https://github.com/user-attachments/assets/2f9ef683-a226-451c-b7b7-83fee108aa15)
![image](https://github.com/user-attachments/assets/4001cea0-8e47-4f1d-bf9f-5b721bd662d5)
![image](https://github.com/user-attachments/assets/b08d1a86-2703-438b-ad04-9b47cb8942a5)

Code:

import streamlit as st
from azure.storage.blob import BlobServiceClient
import os
import pymssql
import uuid
import json
from dotenv import load_dotenv
load_dotenv()

blobConnectionString = os.getenv('BLOB_CONNECTION_STRING')
blobContainerName = os.getenv('BLOB_CONTAINER_NAME')
blobAccountName = os.getenv('BLOB_ACCOUNT_NAME')

SQL_SERVER = os.getenv('SQL_SERVER')
SQL_DATABASE = os.getenv('SQL_DATABASE')
SQL_USER = os.getenv('SQL_USER')
SQL_PASSWORD = os.getenv('SQL_PASSWORD')

st.title('Cadastro de Produtos')

#Formulário de cadastro de produtos
product_name = st.text_input('Nome do Produto')
product_price = st.number_input('Preço do Produto', min_value=0.0, format='%.2f')
product_description = st.text_area('Descrição do Produto')
product_image = st.file_uploader('Imagem do Produto', type=['jpg', 'png', 'jpeg'])

#Save image on blob storage
def upload_blob(file):
    blob_service_client = BlobServiceClient.from_connection_string(blobConnectionString)
    container_client = blob_service_client.get_container_client(blobContainerName)
    blob_name = str(uuid.uuid4()) + file.name
    blob_client = container_client.get_blob_client(blob_name)
    blob_client.upload_blob(file.read(), overwrite=True)
    image_url = f"https://{blobAccountName}.blob.core.windows.net/{blobContainerName}/{blob_name}"
    return image_url

def insert_product(product_name, product_price, product_description, product_image):
    try:
        imagem_url = upload_blob(product_image)
        conn = pymssql.connect(server=SQL_SERVER, user=SQL_USER, password=SQL_PASSWORD, database=SQL_DATABASE)
        cursor = conn.cursor()
        cursor.execute(f"INSERT INTO Produtos (nome, preco, descricao, imagem_url) VALUES ('{product_name}', {product_price}, '{product_description}', '{imagem_url}')")

        conn.commit()
        conn.close()
        return True
    except Exception as e:
        st.error(f'Erro ao inserir produto: {e}')
        return False

def list_products():
        try:
             conn = pymssql.connect(server=SQL_SERVER, user=SQL_USER, password=SQL_PASSWORD, database=SQL_DATABASE)
             cursor = conn.cursor()
             cursor.execute("SELECT * FROM Produtos")
             rows = cursor.fetchall()
             conn.close()
             return rows
        except Exception as e:
             st.error(f'Erro ao listar produtos: {e}')
             return []

def list_produtos_screen():
    products = list_products()
    if products:
        # Define o número de cards por linha
        cards_por_linha = 3
        # Cria as colunas iniciais
        cols = st.columns(cards_por_linha)
        for i, product in enumerate(products):
            col = cols[i % cards_por_linha]
            with col:
                st.markdown(f"### {product[1]}")
                st.write(f"**Descrição:** {product[2]}")
                st.write(f"**Preço:** R$ {product[3]:.2f}")
                if product[4]:
                    html_img = f'<img src="{product[4]}" width="200" height="200" alt="Imagem do produto">'
                    st.markdown(html_img, unsafe_allow_html=True)
                st.markdown("---")
            # A cada 'cards_por_linha' produtos, se ainda houver produtos, cria novas colunas
            if (i + 1) % cards_por_linha == 0 and (i + 1) < len(products):
                cols = st.columns(cards_por_linha)
    else:
        st.info("Nenhum produto encontrado.")   

if st.button('Salvar Produto'):
    insert_product(product_name, product_price, product_description, product_image)
    return_message = 'Produto salvo com sucesso'
    list_produtos_screen()

st.header('Produtos Cadastrados')

if st.button('Listar Produtos'):
    list_produtos_screen()
    return_message = 'Produtos listados com sucesso'




