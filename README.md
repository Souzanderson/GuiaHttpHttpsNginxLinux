# Guia de Configuração HTTP/HTTPS usando NGINX no Linux Ubuntu e Python Flask

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
sudo nano /etc/systemd/system/flakservice.service
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
WatchdogSec=60
Restart=always

[Install]
WantedBy=multi-user.target
```