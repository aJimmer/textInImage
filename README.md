#Text In Image
Author: Angel Jimenez
Class: CPSC-353
Professor: Reza

###Description: 
Steganograhpy is the study of hiding data within data. Steganography can be used to exchange hidden messages which is what the implementation of this program does. 

##Architecture:
The program uses 2 main functions/methods: 
* retr() to retreive text
* hide() to encode text

Helper functions are used to process requests:
* check_format() to veify correct img files or convert to jpg
* str_to_bin() to convert strings to bin strings
* dec_to_bin() to convert integers to binary
* binary_to_str() to convert bin strings to text strings
* rotate() to pad binary strings

Encode/decode are used to manipulate least significant bits in image pixels:
* encode()
* decode()

##Execute:

Encode an image:
`python textInImage.py -e 'message to encode' -i fileToOpen.jpg`

or use the source.txt to enter the text

`python textInImage.py -e source.txt -i fileToOpen.jpg`

Decode an image:

python textInImage.py -d fileToDecode.png -i fileToDecode.png
