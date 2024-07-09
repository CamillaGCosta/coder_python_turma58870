# Documentação do Projeto Parcial

## Descrição

Este projeto busca criar bases de dados a partir de uma lista de CEPs, Códigos de Bancos e CNPJs usando a API do BrasilAPI e o salva-los como Tabelas em um banco de dados SQL. Ele também inclui um sistema de notificação que alerta quando há falhas no carregamento dos dados durante consulta às APIs.

## Bibliotecas Utilizadas
- `numpy`: Usada para manipulação de arrays.
- `pandas`: Usada para manipulação de dataframes.
- `requests`: Usada para fazer requisições HTTP.
- `plyer`: Usada para gerar notificações no sistema.
- `sqlite3`: Usada para interagir com banco de dados SQLite3

## Estrutura do Código

### Importação das Bibliotecas

``` Python

import numpy as np
import pandas as pd
import requests
from plyer import notification
import sqlite3

```

### Função de Alerta 
A função alerta é responsável por gerar notificações no sistema quando uma requisição HTTP falha.

``` Python

def alerta(nivel, dado):
    
    # Condicional para nível do alerta
    if nivel == 400:
        msg_alerta = 'Bad Request'
    elif nivel == 401:
        msg_alerta = 'Unauthorized'
    elif nivel == 403:
        msg_alerta = 'Forbidden'
    elif nivel == 404:
        msg_alerta = 'Not Found'
    elif nivel == 409:
        msg_alerta = 'Conflict'                   
    else:
        msg_alerta = f'Error {nivel}'

    # Mensagem de alerta
    texto = f'Falha no carregamento do {dado}'

     # Gerando o alerta
    notification.notify ( 
        title = msg_alerta,
        message = texto,
        app_name='App Fictício',
        timeout=10  
    )

```
### Criação de Dataframes

#### 1. CEPs 
O trecho do código a seguir cria uma base de dados a partir de uma lista de CEPs. Para cada CEP, ele faz uma requisição à API do BrasilAPI e armazena os dados recebidos em um DataFrame do Pandas.


``` Python

# Lista de CEPs
ceps = ['41720060', '09715022', '22222222']
# Lista para armazenar as respostas das requisições
responses = list()
# Requisições à API
for cep in ceps:
   url = f'https://brasilapi.com.br/api/cep/v1/{cep}'
   response = requests.get(url, verify=False)
   responses.append(response)
# DataFrame inicial (vazio)
df_full = pd.DataFrame(
   {
       'CEP': [np.nan],
       'Estado': [np.nan],
       'Cidade': [np.nan],
       'Bairro': [np.nan],
       'Rua': [np.nan],
       'Serviço': [np.nan],
   }
)
# Processamento das respostas
for n in range(len(responses)):
   if responses[n].status_code == 200:
       data = responses[n].json()
       df = pd.DataFrame(
           {
               'CEP': [data['cep']],
               'Estado': [data['state']],
               'Cidade': [data['city']],
               'Bairro': [data['neighborhood']],
               'Rua': [data['street']],
               'Serviço': [data['service']],
           }
       )
       df_full = pd.concat([df_full, df])
   else:
       nivel = responses[n].status_code
       dado = f'CEP: {ceps[n]}'
       alerta(nivel, dado)
# Removendo linhas com valores vazios
df_full = df_full.dropna()
# DataFrame final
TabelaCEP = pd.DataFrame(df_full)

TabelaCEP

```
#### 2. Bancos 
O trecho do código a seguir cria uma base de dados a partir de uma lista de códigos de bancos. Para cada código, ele faz uma requisição à API do BrasilAPI e armazena os dados recebidos em um DataFrame do Pandas.

``` Python

# Lista de códigos
codes = ['1', '70', '98999']
# Lista para armazenar as respostas das requisições
responses2 = list()
# Requisições à API
for code in codes:
    url = f'https://brasilapi.com.br/api/banks/v1/{code}'
    response2 = requests.get(url,verify=False)
    responses2.append(response2)
# DataFrame inicial (vazio)
df2_full = pd.DataFrame(
    {
        'ISPB': [np.nan],
        'Nome': [np.nan],
        'Código': [np.nan],
        'Nome completo': [np.nan],
    }
)
# Processamento das respostas
for n in range(len(responses2)):
    ## PARSE
    if responses2[n].status_code == 200:
        data2 = responses2[n].json()
        df2 = pd.DataFrame(
            {
                'ISPB': [data2['ispb']],
                'Nome': [data2['name']],
                'Código': [data2['code']],
                'Nome completo': [data2['fullName']],                               
            }
        )
        df2_full = pd.concat([df2_full, df2])
    else:
        nivel = responses2[n].status_code
        dado= f'Código: {codes[n]}'
        alerta(nivel, dado)
# Removendo linhas com valores vazios
df2_full = df2_full.dropna()
# DataFrame final
TabelaBanco=pd.DataFrame(df2_full)

TabelaBanco

```

#### 3. CNPJs
O trecho do código a seguir cria uma base de dados a partir de uma lista de CJPJs. Para cada CNPJ, ele faz uma requisição à API do BrasilAPI e armazena os dados recebidos em um DataFrame do Pandas.

``` Python

# Lista de códigos
cnpjs = ['33041260065290', '47960950000121', '22222222']
# Lista para armazenar as respostas das requisições
responses3 = list()
# Requisições à API
for cnpj in cnpjs:
    url = f'https://brasilapi.com.br/api/cnpj/v1/{cnpj}'
    response3 = requests.get(url,verify=False)
    responses3.append(response3)
# DataFrame inicial (vazio)
f3_full = pd.DataFrame(
    {
        'CNPJ': [np.nan],
        'Razão Social': [np.nan],
        'Nome Fantasia': [np.nan],
        'CEP': [np.nan],
        'Telefone': [np.nan],
    }
)
# Processamento das respostas
for n in range(len(responses3)):
    ## PARSE
    if responses3[n].status_code == 200:
        data3 = responses3[n].json()
        df3 = pd.DataFrame(
            {
                'CNPJ': [data3['cnpj']],
                'Razão Social': [data3['razao_social']],
                'Nome Fantasia': [data3['nome_fantasia']],
                'CEP': [data3['cep']], 
                'Telefone': [data3['ddd_telefone_1']],                            
            }
        )
        df3_full = pd.concat([df3_full, df3])
    else:
        nivel = responses3[n].status_code
        dado= f'Código: {cnpjs[n]}'
        alerta(nivel, dado)
# Removendo linhas com valores vazios
df3_full = df3_full.dropna()
# DataFrame final
TabelaCNPJ=pd.DataFrame(df3_full)

TabelaCNPJ

```

### Salvando e carregando as Tabelas na base de dados SQLite
O trecho do código a seguir contém funções Python para interagir com um banco de dados SQLite, permitindo salvar e carregar os DataFrames criados anteriormente como tabelas no banco de dados Coderhouse.db.

``` Python

def tabelas_bd():
    '''
        Retorna um dataframe com as tabelas do banco de dados.
    '''
    conn = sqlite3.connect('coderhouse.db')

    # Executar uma consulta que retorna as informações do esquema do banco de dados
    query = "SELECT name FROM sqlite_master WHERE type='table'"
    schema = pd.read_sql_query(query, conn)

    conn.close()

    return schema
def salva_bd(df, nome_tabela):
    '''
        Salva dataframe df na tabela nome_tabela.
    '''
    conn = sqlite3.connect('coderhouse.db')

    # Escrever o DataFrame na tabela 'nome_tabela'
    df.to_sql(nome_tabela, conn, if_exists='replace', index=False)

    conn.close()

    return True
def carrega_bd(nome_tabela):
    '''
        Carrega tabela nome_tabela num dataframe. 
    '''
    conn = sqlite3.connect('coderhouse.db')

    # Executar uma consulta SELECT na tabela 'produtos' e converter em um DataFrame
    query = f"SELECT * FROM {nome_tabela}"
    df = pd.read_sql(query, conn)

    conn.close()

    return df

# Salvando Tabela CEP no Banco de Dados
nome_tabela = 'TabelaCEP'
salva_bd(TabelaCEP, nome_tabela)
carrega_bd(nome_tabela)

# Salvando Tabela Banco no Banco de Dados
nome_tabela = 'TabelaBanco'
salva_bd(TabelaBanco, nome_tabela)
carrega_bd(nome_tabela)

# Salvando Tabela CNPJ no Banco de Dados
nome_tabela = 'TabelaCNPJ'
salva_bd(TabelaCNPJ, nome_tabela)
carrega_bd(nome_tabela)

# Validando as tabelas disponibilizadas no banco de dados
tabelas_bd()

```

## Como Executar

1. Certifique-se de ter todas as bibliotecas instaladas:

 ``` Python

pip install numpy pandas requests plyer

```

2. Execute o código Python.
