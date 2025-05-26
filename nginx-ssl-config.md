# Configuração Nginx como Proxy Reverso com Docker e SSL

## 🎯 Objetivo

Configurar o Nginx na VM para atuar como **proxy reverso** para uma aplicação (Frontend/Backend) rodando em contêineres Docker na mesma VM, e habilitar **HTTPS** para o domínio.

> ⚠️ **IMPORTANTE**: Onde for mencionado `meu_projeto` nos caminhos de arquivos, substitua pelo nome real do projeto, como por exemplo `odontoguardiao`.

> ⚠️ **IMPORTANTE**: Onde for mencionado `odontoguardiao.splab.ufcg.edu.br`, substitua pelo domínio real que se pretende deixar público
---

## 1️⃣ Configuração Base do Nginx (HTTP)

Este é o arquivo de configuração Nginx (`nginx.config`) que serve como base. Ele lida com o proxy reverso para o Frontend e Backend via HTTP. As configurações SSL (HTTPS) serão adicionadas automaticamente por uma ferramenta posteriormente.

### 📁 Local do arquivo
```bash
/etc/nginx/sites-available/odontoguardiao/nginx.config
```

### 📝 Conteúdo do `nginx.config`

```nginx
server {
    server_name odontoguardiao.splab.ufcg.edu.br localhost;
    listen 80 default_server;
    listen [::]:80 default_server; # Apenas para IPV6, caso contrário pode remover essa linha

    # Configuração para Frontend
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Configuração para Backend API
    location /api {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Páginas de erro
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

---

## 2️⃣ Ativação da Configuração no Nginx

Após salvar o `nginx.config` acima, é necessário ativá-lo e garantir que ele não entre em conflito com outras configurações.

### 🗑️ Remover configuração padrão (se existir)
```bash
sudo rm /etc/nginx/sites-enabled/default
```

### 🔗 Criar link simbólico para ativar a configuração
```bash
sudo ln -s /etc/nginx/sites-available/odontoguardiao/nginx.config /etc/nginx/sites-enabled/odontoguardiao.conf
```

### ✅ Testar a sintaxe do Nginx
```bash
sudo nginx -t
```
Certifique-se de que o retorno seja `syntax is ok` e `test is successful`. Se houver qualquer erro, ele indicará a linha e o arquivo problemáticos.

### 🔄 Recarregar o Nginx
```bash
sudo systemctl reload nginx
```

Agora, ao acessar o IP da sua VM (ou o domínio configurado), o Nginx deve carregar sua configuração e direcionar para o frontend/backend.

---

## 3️⃣ Habilitando HTTPS com Let's Encrypt e Certbot

Para adicionar SSL/HTTPS, a ferramenta **Certbot** é recomendada para automatizar a obtenção de certificados gratuitos (Let's Encrypt) e a configuração do Nginx.

### 📋 Pré-requisitos

- ✅ O nome de domínio (`odontoguardiao.splab.ufcg.edu.br`) deve estar configurado no DNS para apontar para o IP público da VM
- ✅ As portas **80 (HTTP)** e **443 (HTTPS)** devem estar abertas no firewall da VM
  ```bash
  sudo ufw allow 'Nginx Full'
  ```

### 🛠️ Passos de Instalação

#### 1. Atualizar o sistema
```bash
sudo apt update
sudo apt upgrade -y
```

#### 2. Instalar Certbot
```bash
sudo apt install certbot python3-certbot-nginx -y
```

#### 3. Configurar SSL
```bash
sudo certbot --nginx -d odontoguardiao.splab.ufcg.edu.br
```

### 📝 Durante a execução do Certbot

O Certbot irá solicitar algumas informações:

1. 📧 **Forneça um endereço de e-mail**
2. ✅ **Aceite os termos de serviço**
3. 🔄 **Escolha se deseja redirecionar todo o tráfego HTTP para HTTPS** (recomendado)

### 🤖 O que o Certbot faz automaticamente

- 🔧 Modifica o `nginx.config` para adicionar diretivas SSL
- 🔀 Cria bloco server adicional para redirecionamento de HTTP para HTTPS
- 🔄 Configura renovação automática do certificado
- ✅ Testa e recarrega a configuração do Nginx

---

## ✨ Resultado Final

Após esses passos, sua aplicação deverá estar acessível via **HTTPS** pelo domínio configurado:

🌐 **https://odontoguardiao.splab.ufcg.edu.br**

---

## 🏷️ Tags de Referência

`#nginx` `#docker` `#ssl` `#proxy-reverso` `#lets-encrypt` `#certbot` `#https`
