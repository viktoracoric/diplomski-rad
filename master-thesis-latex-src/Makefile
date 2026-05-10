TEX = xelatex
BIB = bibtex
MAIN = foi

.PHONY: all clean

.IGNORE:

all: $(MAIN).pdf

$(MAIN).pdf: $(MAIN).tex
	$(TEX) -interaction=nonstopmode $(MAIN).tex
	-$(BIB) $(MAIN) || true
	$(TEX) -interaction=nonstopmode $(MAIN).tex
	$(TEX) -interaction=nonstopmode $(MAIN).tex

clean:
	rm -f *.aux *.log *.out *.toc *.pdf *.bbl *.blg *.xdv *.lof *.lot
