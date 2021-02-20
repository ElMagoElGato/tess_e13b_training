# tess_e13b_training
Describes how to train E13B or MICR font for tesseract.  I recommend to use the scratch method to get better result.  Fine tuning from existing eng.traineddata retains its reading capability and it wrongfully drags character bounding around MICR symbols at times.
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
I borrowed these lines from eng.digits.training_text in tessdata_shreetest of Shreeshrii's repository. I changed non-numerical characters to MICR symbols.
### Lines 12719 to end (20318)
I randomly generated lines like shree's text above but with more symbols than numerics.  Each MICR symbol is consit of multiple particles.  They are dificult characters to recognize.  So I thought I'd need more of them.  Also, I put spaces here and there because word boxing seems to be a key functionality of tesseract.  One long word in one line seems to give less training.
## Make training data
First, make training data.  Use --fontlist only to generate data with the e13b font. Also use --training_text to specify the training text.
```
rm -r ~/tesstutorial/e13beval
src/training/tesstrain.sh --fonts_dir /usr/share/fonts --lang eng --linedata_only \
  --noextract_font_properties --langdata_dir ../langdata \
  --tessdata_dir ./tessdata \
  --fontlist "E13Bnsd" --output_dir ~/tesstutorial/e13beval \
  --training_text ../langdata/eng/eng.training_e13b_text
```
## Fine tuning from full eng.traineddata
Second, execute training.  Use --old_traineddata to get the effect as described for fine-tunining on a few characters.
```
rm -r ~/tesstutorial/e13b_from_full
mkdir -p ~/tesstutorial/e13b_from_full
src/training/combine_tessdata -e tessdata/best/eng.traineddata \
  ~/tesstutorial/e13b_from_full/eng.lstm
src/training/lstmtraining --debug_interval -1 \
  --model_output ~/tesstutorial/e13b_from_full/e13b \
  --continue_from ~/tesstutorial/e13b_from_full/eng.lstm \
  --traineddata ~/tesstutorial/e13beval/eng/eng.traineddata \
  --old_traineddata tessdata/best/eng.traineddata \
  --train_listfile ~/tesstutorial/e13beval/eng.training_files.txt \
  --max_iterations 5000 &>~/tesstutorial/e13b_from_full/basetrain.log
```
Finally, make traineddata.
```
src/training/lstmtraining --stop_training \
  --continue_from ~/tesstutorial/e13b_from_full/e13b_checkpoint \
  --traineddata ~/tesstutorial/e13beval/eng/eng.traineddata \
  --model_output ~/tesstutorial/e13b_from_full/e13b.traineddata
```
## Training from scratch
Make training data if it isn't done yet. Then execute training.  5,000 iterations don't seem enough.  10,000 iterations seem good enough.
```
rm -r ~/tesstutorial/e13boutput
mkdir -p ~/tesstutorial/e13boutput
src/training/lstmtraining --debug_interval -1 \
  --traineddata ~/tesstutorial/e13beval/eng/eng.traineddata \
  --net_spec '[1,36,0,1 Ct3,3,16 Mp3,3 Lfys48 Lfx96 Lrx96 Lfx256 O1c111]' \
  --model_output ~/tesstutorial/e13boutput/base --learning_rate 20e-4 \
  --train_listfile ~/tesstutorial/e13beval/eng.training_files.txt \
  --eval_listfile ~/tesstutorial/e13beval/eng.training_files.txt \
  --max_iterations 10000 &>~/tesstutorial/e13boutput/basetrain.log
```
Finally, make traineddata.
```
src/training/lstmtraining --stop_training \
  --continue_from ~/tesstutorial/e13boutput/base_checkpoint \
  --traineddata ~/tesstutorial/e13beval/eng/eng.traineddata \
  --model_output ~/tesstutorial/e13boutput/e13b.traineddata
```
## Phantom Characters
Phantom characters are those that appear in the result text though they don't seem to exist on the original image.
### MICR symbols
The on-us symbol is the biggest problem.  It tends to appear like this:
```
<span class='ocrx_cinfo' title='x_bboxes 1259 902 1262 933; x_conf 98.864532'>&lt;</span>
<span class='ocrx_cinfo' title='x_bboxes 1259 904 1281 933; x_conf 99.018097'>;</span>
```
The first one is a phantom character that starts at the same x axis as the next character and only has 3 point width.  The second one is a real character with appropriate bound box indices.

Good training text basically eliminates these phantom characters.  The --old_traineddata option serves an important role, too.
### Spaces
E13B has some narrow characters while they are printed in the same pitch.  Slightly larger spaces between characters are picked up as word breaks.  Good training text and enouh iterations eliminate these.
### Too many characters
It's a common phenomena across tesseract recognition.  In the case of E13B font, good training text and enough iterations eliminate these.
## Useful commands
Pick all the files in the directory and make hocr output with character boundary boxes.  This can be done easily by using API, too.  See APIExample in tessdoc of tesseract-ocr repository. Locate "Result iterator example" and change RIL_WORD to RIL_SYMBOL, or just "Example of iterator over the classifier choices for a single symbol."
```
ls -1d /imagedir/*|tesseract -l e13b stdin stdout -c lstm_choice_mode=4 -c lstm_choice_amount=0 -c hocr_char_boxes=1 hocr
```
Lists fonts available in the system:
```
text2image --list_available_fonts --fonts_dir=/usr/share/fonts/
```
## Reference site
* Tesseract repository of course: https://github.com/tesseract-ocr
* I consulted the Shreeshrii's repository a lot and found it very helpful: https://github.com/Shreeshrii
