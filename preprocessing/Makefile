CFLAGS = -std=c99 -g -Wall -Wextra -Werror -O3 -I.

# M4RI
CFLAGS += -I$(HOME)/prison/m4ri/install/include
LDFLAGS += -L$(HOME)/prison/m4ri/install/lib


all: programs

programs: splitter dict_checker sorter merger hash_checker forger slicer # indexer index_checker

hasher.o: hasher.c hasher.h preprocessing.h ../types.h

splitter.o: splitter.c sha256.h hasher.h preprocessing.h
splitter.o: CC = mpicc
splitter: preprocessing.o sha256.o hasher.o splitter.o
splitter: CC = mpicc

sorter.o: hasher.h sorter.c preprocessing.h

slicer.o: preprocessing.h
slicer: LDLIBS += -lm4ri -lm

dict_checker.o: dict_checker.c hasher.h preprocessing.h
dict_checker: preprocessing.o sha256.o hasher.o dict_checker.o

forger.o: forger.c hasher.h
forger: sha256.o hasher.o forger.o

sha256.o: sha256.c sha256.h


clean:
	rm -f *.o *.pdf *.aux *.bbl *.blg *.log *.out *.tex
	rm -f splitter dict_checker sorter merger hash_checker indexer index_checker forger
	rm -f splitter.pdf dict_checker.pdf sorter.pdf merger.pdf hash_checker.pdf indexer.pdf index_checker.pdf forger.pdf
	rm -f hasher.c splitter.c dict_checker.c sorter.c merger.c hash_checker.c indexer.c index_checker.c forger.c slicer.c preprocessing.h
