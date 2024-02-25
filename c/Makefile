name := render

CFLAGS ?= -O3
LDFLAGS ?=

CF := -Wall -Wextra $(CFLAGS)
LF = -lm -lpthread $(LDFLAGS)
CC ?= gcc

.PHONY: clean cleanall run profile pgo check_cc

$(name): $(name).c
	$(CC) -o $(name) $(name).c $(LF) $(CF)

$(name).s: $(name).c
	$(CC) -o $(name).s $(name).c $(CF)

$(name).g: check_cc $(name).c
	clang -o $(name).g $(name).c $(LF) $(CF) -fprofile-instr-generate

$(name).profraw: check_cc $(name).g
	LLVM_PROFILE_FILE=$(name).profraw ./$(name).g

$(name).profdata profile: check_cc $(name).profraw
	llvm-profdata merge -output=$(name).profdata $(name).profraw

pgo: check_cc $(name).profdata
	clang -o $(name) $(name).c $(LF) $(CF) -fprofile-instr-use=$(name).profdata

clean:
	rm -f $(name){,.{s,g,profraw,profdata}}

cleanall: clean
	rm -f out.ppm

out.ppm run: $(name)
	./$(name)

check_cc:
	@if [ ${CC} != "clang" ] ; then \
		echo ;\
		echo  "error: llvm/clang are required because llvm-profdata is used for PGO" ;\
		echo ;\
		exit 1;\
	fi
