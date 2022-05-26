# Guia de Configuração HTTP/HTTPS usando NGINX no Linux Ubuntu e Python Flask


* Primeiro transfira seus arquivos para um diretório dentro do seu VPS usando o FileZilla;

* Supondo que você transferiu seus arquivos para o diretório **/home/ubuntu/teste/projeto**
entre no diretório usando o comando:

```
cd /home/ubuntu/teste/projeto
```

* Agora instale o python no seu VPS:

```
sudo apt-get update
sudo apt-get -y install python3
sudo apt-get -y install python3-pip
sudo apt-get -y install python3-venv
```

* Teste a instalação:

```
python3 --version
pip3 --version
```

* Crie um ambiente virtual para seu projeto e instale as dependências do requirements.txt:

```
python3 -m venv env
./env/bin/pip install -r requirements.txt
```

* Teste seu aplicativo (supondo que seu app esteja localizado em **app.py**):

```
./env/bin/python app.py
```

