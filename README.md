# QUEBRA_CSV_PYTHON_JUPYTER

import os
import pandas as pd

# Caminho para o arquivo na pasta Downloads
caminho_arquivo = os.path.join(os.path.expanduser('~'), 'Downloads', 'PA_ListaLoja_prod_DIRETORIA_2_20240520.csv')

# Diretório de destino para salvar os arquivos XLSX
pasta_destino = os.path.join(os.path.expanduser('~'), 'Downloads', 'Dir_2')

# Criar o diretório se ele não existir
os.makedirs(pasta_destino, exist_ok=True)

# Carregar o arquivo CSV, ignorando linhas com erros
df = pd.read_csv(caminho_arquivo, on_bad_lines='skip', sep=';')  # Especificar delimitador como ';'

# Verifique se os dados foram carregados corretamente
print("Primeiras linhas do DataFrame:")
print(df.head())

# Listar e corrigir os nomes das colunas
print("Nomes das colunas antes de corrigir:")
print(df.columns)

# Remover espaços em branco dos nomes das colunas
df.columns = df.columns.str.strip()

print("Nomes das colunas depois de corrigir:")
print(df.columns)

# Verifique se a coluna 'REGIONAL' está presente
if 'REGIONAL' not in df.columns:
    raise ValueError("A coluna 'REGIONAL' não foi encontrada no DataFrame. Verifique os nomes das colunas.")

# Gerar a hierarquia automaticamente
# Assumindo que as colunas do CSV são 'REGIONAL' e 'cd_loja'
hierarquia = {}

for reg_val in df['REGIONAL'].unique():
    fils = df[df['REGIONAL'] == reg_val]['cd_loja'].unique()
    hierarquia[reg_val] = fils.tolist()

# Função para salvar os arquivos de acordo com a hierarquia em formato XLSX
def salvar_xlsx_por_hierarquia(df, hierarquia, pasta_destino):
    for reg_key, fils in hierarquia.items():
        for fil in fils:
            # Filtrar o dataframe original com base na coluna 'cd_loja'
            df_fil = df[(df['REGIONAL'] == reg_key) & (df['cd_loja'] == fil)]
            
            # Verificar se existem dados para a filtragem atual
            if not df_fil.empty:
                # Converter para string antes de dividir e formatar
                fil_str = str(fil)
                reg_str = str(reg_key)
                
                # Remover pontos e espaços extras
                fil_num = fil_str.split(".")[0] if "." in fil_str else fil_str
                reg_num = reg_str.split()[0] if " " in reg_str else reg_str
                
                # Definir o nome do arquivo
                nome_arquivo = f'{reg_num}_{fil_num}.xlsx'
                caminho_arquivo_completo = os.path.join(pasta_destino, nome_arquivo)
                
                # Salvar o arquivo XLSX
                df_fil.to_excel(caminho_arquivo_completo, index=False)
                print(f'Arquivo salvo: {caminho_arquivo_completo}')

# Executar a função
salvar_xlsx_por_hierarquia(df, hierarquia, pasta_destino)
