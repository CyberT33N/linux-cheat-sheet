# PDF

## Remove password from protected pdf

```shell
sudo apt install qpdf
qpdf --password=abc123 --decrypt iban-anfrag.pdf iban-anfrag-no-pw.pdf
```

## Convert multiple files to pdf

```shell
sudo apt install imagemagick
convert image1.jpg image2.png text.txt PDFfile.pdf outputFileName.pdf
```

## Pdf to md

- [CyberT33N/paddleocr-cheat-sheet](https://github.com/CyberT33N/paddleocr-cheat-sheet)