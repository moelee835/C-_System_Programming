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
clean이라는 파일이 있는데, dependency가 없기 때문.  
`.PHONY`로 clean을 등록하면 해결할 수 있다.

```
.PHONY: clean
clean:
    rm -f $(OBJS) main
```

이제 `make clean`하면 clean 파일 유무 관계 없이 명령 실행함.

> `PHONY`는 phony target(실제 파일이 아니다) 라는 의미.

### pattern 사용

```
foo.o: foo.h foo.cc
    $(CC) $(CXXFLAGS) -c foo.cc

bar.o: bar.h bar.cc
    $(CC) $(CXXFLAGS) -c bar.cc
```

위 내용에서 겹치는 패턴을 합친다.

```
%.o : %.h %.cc
    $(CC) $(CXXFLAGS) -c $< // 일부러 어렵게 하려고 한게 아니라 이렇게만 쓸 수 있다.
```
> 여기서 %< 는 첫번째 (왼쪽)의 prerequisite를 의미한다.

패턴(%)는 target, prerequisites 부분에만 사용 가능하다.

![패턴](https://modoocode.com/img/cpp/a.1.2.png)

`$?` : 타겟보다 최신인 의존 파일들
`$+` : $^ 와 비슷하지만, 중복된 이름까지 모두 포함한다.

### Compiler 도움을 받아서 자동으로 prerequisites를 추가하기.

```
main.o : main.cc foo.h bar.h
    $(CC) $(CXXFLAGS) -c $<
```

위와 같은 상황이라면, %.cc %.h 를 적용할 수 없다.  
include 하는 헤더파일 이름이 소스 파일과 다를 수 있다. 그리고 헤더파일은 새롭게 추가되거나 삭제 될 수 있다.  

clang++로 컴파일 할 때, -MD 옵션을 추가하면, .d 파일이 생성되며 안에는 의존파일 목록이 출력되어 있다.  

이제 .d 파일을 Make 파일에 포함하는 방법은 include ~~.d 를 Makefile에 추가하는 것이다.

```
include main.d
```

> Makefile에서 사용 가능한 함수
> 1. wildcard : `$(wildcard $(SRC_DIR)/*.cc)`
> 2. notdir : `$(notdir $(경로 목록))`
> 3. pathsubst : `$(patsubst 패턴, 치환 후 형태, 변수)` 변수로 들어온 것들을 패턴에 맞춰 치환한다.

만약 헤더 파일을 별도 디렉터리(include)에 저장해둔다면, 컴파일 옵션에 -Iinclude 옵션을 추가하면, 헤더파일 경로를 찾아낸다.

### 멀티코어로 make하기
GCC나 커널을 컴파일할 경우 매우 오래걸리는데, 여러 코어를 사용해 빨리 진행 가능하다.

```
> make -j8
```
위는 8코어로 make하게 된다.

```
> make -j$(nproc)
```
시스템 코어 개수를 입력하는 방법은 위와 같다.

# CMake로 빌드하기

프로젝트 크기가 커지는 경우, 여러 플랫폼에 배포해야 하는 경우, make만으로는 어렵다.

## CMake
**빌드 파일**을 생성해주는 프로그램.

CMake 빌드파일 생성 -> Makefile 또는 .ninja 파일을 생성.

반드시 최상위 디렉터리에 CMakeLists.txt 파일이 있어야 한다.
CMakeLists.txt 파일에는 빌드 파일을 만들기 위한 정보가 들어있다.

반드시 포함되어야 하는 내용.

```
# 최소 버전
cmake_minimum_required(VERSION 3.11)

# 프로젝트 버전
project(
    WoonyCode, # 프로젝트 이름. 꼭 필요.
    VERSION 0.1
    DESCRIPTION "SAMPLE"
    LANGUAGES CXX # C = C, C++ = CXX, Default = C, CXX
)
```

그외에는 CUDA, OBJC, OBJCXX, Fortran 등이 있다.

SAMPLE
```
cmake_minimum_required(VERSION 3.5)

project(
    WOONYCode
    VERSION 0.0.1
    DESCRIPTION "sample"
    LANGUAGES CXX
)

add_executable (program main.cc)
```

권장 사항으로 build/ 디렉터리 생성하여 빌드 파일을 만드는 게 좋다.

```
> cd build
> cmake ..
CMake Deprecation Warning at CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.10 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value.  Or, use the <min>...<max> syntax
  to tell CMake that the project requires at least <min> but has been updated
  to work with policies introduced by <max> or earlier.


-- The CXX compiler identification is AppleClang 21.0.0.21000101
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done (0.4s)
-- Generating done (0.0s)
-- Build files have been written to: /Users/woony/dev/daily_study/cpp/cmake_test_build/cmake_test/build
```

지금 보니까, 생성되는 파일이 매우 많아서 꼭 build 디렉터리에 만드는게 낫겠다.

build 디렉터리에서 make를 실행하면 빌드 된다.

출력물을 보니, "program" 이라는 이름으로 생성되었다.

생성할 실행 파일을 추가하는 명령은 add_executable 이다.
