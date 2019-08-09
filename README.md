# tess_e13b_training
Describes how to train E13B or MICR font for tesseract.
## E13B or MICR font
You should find a font somewhere.  The name of mine is E13Bnsd.  I'm sorry but I can't put it here because it isn't mine or free, either.  Please change the font name in the commands below to your font.

Symbols are mapped as follows in my case:
* Transit symbol: :
* On-un symbol: <
* Amount symbol: ;
* Dash symbol: =

Please change the symbols in the training text according to your font.
## Training text
It's the file, eng.training_e13b_text.  It has about 20,000 lines and is divided to 3 parts.
### Lines 1 to 4014
These lines are typical encoding patterns in real life.  I guess these lines should be changed to your case.
### Lines 4015 to 12718
I borrowed these lines from https://github.com/Shreeshrii/tessdata_shreetest/eng.digits.training_text. I changed non-numerical characters to MICR symbols.
### Lines 12719 to end (20318)
I randomly generated lines like shree's text above but with more symbols than numerics.  Each MICR symbol is consit of multiple particles.  They are dificult characters to recognize.  So I thought I'd need more of them.  Also, I put spaces here and there because word boxing seems to be a key functionality of tesseract.  One long word in one line seems to give less training.
## Training from full eng.traineddata
## Training from scratch
## Phantom Characters
### MICR symbols
### Spaces
--oldtraineddata
5,000 iterations to 10,000 iterations
## Useful commands
