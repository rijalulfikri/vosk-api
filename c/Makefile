CFLAGS=-I../src
LDFLAGS=-L../build/lib -lvosk -ldl -lpthread -Wl,-rpath,../build/lib

all: test_vosk test_vosk_speaker

test_vosk: test_vosk.o
	gcc $^ -o $@ $(LDFLAGS)

test_vosk_speaker: test_vosk_speaker.o
	gcc $^ -o $@ $(LDFLAGS)

%.o: %.c
	gcc $(CFLAGS) -c -o $@ $<

clean:
	rm -f *.o *.a test_vosk test_vosk_speaker
