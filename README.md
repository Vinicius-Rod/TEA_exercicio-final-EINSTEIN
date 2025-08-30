# 1 - Preparando o Ambiente

# 1.1 - Diretórios:

```bash
rm -r sample_data/
mkdir TEA_Einstein
mkdir TEA_Einstein/dados
mkdir TEA_Einstein/analises
mkdir TEA_Einstein/dados/annovar
mkdir TEA_Einstein/dados/annovar/amostra_01
mkdir TEA_Einstein/dados/annovar/amostra_02
```

# 1.2 - Download dos arquivos

```bash
output = 'variantes.xlsx'
mv variantes.txt TEA_Einstein/dados/
mv variantes.xlsx TEA_Einstein/dados/
```

# 1.3 - Download do Annovar do Github
```bash
wget https://github.com/Varstation/POS-BIOINFO/raw/master/annovar/annovar.zip \
unzip annovar.zip
```

# 2 - Separar amostras entre:

*   Homens TEA
*   Mulheres TEA
*   Homens Saudáveis
*   Mulheres Saudáveis

```python
import pandas as pd
df_caminhoExcel = '/content/TEA_Einstein/dados/variantes.xlsx'
df_variantes = pd.read_excel(df_caminhoExcel)
```

```python
output_M_Saud = '/content/TEA_Einstein/dados/SAUD_M.txt'  # 1 and M
output_F_Saud = '/content/TEA_Einstein/dados/SAUD_F.txt'  # 1 and F
output_M_TEA = '/content/TEA_Einstein/dados/TEA_M.txt'  # 2 and M
output_F_TEA = '/content/TEA_Einstein/dados/TEA_F.txt'  # 2 and F
```

```python
txt_file_path = '/content/TEA_Einstein/dados/variantes.txt'
with open(txt_file_path, 'r') as txt_file:
    for line, status, sex in zip(txt_file, df_variantes['Affected_Status'], df_variantes['Sex']):
        if status == 1 and sex == 'Male':
            with open(output_M_Saud, 'a') as file_1M:
                file_1M.write(line)
        elif status == 1 and sex == 'Female':
            with open(output_F_Saud, 'a') as file_1F:
                file_1F.write(line)
        elif status == 2 and sex == 'Male':
            with open(output_M_TEA, 'a') as file_2M:
                file_2M.write(line)
        elif status == 2 and sex == 'Female':
            with open(output_F_TEA, 'a') as file_2F:
                file_2F.write(line)

print("Separação concluída.")
```
