# TAREA DE LLM

## Descripción
Este repositorio contiene un conjunto de scripts que utilizan la biblioteca LangChain para realizar operaciones de procesamiento de texto, carga de documentos y búsqueda de similitud en un corpus de conocimiento.

## Requisitos
Para ejecutar estos scripts, necesitarás tener instaladas las siguientes bibliotecas:
- `langchain_community`
- `langchain`
- `pinecone`

Además, necesitarás configurar las siguientes variables de entorno:
- `OPENAI_API_KEY`: Tu clave de API de OpenAI.
- `PINECONE_API_KEY`: Tu clave de API de Pinecone.
- `PINECONE_ENV`: El entorno de Pinecone que estés utilizando (por ejemplo, `"gcp-starter"`).

## Cómo ejecutar
1. Clona este repositorio en tu máquina local.
2. Asegúrate de tener todas las dependencias instaladas.
3. Configura las variables de entorno con tus claves de API.
4. Coloca el archivo `Conocimiento.txt` en el mismo directorio que los scripts.
5. Ejecuta el script `main.py`.

## Arquitectura
El script `main.py` carga documentos de texto utilizando la biblioteca LangChain, divide los documentos en fragmentos y los almacena en un índice de Pinecone para realizar búsquedas de similitud. La arquitectura general del sistema implica las siguientes partes:

- **Carga de documentos**: Utiliza `TextLoader` de LangChain para cargar documentos de texto desde un archivo.
- **División de texto**: Utiliza `RecursiveCharacterTextSplitter` de LangChain para dividir los documentos en fragmentos.
- **Embeddings**: Utiliza `OpenAIEmbeddings` de LangChain para generar incrustaciones de texto.
- **Almacenamiento y búsqueda**: Utiliza `PineconeVectorStore` de Pinecone para almacenar los vectores de incrustación y realizar búsquedas de similitud.

## Ejemplo de uso
```python
from langchain_community.document_loaders import TextLoader
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_pinecone import PineconeVectorStore
from pinecone import Pinecone, PodSpec
import os

# Configurar las variables de entorno
os.environ["OPENAI_API_KEY"] = "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"
os.environ["PINECONE_API_KEY"] = "2796de3c-989e-4880-8810-e9912c4c91dd"
os.environ["PINECONE_ENV"] = "gcp-starter"

def loadText():
    # Cargar documentos
    loader = TextLoader("Conocimiento.txt")
    documents = loader.load()

    # Dividir documentos en fragmentos
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size = 1000,
        chunk_overlap  = 200,
        length_function = len,
        is_separator_regex = False,
    )
    docs = text_splitter.split_documents(documents)

    # Generar incrustaciones de texto y almacenar en Pinecone
    embeddings = OpenAIEmbeddings()
    index_name = "langchain-demo"
    pc = Pinecone(api_key='2796de3c-989e-4880-8810-e9912c4c91dd')
    
    # Crear un nuevo índice si no existe
    if len(pc.list_indexes()) == 0:
        pc.create_index(
            name=index_name,
            dimension=1536,
            metric="cosine",
            spec=PodSpec(
                environment=os.getenv("PINECONE_ENV"),
                pod_type="p1.x1",
                pods=1
            )
        )
    
    # Almacenar vectores de incrustación en Pinecone
    PineconeVectorStore.from_documents(docs, embeddings, index_name=index_name)

def search():
    # Realizar búsqueda de similitud
    embeddings = OpenAIEmbeddings()
    index_name = "langchain-demo"
    docsearch = PineconeVectorStore.from_existing_index(index_name, embeddings)
    query = "Qué es un ataque de XSS"
    docs = docsearch.similarity_search(query)
    print(docs[0].page_content)

# Cargar documentos y realizar búsqueda
loadText()
search()
