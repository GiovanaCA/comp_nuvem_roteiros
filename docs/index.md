# Computação em Nuvem


???+ info inline end "Edição"

    2024.2: roteiros 

    2025.1: projeto


## KIT-U

- Giovana Cassoni Andrade
- Lucas Hix

## Entregas

- [X] Roteiro 1 - Data 26/03/2025
- [X] Roteiro 2 - Data 26/03/2025
- [X] Roteiro 3 - Data 28/05/2025
- [X] Roteiro 4 - Data 28/05/2025
- [X] Projeto - Data 28/05/2025


## Textos e imagens dos roteiros

Para ver os roteiros originais em PDF, eles estão presentes em suas respectivas abas de roteiro.

Para obter os textos, legendas e imagens dos roteiros para colocá-los nas páginas, foram instaladas a biblioteca poppler e a ferramenta pdfminer.six com:

``` sh
brew install poppler
pip install pdfminer.six
```

E, para cada um dos roteiros, foram utilizados os comandos a seguir:

``` sh
pdfimages -png Roteiro_X_de_Cloud.pdf imgs/roteiroX
pdf2txt.py Roteiro_X_de_Cloud.pdf > Roteiro_X_de_Cloud.md
```