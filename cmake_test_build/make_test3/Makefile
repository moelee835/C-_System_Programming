CC = clang++
CXXFLAGS = -Wall -O2
OBJS = foo.o bar.o main.o

%.o : %.cc
	$(CC) $(CXXFLAGS) -c $<

main : $(OBJS)
	echo $?
	$(CC) $(OBJS) -o main

.PHONY: clean
clean :
	rm -f $(OBJS) main

-include $(OBJS:.o=.d)