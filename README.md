# Odonto Guardião – Implantação com Docker

Este repositório contém os arquivos necessários para executar o sistema **Odonto Guardião** utilizando Docker e Docker Compose.

## 📦 Requisitos

Antes de continuar, certifique-se de que os seguintes softwares estão instalados:

- [Docker](https://www.docker.com/products/docker-desktop)
- [Docker Compose v2](https://docs.docker.com/compose/)

Caso precise de ajuda, consulte o documento `guia-instalacao-docker-compose.pdf` incluído neste repositório.

## 🚀 Como subir o sistema

1. **Clone este repositório ou baixe os arquivos**

2. **Navegue até o diretório onde está o arquivo `docker-compose.yml`**

3. **Execute o seguinte comando no terminal:**

```bash
docker compose up -d
```

Este comando irá:

- Baixar as imagens do Docker Hub
- Criar dois containers: um para o backend e outro para o frontend
- Expor as portas padrão:
  
```bash
Frontend: http://localhost:3000
Backend: http://localhost:8080
````

⚠️ Você pode alterar as portas no arquivo docker-compose.yml se necessário, caso alguma já esteja em uso.
