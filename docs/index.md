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
- [ ] Roteiro 3 - Data 28/05/2025
- [ ] Roteiro 4 - Data 28/05/2025
- [ ] Projeto - Data 28/05/2025


## Textos e imagens dos roteiros

Para ver os roteiros originais em PDF, eles estão presentes em suas respectivas abas de roteiro.

Para obter os textos, legendas e imagens dos roteiros para colocá-los nas páginas, foram utilizados os comandos a seguir:

``` sh
$ brew install poppler
$ pdfimages -png Roteiro_X_de_Cloud.pdf imgs/roteiroX
$ pip install pdfminer.six
$ pdf2txt.py Roteiro_X_de_Cloud.pdf > Roteiro_X_de_Cloud.md
```