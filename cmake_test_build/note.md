# Make로 빌드하기.

## Make
Makefile을 읽어서 주어진 방식대로 명령어를 처리함.  
많은 파일들을 명령어 읽어 한 번에 처리함.

---

맥에서 컴파일

### clang++
mac os에서는 LLVM/Clang을 기본적으로 사용함.

내부적으로 전처리(#include 등), 컴파일 (C++ -> Assembly), 어셈블 (Assembly -> Obj file)  
을 거쳐서 .o 파일을 출력한다.

**링킹은 하지 않는다.**

#### 옵션
1. -g : 디버그 정보를 생성. 함수이름, 변수이름, 소스파일, 소스코드 - 기계어 대응 관계  
이를통해 LLDB, GDB, llvm-objdump -S 등이 소스코드와 함께 보여줄 수 있음.

2. -O0 : Optimization level 0. 최적화를 끄는 기능. 소스코드를 번역할 때, 개선 가능한 곳을 최적화하여 컴파일하는 옵션.

3. -c : 링킹 X , 없으면 .out 파일까지 만들어짐.

4. -o : 출력 파일의 이름.

### llvm-objdump
Object File을 읽는 프로그램. 실행파일, object file, library를 분석함.

#### 옵션
1. -d : Disassemble. 기계어를 Assembly로 변환함.

2. -S : Source. 소스코드와 어셈블리를 함께 출력함. 단, 컴파일 옵션이 -g 였어야 함.

3. -h : Section Header를 출력함. 각 섹션 이름, 크기, 주소를 출력.
```
~/dev/daily_study/cpp/cmake_test_build ❯ llvm-objdump -h main.o                                                                                                                                                   23:53:18

main.o: file format mach-o arm64

Sections:
Idx Name             Size     VMA              Type
  0 __text           00000030 0000000000000000 TEXT
  1 __debug_abbrev   0000003e 0000000000000030 DATA, DEBUG
  2 __debug_info     00000039 000000000000006e DATA, DEBUG
  3 __debug_str_offs 00000024 00000000000000a7 DATA, DEBUG
  4 __debug_str      000000a4 00000000000000cb DATA, DEBUG
  5 __debug_addr     00000010 000000000000016f DATA, DEBUG
  6 __debug_names    00000070 0000000000000180 DATA, DEBUG
  7 __compact_unwind 00000020 00000000000001f0 DATA
  8 __debug_line     0000005e 0000000000000210 DATA, DEBUG
  9 __debug_line_str 0000003a 000000000000026e DATA, DEBUG
```
---

foo.h, foo.cc, bar.h, bar.cc, main.cc 가 있을 때, Make 없이 빌드하는 방법.

```
> clang++ -c foo.cc
> clang++ -c bar.cc
> clang++ -c main.cc
> clang++ main.o foo.o bar.o -o main
> main
Foo!
Bar!
```

쉘스크립트 짜서 안하는 이유 : 파일 수정 한 뒤에는 수정한 파일만 재컴파일 하면 되므로.

### Make
주어진 쉘 명령어들을 조건에 맞게 실행하는 프로그램. 조건을 명시한 파일 = Makefile  
터미널에서 make 실행 시, 해당 위치의 Makefile을 찾아서 읽음.

### Makefile
구조
```
target ... : prerequisites ...
<tab>recipe
    ...
    ...
```

1. target
```
> make abc
```
abc가 target 중에 있으면, 이를 찾아서 대응되는 명령 실행.

2. recipes
주어진 타겟을 make할 때 실행할 명령어들.
명령어 쓸 때는 반드시 탭 한 번씩 해준다.
스페이스바 안되고 반드시 탭!

3. prerequisites
make할 때 사용될 파일 목록. 의존 파일(dependency)라고도 함.  
명시된 파일들의 최종 수정 시간이 현재 파일에 대한 명령 실행 시간보다 이전이어야 함.  
만약 더 나중이면 작업하지 않음. (이미 작업되어 있다고 판단하기 때문.)

```
foo.o : foo.h foo.cc
	clang++ -c foo.cc


bar.o : bar.h bar.cc
	clang++ -c bar.cc

main.o : main.cc foo.h bar.h
	clang++ -c main.cc

main : foo.o bar.o main.o
	clang++ main.o foo.o bar.o -o main
```

```
> make main
```

target main을 찾는다.  
foo.o, bar.o, main.o 필요한 파일을 찾아 명령어 실행한다.  
main의 명령어를 실행한다.


이후 foo.cc만 수정하고 다시 실행하면, foo.o의 생성 시간보다 foo.cc의 수정 시간이 더 나중이니 명령을 실행.  
안바꿔도 되는 main.o, bar.o는 패스.

```
❯ make main
clang++ -c foo.cc
clang++ main.o foo.o bar.o -o main
```


### Make 변수

```
CC = clang++

foo.o : foo.h foo.cc
    $(CC) -c foo.cc
```

위의 $(CC)는 clang++로 치환된다. 변수 선언 안된걸 사용하면 빈 문자열로 처리됨.

> TMI. 변수를 변수로 초기화하는 법  
> `B = $(A)` `C = $(B)` 이렇게 선언하면, 이후 A 값이 정해질 떄까지 빈 문자열로 처리됨.
> `B := $(A)` `A = clang++` 이렇게 하면, 선언 당시 값으로 고정되므로, 빈 문자열만 저장함.  
> 일반적으로 = 만 사용해도 충분하다. := 는 자기자신을 수정하는 경우 무한루프 피하는 용으로 사용.


#### .o 파일 제거하기

```
clean :
    rm -f $(OBJS) main
```

이렇게 작성하면 `make clean`으로 지울 수 있지만, clean이라는 파일이 있으면 동작하지 않는다.  
`.PHONY`로 clean을 등록하면 해결할 수 있다.

```
.PHONY: clean
clean:
    rm -f $(OBJS) main
```

이제 `make clean`하면 clean 파일 유무 관계 없이 명령 실행함.