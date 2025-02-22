# Computação em Nuvem


???+ info inline end "Edição"

    2024.2: roteiros 

    2025.1: projeto


## KIT-U

- Giovana Cassoni Andrade
- Lucas Hix

## Entregas

- [ ] Roteiro 1 - Data 26/03/2025
- [ ] Roteiro 2 - Data 26/03/2025
- [ ] Roteiro 3
- [ ] Roteiro 4
- [ ] Projeto


## Textos e imagens dos roteiros

Para ver os roteiros originais em PDF, foi utilizado o formato abaixo, com a mensagem em display dentro dos colchetes [ ] e a localização do pdf nas pastas dentro dos parênteses ( ):

```
[Roteiro X - PDF](./Roteiro_X_de_Cloud.pdf)
```

Para obter os textos, legendas e imagens dos roteiros para colocá-los na página, foram utilizados os comandos a seguir:

``` sh
brew install poppler
pdfimages -png Roteiro_X_de_Cloud.pdf imgs/roteiroX
pip install pdfminer.six
pdf2txt.py Roteiro_X_de_Cloud.pdf > Roteiro_X_de_Cloud.md
```