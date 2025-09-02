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

# 4 - Análise

```bash
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```

```bash
TEA_F = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_TEA_F.hg19_multianno.csv')
TEA_M = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_TEA_M.hg19_multianno.csv')
SAUD_F = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_SAUD_F.hg19_multianno.csv')
SAUD_M = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_SAUD_M.hg19_multianno.csv')
```

```bash
genes = ['Pathogenic', 'Likely pathogenic', 'Uncertain significance', 'Likely benign', 'Benign']

# Função para contar a ocorrência de cada gene em um DataFrame
def contar_genes(df, genes):
    gene_count = {}
    for gene in genes:
        gene_count[gene] = df['InterVar_automated'].apply(lambda x: gene in x).sum()
    return gene_count

# Contagem de genes para cada DataFrame
count_TEA_F = contar_genes(TEA_F, genes)
count_TEA_M = contar_genes(TEA_M, genes)
count_SAUD_F = contar_genes(SAUD_F, genes)
count_SAUD_M = contar_genes(SAUD_M, genes)

# Criando DataFrame final
resultados = pd.DataFrame([count_TEA_F, count_TEA_M, count_SAUD_F, count_SAUD_M], index=['Mulheres Autistas', 'Homens Autistas', 'Mulheres Saudáveis', 'Homens Saudáveis']).T

# Configurando paleta de cores amigável para daltônicos
sns.set_palette("colorblind")

# Exibindo gráfico de barras com barras mais largas
ax = resultados.plot(kind='bar', figsize=(10, 6), width=0.8)
plt.title('Significado das Variantes')
plt.xlabel('')
plt.ylabel('Nº de Variantes')

# Configurando a rotação dos rótulos no eixo x
ax.set_xticklabels(ax.get_xticklabels(), rotation=0)

# Adicionando o número da quantidade no topo de cada barra
for p in ax.patches:
    ax.annotate(str(p.get_height()), (p.get_x() + p.get_width() / 2., p.get_height()), ha='center', va='center', xytext=(0, 10), textcoords='offset points')

# Configurando o limite máximo do eixo y em 5000
ax.set_ylim(0, 5000)

# Exibindo o gráfico
plt.show()

"""# Verificando a proporção de variantes patogênicas e provavelmente patogênicas entre homens e mulheres autistas"""
```

```bash
import pandas as pd

# Definindo os genes e a função para contar os genes
genes = ['Pathogenic', 'Likely pathogenic']

def contar_genes(df, genes):
    gene_count = {}
    for gene in genes:
        gene_count[gene] = df['InterVar_automated'].apply(lambda x: gene in x).sum()
    return gene_count


# Contagem de genes para cada DataFrame
count_TEA_F = contar_genes(TEA_F, genes)
count_TEA_M = contar_genes(TEA_M, genes)

# Criando a crosstab apenas para 'Pathogenic' e 'Likely pathogenic' para homens e mulheres
crosstab_TEA_F = pd.DataFrame({'Mulheres Autistas': [count_TEA_F['Pathogenic'], count_TEA_F['Likely pathogenic']]}, index=['Pathogenic', 'Likely pathogenic'])
crosstab_TEA_M = pd.DataFrame({'Homens Autistas': [count_TEA_M['Pathogenic'], count_TEA_M['Likely pathogenic']]}, index=['Pathogenic', 'Likely pathogenic'])

# Exibindo as crosstabs
print("Crosstab para Mulheres Autistas:")
print(crosstab_TEA_F)
print("\nCrosstab para Homens Autistas:")
print(crosstab_TEA_M)

import matplotlib.pyplot as plt
from matplotlib_venn import venn2

# Dados do crosstab
mulheres_autistas_pathogenic = 29
homens_autistas_pathogenic = 122

# Criando o diagrama de Venn
venn2(subsets=(mulheres_autistas_pathogenic, homens_autistas_pathogenic, 0), set_labels=('Mulheres Autistas', 'Homens Autistas'))
plt.title('Diagrama de Venn - Variantes Patogênicas por sexo')
plt.show()

"""Calculando a proporção das variantes em homens e mulheres autistas em relação a amostra total"""
```

```bash
# Calculando o número total de variantes em cada dataframe
total_variantes_homensTea = df_tea_masc.shape[0]
total_variantes_mulheresTea = df_tea_fem.shape[0]
todas_variantes_homens = 11578
todas_variantes_mulheres = 4211

# Calculando a proporção em relação ao número total de amostras para homens e mulheres
proporcao_homens_tea = (total_variantes_homensTea / todas_variantes_homens) * 100
proporcao_mulheres_tea = (total_variantes_mulheresTea / todas_variantes_mulheres) * 100

print("Proporção total de variantes para Homens Autistas: {:.2f}%".format(proporcao_homens_tea))
print("Proporção total de variantes para Mulheres Autistas: {:.2f}%".format(proporcao_mulheres_tea))

# Calculando a diferença entre as proporções
diferenca_proporcao_tea = proporcao_homens_tea - proporcao_mulheres_tea

# Imprimindo a diferença entre as proporções
print("Diferença na proporção entre Homens e Mulheres Autistas: {:.2f}%".format(diferenca_proporcao_tea))

"""Calculando a proporção das variantes em homens e mulheres saudáveis em relação a amostra total"""
```

```bash
# Calculando o número total de variantes em cada dataframe
total_variantes_homensSaud = df_saud_masc.shape[0]
total_variantes_mulheresSaud = df_saud_fem.shape[0]
todas_variantes_homens = 11578
todas_variantes_mulheres = 4211

# Calculando a proporção em relação ao número total de amostras para homens e mulheres
proporcao_homens_saud = (total_variantes_homensSaud / todas_variantes_homens) * 100
proporcao_mulheres_saud = (total_variantes_mulheresSaud / todas_variantes_mulheres) * 100

print("Proporção total de variantes para Homens Saudáveis: {:.2f}%".format(proporcao_homens_saud))
print("Proporção total de variantes para Mulheres Saudáveis: {:.2f}%".format(proporcao_mulheres_saud))

# Calculando a diferença entre as proporções
diferenca_proporcao_saud = proporcao_mulheres_saud - proporcao_homens_saud

# Imprimindo a diferença entre as proporções
print("Diferença na proporção entre Homens e Mulheres Saudáveis: {:.2f}%".format(diferenca_proporcao_saud))
```
```bash
"""Calculando a proporção das variantes patogênicas em homens e mulheres autistas"""



# Definindo o número total de amostras para homens e mulheres autistas
total_amostras_homens_tea = 9868
total_amostras_mulheres_tea = 2298

# Calculando a proporção para mulheres autistas
proporcao_mulheres_pathogenic = crosstab_TEA_F.loc['Pathogenic', 'Mulheres Autistas'] / total_amostras_mulheres_tea * 100

# Calculando a proporção para homens autistas
proporcao_homens_pathogenic = crosstab_TEA_M.loc['Pathogenic', 'Homens Autistas'] / total_amostras_homens_tea * 100


print("Proporção de variantes 'Pathogenic' em relação ao total de amostras:")
print("Para Mulheres Autistas:", "{:.2f}%".format(proporcao_mulheres_pathogenic))
print("Para Homens Autistas:", "{:.2f}%".format(proporcao_homens_pathogenic))

# Proporções em formato decimal
proporcao_mulheres_decimal_tea = 1.26 / 100
proporcao_homens_decimal_tea = 1.24 / 100

# Calculando a diferença entre as proporções
diferenca_proporcao_pathogenic = (proporcao_mulheres_decimal_tea - proporcao_homens_decimal_tea) * 100

print("Diferença na proporção entre Homens e Mulheres Autistas: {:.2f}%".format(diferenca_proporcao_pathogenic))

"""Nós escolhemos utilizar o teste de diferença de proporções de duas amostras para investigar possíveis diferenças nas proporções de variantes patogênicas entre mulheres autistas e homens autistas. Esta escolha é motivada pela necessidade de avaliar se as frequências dessas variantes genéticas estão associadas ao Transtorno do Espectro Autista (TEA) de maneira diferente entre homens e mulheres."""
```

```bash
import numpy as np
from statsmodels.stats.proportion import proportions_ztest

# Proporções das mulheres autistas e homens autistas com a variante 'Pathogenic'
proporcao_mulheres = 1.26 / 100
proporcao_homens = 1.24 / 100

# Tamanho das amostras de mulheres e homens autistas
total_amostras_mulheres_tea = 2298
total_amostras_homens_tea = 9868

# Número de mulheres autistas com a variante 'Pathogenic'
mulheres_pathogenic = int(proporcao_mulheres * total_amostras_mulheres_tea)

# Número de homens autistas com a variante 'Pathogenic'
homens_pathogenic = int(proporcao_homens * total_amostras_homens_tea)

# Contagem total de mulheres autistas
total_mulheres = total_amostras_mulheres_tea

# Contagem total de homens autistas
total_homens = total_amostras_homens_tea

# Aplicar o teste de proporção de duas amostras
count = np.array([mulheres_pathogenic, homens_pathogenic])
nobs = np.array([total_mulheres, total_homens])
stat, p_valor = proportions_ztest(count, nobs)

# Imprimir o resultado do teste
print("Estatística z:", stat)
print("Valor p:", p_valor)

"""Usamos esses artigos para buscar na nossa amostra genes sinápticos de interesse relacionados a TEA:

* Kerche, Leandra & Camparoto, Marjori & Rodrigues, Felipe. (2020). AS ALTERAÇÕES GENÉTICAS E A NEUROFISIOLOGIA DO AUTISMO GENETIC ALTERATIONS AND THE NEUROPHYSIOLOGY OF AUTISM. 15. 1-17. https://revista2.grupointegrado.br/revista/index.php/sabios/article/view/2932


* Jamain S, Quach H, Betancur C, Råstam M, Colineaux C, Gillberg IC, Soderstrom H, Giros B, Leboyer M, Gillberg C, Bourgeron T; Paris Autism Research International Sibpair Study. Mutations of the X-linked genes encoding neuroligins NLGN3 and NLGN4 are associated with autism. Nat Genet. 2003 May;34(1):27-9. doi: 10.1038/ng1136. PMID: 12669065; PMCID: PMC1925054. https://pubmed.ncbi.nlm.nih.gov/12669065/

Nós rodamos uma primeira vez o código e descobrimos que os seguintes genes não apareciam na nossa amostra, então decidimos tirá-los do gráfico: SYN1, RIMS3, CACNB2, KCNMB4, KCNQ5, CDH8, CDH9, PCDH19.
```

```bash
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import re

# Lendo os arquivos CSV
TEA_F = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_TEA_F.hg19_multianno.csv')
TEA_M = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_TEA_M.hg19_multianno.csv')
SAUD_F = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_SAUD_F.hg19_multianno.csv')
SAUD_M = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_SAUD_M.hg19_multianno.csv')

# Lista de genes a serem pesquisados
genes = ['SYN2', 'CACNA1E', 'SCN1A', 'SCN2A', 'SCN3A', 'KCNMA1', 'KCNQ3', 'KCND2', 'NRXN1', 'NLGN3', 'NLGN4X', 'NLGN4Y',
         'CNTNAP2', 'CDH5', 'CDH10', 'CDH11', 'CDH13', 'CDH15', 'PCDHB4', 'PCDH10']

# Função para contar a ocorrência de cada gene em um DataFrame
def contar_genes(df, genes):
    gene_count = {}
    for gene in genes:
        regex = r"\b" + re.escape(gene) + r"\b"
        gene_count[gene] = df['Gene.refGene'].str.contains(regex, regex=True).sum()
    return gene_count

# Contagem de genes para cada DataFrame
count_TEA_F = contar_genes(TEA_F, genes)
count_TEA_M = contar_genes(TEA_M, genes)
count_SAUD_F = contar_genes(SAUD_F, genes)
count_SAUD_M = contar_genes(SAUD_M, genes)

# Criando DataFrame final
resultados = pd.DataFrame([count_TEA_F, count_TEA_M, count_SAUD_F, count_SAUD_M], index=['Mulheres Autistas', 'Homens Autistas', 'Mulheres Saudáveis', 'Homens Saudáveis']).T

# Configurando paleta de cores amigável para daltônicos
sns.set_palette("colorblind")

# Exibindo gráfico de barras com espaçamento entre as barras
ax = resultados.plot(kind='bar', figsize=(10, 6), width=0.8)  # Ajuste a largura conforme necessário
plt.title('Contagem de variantes por gene de interesse')
plt.xlabel('Gene')
plt.ylabel('Nº de Variantes')

# Adicionando os números correspondentes a cada barra (exceto 0)
for p in ax.patches:
    if p.get_height() > 0:
        ax.annotate(str(int(p.get_height())), (p.get_x() + p.get_width() / 2., p.get_height()), ha='center', va='bottom')

# Definindo os intervalos do eixo y para apenas números inteiros múltiplos de 5
plt.yticks(range(0, 21, 5))

plt.show()

"""O resultado inicial relevante deste trabalho destaca a presença significativamente elevada de variantes do gene SCN2A. Após analisar a literatura existente confirmou-se que SCN2A é uma variante fortemente relacionada ao autismo, como indicado no estudo SCN2A channelopathies in the autism spectrum of neuropsychiatric disorders: a role for pluripotent stem cells? Mol Autism. 2020 Apr 7;11(1):23. doi: 10.1186/s13229-020-00330-9. PMID: 32264956; PMCID: PMC7140374. https://pubmed.ncbi.nlm.nih.gov/32264956/

Tirar a proporção entre autistas homens e mulheres, proporção entre homens saudáveis e mulheres saudáveis. Tirar a proporção de variantes do gene SCN2A.  
Avaliar quais são as trocas que estão ocorrendo, avaliar a localização gênica nas variantes.
```
```bash
# Definindo o número total de amostras para homens e mulheres autistas
total_amostras_homens_tea = 9868
total_amostras_mulheres_tea = 2298

# Calculando a proporção DA VARIANTE SCN2A EM HOMENS AUTISTAS
proporcao_homensTEA_SCN2A = 16 / total_amostras_homens_tea * 100

# Calculando a proporção para homens autistas
proporcao_mulheresTEA_SCN2A = 5 / total_amostras_mulheres_tea * 100

# Exibindo as proporções em porcentagem
print("Proporção de variantes 'SCN2A' em relação ao total de amostras dos pacientes TEA:")
print("Para Mulheres Autistas:", "{:.2f}%".format(proporcao_mulheresTEA_SCN2A))
print("Para Homens Autistas:", "{:.2f}%".format(proporcao_homensTEA_SCN2A))

"""# Priorização de variantes patogênicas"""

print(SAUD_M['InterVar_automated'])
```
```bash
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Lendo os arquivos CSV
TEA_F = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_TEA_F.hg19_multianno.csv')
TEA_M = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_TEA_M.hg19_multianno.csv')
SAUD_F = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_SAUD_F.hg19_multianno.csv')
SAUD_M = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_SAUD_M.hg19_multianno.csv')

# Lista de genes a serem pesquisados
genes = ['CACNA1E','SCN1A','SCN2A','KCNQ3','NRXN1']

# Função para contar a ocorrência de cada gene em um DataFrame
def contar_genes(df, genes):
    gene_count = {}
    for gene in genes:
        regex = r"\b" + re.escape(gene) + r"\b"
        gene_count[gene] = df['Gene.refGene'].str.contains(regex, regex=True).sum()
    return gene_count

# Função para contar a ocorrência de cada gene em um DataFrame
def contar_genes(df, genes):
    clnsig_values = ['Pathogenic', 'Likely pathogenic']

    gene_count = {}
    for gene in genes:
        gene_count[gene] = df[(df['InterVar_automated'].isin(clnsig_values)) & (df['Gene.refGene'].apply(lambda x: gene in x))].shape[0]
    return gene_count

# Contagem de genes para cada DataFrame
count_TEA_F = contar_genes(TEA_F, genes)
count_TEA_M = contar_genes(TEA_M, genes)
count_SAUD_F = contar_genes(SAUD_F, genes)
count_SAUD_M = contar_genes(SAUD_M, genes)

# Criando DataFrame final
resultados = pd.DataFrame([count_TEA_F, count_TEA_M, count_SAUD_F, count_SAUD_M], index=['Mulheres Autistas', 'Homens Autistas', 'Mulheres Saudáveis', 'Homens Saudáveis']).T

# Configurando paleta de cores amigável para daltônicos
sns.set_palette("colorblind")

# Exibindo gráfico de barras com barras mais largas
ax = resultados.plot(kind='bar', figsize=(10, 6), width=0.8)
plt.title('Contagem de variantes com genes de interesse (InterVar Pathogenic ou Likely Pathogenic)')
plt.xlabel('Gene')
plt.ylabel('Nº de Variantes')

# Adicionando os números correspondentes a cada barra (exceto 0)
for p in ax.patches:
    if p.get_height() > 0:
        ax.annotate(str(int(p.get_height())), (p.get_x() + p.get_width() / 2., p.get_height()), ha='center', va='bottom')

plt.show()

"""Apenas homens e mulheres diagnosticados com TEA possuem variantes patogênicas ou provavelmente patogênicas nos genes CACNA1E, SCN1A, SCN2A, KCNQ3 e NRXN1"""

# Definindo o número total de amostras para homens e mulheres autistas
total_amostras_homens_tea = 9868
total_amostras_mulheres_tea = 2298

# Calculando a proporção DA VARIANTE SCN2A EM HOMENS AUTISTAS
proporcao_homensTEApatho_SCN2A = 5 / total_amostras_homens_tea * 100

# Calculando a proporção para homens autistas
proporcao_mulheresTEApatho_SCN2A = 2 / total_amostras_mulheres_tea * 100

# Exibindo as proporções em porcentagem
print("Proporção de variantes 'SCN2A' em relação ao total de amostras dos pacientes TEA:")
print("Para Mulheres Autistas:", "{:.2f}%".format(proporcao_mulheresTEApatho_SCN2A))
print("Para Homens Autistas:", "{:.2f}%".format(proporcao_homensTEApatho_SCN2A))
```
```bash
"""# Qual a variante patogênica ou provavelmente patogênica mais comum no gene SCN2A?"""


import pandas as pd

# Leitura dos dataframes
TEA_F = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_TEA_F.hg19_multianno.csv').copy()
TEA_M = pd.read_csv('/content/TEA_Einstein/dados/dataframes/amostra_TEA_M.hg19_multianno.csv').copy()

# Filtrando os valores onde a coluna Gene.refGene é igual a 'SCN2A' e InterVar_automated é igual a 'Pathogenic' ou 'Likely Pathogenic'
TEA_F = TEA_F[(TEA_F['Gene.refGene'] == 'SCN2A') & (TEA_F['InterVar_automated'].isin(['Pathogenic', 'Likely pathogenic']))].copy()
TEA_M = TEA_M[(TEA_M['Gene.refGene'] == 'SCN2A') & (TEA_M['InterVar_automated'].isin(['Pathogenic', 'Likely pathogenic']))].copy()

# Função para combinar os valores de cada linha
def combine_values(row):
    return ':'.join(row.astype(str))

# Aplicação da função para combinar os valores de cada linha
TEA_F['Combined'] = TEA_F[['Chr', 'Start', 'End', 'Ref', 'Alt','Func.refGene','ExonicFunc.refGene']].apply(combine_values, axis=1)
TEA_M['Combined'] = TEA_M[['Chr', 'Start', 'End', 'Ref', 'Alt','Func.refGene','ExonicFunc.refGene']].apply(combine_values, axis=1)

# Contagem de ocorrências de cada combinação única de valores em cada dataframe
TEA_F_counts = TEA_F['Combined'].value_counts()
TEA_M_counts = TEA_M['Combined'].value_counts()

# Concatenando as contagens de cada dataframe
combination_counts = pd.concat([TEA_F_counts, TEA_M_counts], axis=1, keys=['TEA_F', 'TEA_M']).fillna(0)

# Adicionando coluna de soma
combination_counts['Total'] = combination_counts.sum(axis=1)

# Adicionando linha de total
combination_counts.loc['Total'] = combination_counts.sum()

# Definindo o formato de exibição dos números inteiros
pd.options.display.float_format = '{:,.0f}'.format

# Exibição da tabela usando o display do Pandas
display(combination_counts)
```
```bash
#Indivíduos masculinos que possuem a variante 2:166201311:C:T no gene SCN2A


import pandas as pd

# Caminho do arquivo Excel no Google Drive
caminho_arquivo = '/content/TEA_Einstein/dados/dados_brutos/variantes.xlsx'

# Carrega o arquivo Excel em um DataFrame do pandas
df = pd.read_excel(caminho_arquivo)

# Determina a coluna e o valor que você quer filtrar
coluna_alvo = 'Variant'
valor_alvo = '2:166201311:C:T'

# Filtra as linhas do DataFrame onde a coluna_alvo tem o valor_alvo
linhas_filtradas = df[df[coluna_alvo] == valor_alvo]

# Mostra as linhas filtradas usando display
display(linhas_filtradas)

"""o gene *SCN2A* é um gene de canal de sódio tipo 2, presente no *locus* 2q24.3, cuja função está relacionada com a excitabilidade neuronal


Podemos perceber que a variante mais comum anotada como patogênica ou provavelente patogênica sobre o gene *SCN2A* é a mutação:

2:166201311:C:T

Essa mutação ocorre no cromossomo 2, na posição 166201311 e nas duas ocorrências é uma troca de uma citosina por uma timina e é uma mutação não sinônima. Essa mutação está anotada como provavelmente patogênica pelo INTERVAR e patogênica pelo CLINSIG.

Essa mutação ocorreu somente na amostra de Homens diagnosticados com TEA.
```
