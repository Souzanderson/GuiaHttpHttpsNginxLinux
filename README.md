# Guia de Configuração HTTP/HTTPS usando NGINX no Linux Ubuntu e Python Flask

## Índice

- [Instalação, configuração e execução do projeto em Flask](#Instalação,-configuração-e-execução-do-projeto-em-Flask)
- [Adicionando o projeto como serviço ao sytemd do Linux Ubuntu](#Adicionando-o-projeto-como-serviço-ao-sytemd-do-Linux-Ubuntu)
- [Instalação e configuração do NGINX](#Instalação-e-configuração-do-NGINX)
- [Gerando os certificados para nosso domínio usando o CERTBOT](#gerando-os-certificados-para-nosso-domínio-usando-o-certbot-será-necessário-ter-um-domínio-configurado-para-o-ip-do-vps)
- [Instalando os certificados e configurando o NGINX para responder a conexões HTTPS (Porta 443)](#instalando-os-certificados-e-configurando-o-nginx-para-responder-a-conexões-https-porta-443)

## Instalação, configuração e execução do projeto em Flask

Inicialmente iremos enviar os arquivos do projeto, configurar o python e rodar em um ambiente virtual para testar nossa aplicação:

* Primeiro transfira seus arquivos para um diretório dentro do seu VPS usando o FileZilla;

* Supondo que você transferiu seus arquivos para o diretório **/home/ubuntu/teste/projeto**
entre no diretório usando o comando:

```bash
cd /home/ubuntu/teste/projeto
```

* Agora instale o python no seu VPS:

```bash
sudo apt-get update
sudo apt-get -y install python3
sudo apt-get -y install python3-pip
sudo apt-get -y install python3-venv
```

* Teste a instalação:

```bash
python3 --version
pip3 --version
```

* Crie um ambiente virtual para seu projeto e instale as dependências do requirements.txt:

```bash
python3 -m venv env
./env/bin/pip install -r requirements.txt
```

* Teste seu aplicativo (supondo que seu app esteja localizado em **app.py**):

```bash
./env/bin/python app.py
```

Agora seu projeto está pronto para rodar no Linux!

## Adicionando o projeto como serviço ao sytemd do Linux Ubuntu

Para que nosso projeto rode em segundo plano e execute ao ligar o VPS, ou mesmo que ele falhe, volte a executar,
temos que configurar nosso aplicativo para rodar dentro dos processos de serviço do linux, conhecido como **systemd**

* primeiro criaremos um arquivo na área de serviços do linux ubuntu:

```bash
sudo nano /etc/systemd/system/flaskservice.service
```

* Ao criar o arquivo, ele será aberto vazio no shell, adicionaremos o seguinte conteúdo a ele:

```ini
[Unit]
Description=Flask Server
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/teste/projeto
ExecStart=/home/ubuntu/teste/projeto/env/bin/python /home/ubuntu/teste/projeto/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

* Salve o arquivo usando **CTRL+S** e feche usando **CTRL+X**, agora vamos iniciar o nosso aplicativo como serviço
com os seguintes comandos:

```bash
sudo systemctl daemon-reload
sudo systemctl start flaskservice
```

* Se nenhum erro foi emitido no prompt, nosso aplicativo está rodando como serviço, para vê-lo executando:

```bash
systemctl status flaskservice
```

* Para ver o log da nossa aplicação:

```bash
journalctl -u flaskservice -f
```

Agora nossa aplicação está rodando como serviço, é possível configurar outras coisas, como watchdogs para reiniciá-la periódicamente,
ou workers para ter várias instâncias rodando da mesma aplicação. Mas nos limitaremos ao que fizemos e iniciaremos uma configuração de
servidor http.

## Instalação e configuração do NGINX

Para acessar a aplicação de forma externa, através do IP de nosso VPS ou de um domínio configurado para direcionar as requisições para o 
IP do VPS, precisaremos de um servidor http, para isso vamos instalar o NGINX, um servidor HTTP estável e com boas possibilidades de configuração.

* Instalaremos o NGINX em nosso servidor linux com o comando:

```bash
sudo apt-get -y install nginx
```

* Para verificar a instalação, basta abrir o link do domínio direcionado para seu VPS ou o IP de seu VPS no navegador, uma mensagem
de boas-vindas do NGINX deverá aparecer! Podemos checar o funcionamento com o **systemctl**:

```bash
systemctl status nginx
```

* Agora iremos configurar o NGINX para direcionar a rota principal para nosso serviço Flask que está rodando na porta que configuramos 
em nosso aplicativo (neste caso **porta 5001**):

```bash
sudo nano /etc/nginx/sites-available/default
```

* Um arquivo de texto contendo as configurações do NGINX aparecerá, edite a parte do server que contém a porta 80,
para a seguinte forma:


```nginx
server {
            listen       80;
            server_name  localhost 127.0.0.1;
            client_max_body_size 2048M;
            
            location /api {
                            proxy_pass  "http://0.0.0.0:5001/";
                            proxy_set_header    X-Forwarded-For $remote_addr;
                       }
        }   
```

* Salve o arquivo usando **CTRL+S** e feche usando **CTRL+X**, e vamos reiniciar o nginx:

```bash
sudo systemctl restart nginx
systemctl status nginx
```

* Se tudo estiver correto, nosso servidor agora está direcionado para atender chamadas http para 
o nosso aplicativo flask na rota principal. Basta abrir o navegador e digitar o IP do VPS ou o domínio
direcionado para ele, lembrando que, por hora, nosso servidor só atenderá chamadas http (chamadas https serão negadas).
Exemplo, supondo que o IP do servidor é **http://123.12.123.12**.


## Gerando os certificados para nosso domínio usando o CERTBOT (será necessário ter um domínio configurado para o IP do VPS)

Agora vamos passar para a configuração de certificados de https/tsl, para tanto,
vamos gerar certificados válidos:

* Primeiramente vamos instalar o **snapd** (gerenciador de pacotes) que vai ser o responsável pela instalação do **certbot** (gerenciador de certificados):


```bash
sudo apt-get -y install snapd
sudo snap install core
sudo snap refresh core
```

* Após instalado, basta instalar o Certbot:

```bash
sudo snap install --classic certbot
```

* Criamos um symlink para garantir que o Certbot está rodando:

```bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

* Agora geraremos um certificado válido usando:

```bash
sudo certbot certonly --nginx
```

* Responderemos às perguntas do certbot:
    - Primeiramente o domínio que iremos certificar: <seu_domínio__aqui.com>
    - Um par Chave/Certificado será gerado, copie os links, os arquivos geralmente estão em: 

```bash
    /etc/letsencrypt/live/<seu_domínio__aqui.com>/fullchain.pem
    /etc/letsencrypt/live/<seu_domínio__aqui.com>/privkey.pem
```

## Instalando os certificados e configurando o NGINX para responder a conexões HTTPS (Porta 443)

Para configurar o HTTPS do NGINX, vamos reabrir o arquivo de configurações e adicionar a configuração da porta 443.


* Abriremos o arquivo de configuração:

```bash
sudo nano /etc/nginx/sites-available/default
```

* Um arquivo de texto contendo as configurações do NGINX aparecerá, edite a parte do server que contém a porta 443,
ou crie as seguintes instruções (agora precisaremos inserir as informações do certificado gerado na etapa anterior):


```nginx
server {
            listen 443;

            ssl_certificate     /etc/letsencrypt/live/<seu_domínio__aqui.com>/fullchain.pem;
            ssl_certificate_key /etc/letsencrypt/live/<seu_domínio__aqui.com>/privkey.pem;

            ssl on;

            location ^~ / {
                proxy_pass  "http://0.0.0.0:5001/";
                add_header Last-Modified $date_gmt;
                add_header Cache-Control 'no-store, no-cache, must-revalidate, proxy-revalidate, max-age=0';
                if_modified_since off;
                expires off;
                etag off;
            }
        }
```

* Salve o arquivo usando **CTRL+S** e feche usando **CTRL+X**, e vamos reiniciar o nginx:

```bash
sudo systemctl restart nginx
systemctl status nginx
```

* Se tudo estiver correto, nosso servidor agora está direcionado para atender chamadas http/https para 
o nosso aplicativo flask na rota principal. Basta abrir o navegador e digitar o domínio
direcionado para ele. Exemplo, supondo que o IP do servidor é **https://<seu_domínio__aqui.com>**.
