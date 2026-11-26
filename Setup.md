# Guia para o setup do repositorio

### 0. Clone e acesse o repositorio:
```sh
git clone https://github.com/jvcmtr/analise_vulnerabilidades_tp3.git
cd analise_vulnerabilidades_tp3
```

## 1. Crie e acesse o ambiente virtual:
Crie o ambiente (linux):
```sh
python3 -m venv venv
source venv/bin/activate
```

## 2. Instale as dependencias:
```sh
pip install -r requirements.txt
```

## 3. Inicialize as variaveis de ambiente:
Crie um arquivo `.env` com base no exemplo fornescido em `.env.example`

## 4. Instale as ferramentas (linux):
execute:
```sh
sudo apt update &&
sudo apt install nmap gobuster
```