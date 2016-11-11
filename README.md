# phd-symp

## Collaborate

Edit the bianca.md file using whichever text editor you want (Vim, Sublime, Word). 
You can even edit it online, directly on github: https://github.com/PapersOfMathieuNls/phd-symp/edit/master/bianca.md

The only thing that matters is to save it in text format.

## Build the pdf

The pdf generation is based on [Pandoc](http://pandoc.org/). Pandoc transforms the markdown to latex and then, to pdf. 
Here's the command, assuming pandoc is installed on your system and that you are on the Phd-Symp directory :

### For the paper

```bash
pandoc -s -S --filter pandoc-citeproc --number-sections --template="config/default.latex" -o symposium.md.pdf symposium.md
```
