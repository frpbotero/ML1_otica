Com certeza! Criar um conjunto de dados sintético, mas realista, é um excelente primeiro passo para um projeto de modelagem de regressão ou classificação para prever o churn (perda de clientes).

Para atender à sua solicitação, seguirei estes passos:

Definirei a Arquitetura do Banco de Dados: Vou projetar tabelas relacionais limpas para Clientes, Produtos, Vendas, Itens de Venda e Campanhas de Marketing. Isso garante que os dados sejam organizados e possam ser consultados (querys) de forma eficiente.

Injetarei Realismo: Vou considerar os eventos macroeconômicos e sociais do Brasil e do Amazonas entre 2015 e 2025 para modular os dados de vendas, como:

Crise Econômica (2015-2016): Queda na confiança do consumidor, possível redução no ticket médio e busca por produtos mais baratos.

Recuperação Lenta (2017-2019): Crescimento gradual nas vendas.

Pandemia de COVID-19 (2020-2021): Queda drástica nas vendas de lojas físicas, possível aumento em vendas online (se a ótica se adaptasse), foco em lentes com filtro de luz azul (home office) e uma queda nas vendas de óculos de sol. A crise de oxigênio em Manaus (início de 2021) certamente impactaria negativamente o comércio local.

Pós-Pandemia e Cenário Político (2022-2024): Incerteza econômica, inflação alta, impactando o poder de compra. Aumento de campanhas de marketing para recuperar clientes.

Gerarei um Script em Python: Criarei um script que gera os dados seguindo essa lógica e salva tudo em um único arquivo CSV, como solicitado. Usar um script é mais poderoso, pois você pode facilmente ajustar os parâmetros para gerar mais ou menos dados.

1. Estrutura do Banco de Dados (Tabelas)
   Aqui está um modelo de como as tabelas seriam estruturadas em um banco de dados relacional (como PostgreSQL ou MySQL).

Tabela 1: Clientes
Armazena informações sobre cada cliente único.

Nome da Coluna Tipo de Dado Descrição
cliente_id INTEGER (PK) Identificador único para cada cliente.
nome_completo VARCHAR(255) Nome fictício do cliente.
data_cadastro DATE Data em que o cliente fez a primeira compra.
cidade VARCHAR(100) Cidade do cliente (ex: Manaus, Itacoatiara).
data_nascimento DATE Data de nascimento para análise demográfica.

Exportar para as Planilhas
Tabela 2: Produtos
Catálogo de todos os produtos que a ótica vende.

Nome da Coluna Tipo de Dado Descrição
produto_id INTEGER (PK) Identificador único para cada produto.
descricao VARCHAR(255) Descrição do produto (ex: Lente Transitions, Armação Oakley).
tipo_produto VARCHAR(50) Categoria (ex: Armação, Lente, Óculos de Sol, Lente de Contato).
preco_base DECIMAL(10, 2) Preço de venda padrão do produto.

Exportar para as Planilhas
Tabela 3: CampanhasMarketing
Registros de todas as ações de marketing realizadas.

Nome da Coluna Tipo de Dado Descrição
campanha_id INTEGER (PK) Identificador único da campanha.
nome_campanha VARCHAR(255) Nome (ex: "Dia das Mães 2018", "Black Friday Pandemia").
data_inicio DATE Data de início da campanha.
data_fim DATE Data de término da campanha.
tipo_campanha VARCHAR(50) Mídia (ex: Rádio Local, Instagram, Panfleto, SMS).

Exportar para as Planilhas
Tabela 4: Vendas
O registro mestre de cada transação.

Nome da Coluna Tipo de Dado Descrição
venda_id INTEGER (PK) Identificador único da venda.
cliente_id INTEGER (FK) Chave estrangeira referenciando Clientes.
data_venda TIMESTAMP Data e hora exatas da venda.
desconto_total DECIMAL(10, 2) Valor total do desconto aplicado na venda.
valor_final DECIMAL(10, 2) Valor final pago pelo cliente.
campanha_id INTEGER (FK) Chave estrangeira opcional referenciando CampanhasMarketing.

Exportar para as Planilhas
Tabela 5: ItensVenda
Detalha quais produtos foram vendidos em cada transação (uma venda pode ter múltiplos itens).

Nome da Coluna Tipo de Dado Descrição
item_id INTEGER (PK) Identificador único do item da venda.
venda_id INTEGER (FK) Chave estrangeira referenciando Vendas.
produto_id INTEGER (FK) Chave estrangeira referenciando Produtos.
quantidade INTEGER Quantidade vendida do produto.
preco_unitario DECIMAL(10, 2) Preço do produto no momento da venda.

Exportar para as Planilhas 2. Script Python para Geração do CSV
Este script Python usará as bibliotecas pandas, faker e numpy para criar um arquivo CSV realista chamado registro_operacoes_otica.csv.

Instruções para rodar o código:

Instale as bibliotecas necessárias:

Bash

pip install pandas faker numpy
Copie e cole o código abaixo em um arquivo chamado gerador_dados.py.

Execute o arquivo: python gerador_dados.py.

Um arquivo CSV será criado no mesmo diretório.

Python

import pandas as pd
import numpy as np
from faker import Faker
import random
from datetime import datetime, timedelta

# Inicialização

fake = Faker('pt_BR')
Faker.seed(0)
random.seed(0)

# --- CONFIGURAÇÃO DA SIMULAÇÃO ---

DATA_INICIO = datetime(2015, 1, 1)
DATA_FIM = datetime(2025, 1, 1)
N_CLIENTES_INICIAIS = 800
N_PRODUTOS = 50
CIDADES_AM = ['Manaus', 'Manacapuru', 'Itacoatiara', 'Parintins', 'Tefé', 'Coari']

# --- DEFINIÇÃO DOS EVENTOS REAIS E SEU IMPACTO ---

# O modificador afeta o número de vendas diárias. 1.0 = normal.

eventos = {
"crise_economica_2015_2016": {"inicio": datetime(2015, 3, 1), "fim": datetime(2016, 12, 31), "modificador": 0.75},
"recuperacao_lenta_2017_2019": {"inicio": datetime(2017, 1, 1), "fim": datetime(2019, 12, 31), "modificador": 1.1},
"pandemia_inicio_2020": {"inicio": datetime(2020, 3, 15), "fim": datetime(2020, 6, 30), "modificador": 0.3},
"pandemia_crise_o2_manaus_2021": {"inicio": datetime(2021, 1, 10), "fim": datetime(2021, 2, 28), "modificador": 0.2},
"pandemia_adaptacao_2021": {"inicio": datetime(2020, 7, 1), "fim": datetime(2021, 12, 31), "modificador": 0.8},
"pos_pandemia_inflacao_2022_2024": {"inicio": datetime(2022, 1, 1), "fim": datetime(2024, 12, 31), "modificador": 0.9},
}

def get_modificador_vendas(data):
"""Retorna o modificador de vendas com base na data e eventos."""
for evento, detalhes in eventos.items():
if detalhes["inicio"] <= data <= detalhes["fim"]:
return detalhes["modificador"]
return 1.0 # Retorna normal se nenhum evento estiver ativo

# --- 1. GERAÇÃO DAS TABELAS BASE ---

# Clientes

clientes_data = []
for i in range(N_CLIENTES_INICIAIS):
data_cadastro = DATA_INICIO + timedelta(days=random.randint(0, 365))
clientes_data.append({
"cliente_id": i + 1,
"nome_completo": fake.name(),
"data_cadastro": data_cadastro.date(),
"cidade": random.choice(CIDADES_AM),
"data_nascimento": fake.date_of_birth(minimum_age=16, maximum_age=80)
})
df_clientes = pd.DataFrame(clientes_data)
cliente_id_counter = N_CLIENTES_INICIAIS + 1

# Produtos

tipos_produto = {
"Armação": (150, 800),
"Lente Simples": (100, 400),
"Lente Multifocal": (400, 1500),
"Lente com Antirreflexo": (250, 900),
"Lente Transitions": (500, 2000),
"Óculos de Sol": (200, 1200),
"Lente de Contato": (80, 250)
}
produtos_data = []
for i in range(N_PRODUTOS):
tipo = random.choice(list(tipos_produto.keys()))
preco_min, preco_max = tipos_produto[tipo]
produtos_data.append({
"produto_id": i + 1,
"descricao": f"{tipo} Marca {chr(65 + random.randint(0, 4))}",
"tipo_produto": tipo,
"preco_base": round(random.uniform(preco_min, preco_max), 2)
})
df_produtos = pd.DataFrame(produtos_data)

# Campanhas de Marketing (simplificado)

campanhas_data = [
{"campanha_id": 1, "nome_campanha": "Dia das Mães 2017", "data_inicio": datetime(2017, 5, 1), "data_fim": datetime(2017, 5, 14)},
{"campanha_id": 2, "nome_campanha": "Black Friday 2018", "data_inicio": datetime(2018, 11, 15), "data_fim": datetime(2018, 11, 23)},
{"campanha_id": 3, "nome_campanha": "Volta às Aulas 2019", "data_inicio": datetime(2019, 1, 10), "data_fim": datetime(2019, 2, 10)},
{"campanha_id": 4, "nome_campanha": "Fique em Casa - Lentes Blue Light 2020", "data_inicio": datetime(2020, 4, 1), "data_fim": datetime(2020, 6, 30)},
{"campanha_id": 5, "nome_campanha": "Recupera Amazonas 2022", "data_inicio": datetime(2022, 7, 1), "data_fim": datetime(2022, 7, 31)},
]

# --- 2. SIMULAÇÃO DAS VENDAS AO LONGO DO TEMPO ---

vendas_registros = []
venda_id_counter = 1
dias_totais = (DATA_FIM - DATA_INICIO).days

for dia in range(dias_totais):
data_atual = DATA_INICIO + timedelta(days=dia)
modificador = get_modificador_vendas(data_atual)

    # Simula o número de vendas diárias
    n_vendas_dia = int(np.random.poisson(lam=10) * modificador) # Média de 10 vendas/dia em tempos normais

    for _ in range(n_vendas_dia):
        # Decide se é um cliente novo ou existente
        if random.random() < 0.1 and modificador > 0.5: # 10% de chance de ser cliente novo
            novo_cliente = {
                "cliente_id": cliente_id_counter,
                "nome_completo": fake.name(),
                "data_cadastro": data_atual.date(),
                "cidade": random.choice(CIDADES_AM),
                "data_nascimento": fake.date_of_birth(minimum_age=16, maximum_age=80)
            }
            df_clientes = pd.concat([df_clientes, pd.DataFrame([novo_cliente])], ignore_index=True)
            cliente_id_selecionado = cliente_id_counter
            cliente_id_counter += 1
        else:
            # Pondera para clientes mais recentes terem mais chance de comprar de novo
            pesos = (pd.to_datetime(df_clientes['data_cadastro']) - pd.to_datetime(data_atual.date())).dt.days.abs()
            cliente_id_selecionado = df_clientes.sample(weights=1/(pesos + 1), n=1)['cliente_id'].iloc[0]

        # Simula os itens da venda
        n_itens = random.choices([1, 2], weights=[0.4, 0.6], k=1)[0] # Vende 1 ou 2 itens (armação + lente)
        produtos_vendidos = df_produtos.sample(n=n_itens)

        # Aplica desconto?
        desconto = 0.0
        if random.random() < 0.25: # 25% de chance de ter desconto
            desconto = random.choice([0.05, 0.1, 0.15]) # 5%, 10% ou 15%

        for _, produto in produtos_vendidos.iterrows():
            preco_venda = produto['preco_base'] * (1 - desconto)
            vendas_registros.append({
                "venda_id": venda_id_counter,
                "cliente_id": cliente_id_selecionado,
                "produto_id": produto['produto_id'],
                "quantidade": 1,
                "preco_unitario": produto['preco_base'],
                "desconto_aplicado": desconto,
                "valor_final_item": preco_venda,
                "data_venda": data_atual,
                "descricao_produto": produto['descricao'],
                "tipo_produto": produto['tipo_produto']
            })

        venda_id_counter += 1

df_vendas_final = pd.DataFrame(vendas_registros)

# Adiciona info do cliente ao registro de vendas

df_vendas_final = pd.merge(df_vendas_final, df_clientes, on='cliente_id')

# --- 3. EXPORTAÇÃO PARA CSV ---

colunas_ordenadas = [
'venda_id', 'data_venda', 'cliente_id', 'nome_completo', 'cidade', 'data_nascimento', 'data_cadastro',
'produto_id', 'descricao_produto', 'tipo_produto', 'quantidade', 'preco_unitario',
'desconto_aplicado', 'valor_final_item'
]
df_vendas_final = df_vendas_final[colunas_ordenadas]

df_vendas_final.to_csv("registro_operacoes_otica.csv", index=False, date_format='%Y-%m-%d %H:%M:%S')

print(f"Arquivo 'registro_operacoes_otica.csv' gerado com sucesso!")
print(f"Total de registros de vendas: {len(df_vendas_final)}")
print(f"Total de clientes: {len(df_clientes)}")

3. Exemplo do CSV Gerado (registro_operacoes_otica.csv)
   O arquivo CSV terá o seguinte formato, consolidando as informações para facilitar a análise:

Snippet de código

venda_id,data_venda,cliente_id,nome_completo,cidade,data_nascimento,data_cadastro,produto_id,descricao_produto,tipo_produto,quantidade,preco_unitario,desconto_aplicado,valor_final_item
1,2015-01-02 00:00:00,451,Srta. Lara Assunção,Manaus,1980-05-12,2015-01-01,23,Lente Transitions Marca C,Lente Transitions,1,1250.75,0.1,1125.68
2,2015-01-02 00:00:00,120,Sra. Laís Dias,Itacoatiara,1995-02-28,2015-01-01,45,Armação Marca E,Armação,1,450.20,0.0,450.20
3,2015-01-03 00:00:00,789,Dr. Kevin Rocha,Manacapuru,1975-11-20,2015-01-02,10,Óculos de Sol Marca B,Óculos de Sol,1,680.00,0.05,646.00
... 4. Como Usar Estes Dados para seu Projeto de Regressão
Com este CSV em mãos, você pode começar a engenharia de features para prever o churn.

Defina a Variável Alvo (Churn):

Primeiro, estabeleça uma regra. Exemplo: "Um cliente é considerado 'churned' (perdido) se ele não fizer uma nova compra nos últimos 24 meses".

Para cada cliente, crie uma coluna churn (1 para sim, 0 para não) com base nessa regra.

Crie as Variáveis Preditivas (Features):

Recência: Dias desde a última compra.

Frequência: Número total de compras do cliente.

Monetaridade (Ticket Médio): Gasto médio por transação.

Tempo de Vida do Cliente: Dias desde a data de cadastro.

Engajamento com Descontos: Percentual de compras feitas com desconto.

Variedade de Produtos: Quantos tipos de produtos diferentes o cliente já comprou (Armação, Lente de Sol, etc.).

Com essas variáveis, você terá um dataset pronto para treinar um modelo de regressão logística, XGBoost, ou outra técnica para prever a probabilidade de um cliente abandonar sua ótica, permitindo ações de marketing proativas e direcionadas.
