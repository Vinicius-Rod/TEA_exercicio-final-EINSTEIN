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

