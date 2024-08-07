CC = gcc
PROGS = linuxxdoom
LDLIBS = -lm -lXext

PKGS += x11
ifdef PKGS
	LDLIBS += $(shell pkg-config --libs $(PKGS))
	CFLAGS += $(shell pkg-config --cflags $(PKGS))
endif

SRCS = $(wildcard *.c)
OBJS = $(SRCS:.c=.o)
CFLAGS += -Werror

CFLAGS += -Wno-error=implicit-int \
          -Wno-error=int-to-pointer-cast \
          -Wno-error=pointer-to-int-cast \
          -Wno-error=deprecated-declarations \
          -Wno-error=format-overflow
CFLAGS += -DNORMALUNIX -DLINUX

# Enable the sound server:
CFLAGS += -DSNDSERV

MAKEFLAGS=" -j $(nproc)"

# FINAL=y
ifeq ($(FINAL), y)
	LDFLAGS += -s
	CFLAGS += -DNDEBUG -g0 -O2
	CFLAGS += -Wpedantic -Wextra -Wmaybe-uninitialized
	CFLAGS += -W -Wformat=2 -Wpointer-arith -Wbad-function-cast
	CFLAGS += -Wstrict-prototypes -Wmissing-prototypes -Winline
	CFLAGS += -Wdisabled-optimization -Wfloat-equal -Wall
else
	# ASAN=y
	CFLAGS += -g3 -DDEBUG
	ifeq ($(ASAN), y)
		CFLAGS += -fsanitize=address -O1 -fno-omit-frame-pointer
		LDLIBS += -fsanitize=address
	endif
endif

.PHONY: all debug run cppcheck valgrind xephyr clean
all: $(PROGS)

debug:
	@echo "*** Debug info:"
	@echo "Source-files:" $(SRCS)
	@echo "Object-files:" $(OBJS)
	@echo "Compiler-flags:" $(CFLAGS)
	@echo "Linker-flags:" $(LDFLAGS)
	@echo "Linker-libs:" $(LDLIBS)

$(PROGS): $(OBJS)
	$(CC) $(LDFLAGS) $^ $(LDLIBS) -o $@

xephyr:
	@Xephyr :2 -ac -screen 640x480x8

run: $(PROGS)
	@DISPLAY=:2 ./$(PROGS) -2

cppcheck:
	@echo "*** Static code analysis"
	@cppcheck $(shell find . -name "*.[ch]") \
		--verbose --enable=all -DDEBUG=1 \
		--suppress=missingIncludeSystem \
		--suppress=unknownMacro \
		--suppress=unusedFunction

valgrind: $(PROGS)
	@valgrind \
		--tool=memcheck \
		--leak-check=yes \
		--show-reachable=yes \
		--num-callers=20 \
		--track-fds=yes \
		./$(PROGS)

clean:
	$(RM) $(PROGS) $(OBJS)
