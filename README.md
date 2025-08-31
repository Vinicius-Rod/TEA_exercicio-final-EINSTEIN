Trabalho de Conclusão de Curso - Curso de Bioinformática aplicada a Genomica Medica - HIAE

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

# 3 - Download das bases de dados do Annovar para o hg19 (genoma de referência) e Anotação

*   refGene
*   clinvar_20221231
*   revel
*   intervar

```bash
#hg19 - refGene
perl annovar/annotate_variation.pl -buildver hg19 -downdb -webfrom annovar refGene annovar/humandb/

#hg19 - clinvar_20221231
perl annovar/annotate_variation.pl -buildver hg19 -downdb -webfrom annovar clinvar_20221231 annovar/humandb/

#hg19 - revel
perl annovar/annotate_variation.pl -buildver hg19 -downdb -webfrom annovar revel annovar/humandb/

#hg19 - intervar
perl annovar/annotate_variation.pl -buildver hg19 -downdb -webfrom annovar intervar_20180118 annovar/humandb/
```

# 3.1 - Anotação dos Saudáveis

```bash
#Anotação dos SAUD_ Masculinos
perl /content/annovar/table_annovar.pl /content/TEA_Einstein/dados/SAUD_M.txt \
annovar/humandb/ -buildver hg19 \
-out /content/TEA_Einstein/dados/annovar/amostra_SAUD_M \
-protocol revel,refGene,clinvar_20221231,intervar_20180118 -operation f,g,f,f -nastring "."
```

```bash
#Anotação dos SAUD_ Femininos
perl /content/annovar/table_annovar.pl /content/TEA_Einstein/dados/SAUD_F.txt \
annovar/humandb/ -buildver hg19 \
-out /content/TEA_Einstein/dados/annovar/amostra_SAUD_F \
-protocol revel,refGene,clinvar_20221231,intervar_20180118 -operation f,g,f,f -nastring "."
```

```bash
mv /content/TEA_Einstein/dados/annovar/amostra_SAUD_* /content/TEA_Einstein/dados/annovar/amostra_01 # Mover arquivos da amostra_SAUD_ para o diretório
```

# 3.2 - Anotação dos TEA

```bash
#Anotação dos Autistas Masculinos
perl /content/annovar/table_annovar.pl /content/TEA_Einstein/dados/TEA_M.txt \
annovar/humandb/ -buildver hg19 \
-out /content/TEA_Einstein/dados/annovar/amostra_TEA_M \
-protocol revel,refGene,clinvar_20221231,intervar_20180118 -operation f,g,f,f -nastring "."
```

```bash
#Anotação dos Autistas Femininos
# perl /content/annovar/table_annovar.pl /content/TEA_Einstein/dados/TEA_F.txt \
annovar/humandb/ -buildver hg19 \
-out /content/TEA_Einstein/dados/annovar/amostra_TEA_F \
-protocol revel,refGene,clinvar_20221231,intervar_20180118 -operation f,g,f,f -nastring "."
```

```bash
mv /content/TEA_Einstein/dados/annovar/amostra_TEA_* /content/TEA_Einstein/dados/annovar/amostra_02 # Mover arquivos da amostra_TEA_ para o diretório
```

# 3.3 - Converter arquivos para .csv

```bash
import pandas as pd
df_saud_fem = pd.read_csv('/content/TEA_Einstein/dados/annovar/amostra_01/amostra_SAUD_F.hg19_multianno.txt', sep='\t')
df_saud_fem.to_csv('/content/TEA_Einstein/dados/amostra_SAUD_F.hg19_multianno.csv', index=False)
display(df_saud_fem)

import pandas as pd
df_saud_masc = pd.read_csv('/content/TEA_Einstein/dados/annovar/amostra_01/amostra_SAUD_M.hg19_multianno.txt', sep='\t')
df_saud_masc.to_csv('/content/TEA_Einstein/dados/amostra_SAUD_M.hg19_multianno.csv', index=False)
display(df_saud_masc)

import pandas as pd
df_tea_fem = pd.read_csv('/content/TEA_Einstein/dados/annovar/amostra_02/amostra_TEA_F.hg19_multianno.txt', sep='\t')
df_tea_fem.to_csv('/content/TEA_Einstein/dados/amostra_TEA_F.hg19_multianno.csv', index=False)
display(df_tea_fem)

import pandas as pd
df_tea_masc = pd.read_csv('/content/TEA_Einstein/dados/annovar/amostra_02/amostra_TEA_M.hg19_multianno.txt', sep='\t')
df_tea_masc.to_csv('/content/TEA_Einstein/dados/amostra_TEA_M.hg19_multianno.csv', index=False)
display(df_tea_masc)
```

```bash
mkdir TEA_Einstein/dados/dados_brutos
mkdir TEA_Einstein/dados/dataframes
```

```bash
mv /content/TEA_Einstein/dados/amostra* /content/TEA_Einstein/dados/dataframes
mv /content/TEA_Einstein/dados/variantes* /content/TEA_Einstein/dados/dados_brutos
mv /content/TEA_Einstein/dados/TEA_* /content/TEA_Einstein/dados/dados_brutos
mv /content/TEA_Einstein/dados/SAUD_* /content/TEA_Einstein/dados/dados_brutos
```
