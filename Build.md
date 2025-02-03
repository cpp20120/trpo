
# Гайд по сборке C/C++ для начинающих

Исходный C++ файл который вы пишете в .c и .cpp это всего лишь код, его нельзя запустить напрямую. Чтобы из .c/.cpp получить .exe/.dll(на windows) .out, .so(на linux) его необходимо скомпилировать.

Работы будут выполняться на linux поэтому вот сетпа рабочего окружения под него:

## Нужные пакеты на linux

Сохраните это как setup_script.sh
потом выполните ```chmod +x setup_script.sh | ./setup_script.sh```
```bash
#!/bin/bash
# Проверка типа дистрибутива
#!/bin/bash
if [ -x "$(command -v apt-get)" ]; then
    sudo add-apt-repository contrib
    sudo add-apt-repository non-free
    sudo apt-get update
    sudo apt-get install -y  build-essential neofetch git clang clang-tools mold clang-format gcc cmake ninja-build lld lldb valgrind libgtest-dev lcov gcovr python3-pip doxygen neovim qtbase5-dev qt6-base-dev libglfw3 libglfw3-dev glew-utils libglew-dev libglm-dev libvulkan1 vulkan-validationlayers glslang-dev spirv-tools spirv-cross libsdl2-dev libsfml-dev
elif [ -x "$(command -v dnf)" ]; then
    sudo dnf install -y dnf5
    sudo dnf5 install -y https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
    sudo dnf5 install -y https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
    sudo dnf5 install -y  @development-tools neofetch git clang clang-tools-extra compiler-rt mold gcc cmake ninja-build lld lldb valgrind lcov gcovr python3 python3-pip gtest doxygen neovim SFML SFML-devel SDL2-devel SDL2 qt5-qtbase-devel qt5-qtbase qt6-core qt6-qtbase qt6-qtbase-devel qt6-qtmultimedia glfw glm-devel glew vulkan-headers vulkan-loader vulkan-tools vulkan-volk-devel glslang spirv-tools spirv-llvm-translator
elif [ -x "$(command -v pacman)" ]; then
    sudo pacman -Syyu --noconfirm  base-devel neofetch neovim python python-pip lua git clang mold compiler-rt gcc cmake doxygen ninja make lld lldb valgrind gcov gcovr lcov gtest qt5-base qt5-multimedia qt5-quick3d qt6-tools qt6-quick3d qt6-multimedia glfw glew glm vulkan-extra-layers vulkan-extra-tools vulkan-headers vulkan-tools vulkan-validation-layers spirv-llvm-translator sfml sdl2_image sdl2-compat
elif [ -x "$(command -v brew)" ]; then
    brew install  xcodebuild neofetch neovim python3 git clang cmake doxygen ninja make lld lldb valgrind lcov gcovr qt5 qt6 glfw glew glm vulkan-headers vulkan-loader vulkan-tools vulkan-extenstionlayer vulkan-validationlayer spirv-cross spirv-headers spirv-llvm-translator xcode-build-server googletest sfml sdl2 sdl2_image
else
    echo "Не удалось определить дистрибутив и установщик пакетов."
    exit 1
fi


```
gcc или clang на выбор

## Процесс компиляции C++ программы:

### Шаг 1: Препроцессинг

Препроцессинг - это первый этап компиляции, где препроцессор обрабатывает исходный код до самой компиляции. На этом этапе выполняются следующие действия:

Обработка директив препроцессора: ```#include``` вставляет содержимое указанного файла прямо в код. Макросы объявленные при помощи ```#define``` подставляются во всех местах в коде где они были использованы. Так же на этом этапе удаляются комментарии.

```cpp
// Пример
#include  <iostream>  //подключение заголовочного файла ввода вывода в C++
// будет произведена замена PI на 3.14159 во всех частях кода
#define PI 3.14159

int main()  {
	std::cout <<"hello"  << PI;
	return  0;
}
```

Чтобы получить файл который получится после препроцессинга можно воспользоваться флагом -E в компиляторе

```gcc/clang -E``` 
```clang++ -E main.cpp -o main.ii```

[Результат на godbolt](https://godbolt.org/z/h6417eh51)

### Шаг 2: Компиляция
Компиляция - это второй этап, на котором препроцессированный код преобразуется в объектные файлы.
Компиляция включает в себя преобразование препроцессированного кода на языке программирования в набор инструкций ассемблера. Этот этап создает объектные файлы (.o или .obj), содержащие машинный код, который представляет собой набор инструкций для конкретной архитектуры.(В примерах используется g++ но вы можете использовать clang++ просто введя вместо g++ - clang++)

Пример использования компилятора GCC для компиляции исходного файла:

```bash 
g++ -c main.cpp -o main.o
``` 

В данном примере:

-   `g++` - это вызов компилятора GCC для языка C++.
-   `-c` - флаг, указывающий компилятору создать только объектный файл.
-   `main.cpp` - имя исходного файла.
-   `-o main.o` - опция, указывающая имя выходного объектного файла.

### Шаг 3: Линковка

Линковка - это процесс объединения нескольких объектных файлов в один исполняемый файл. На этом этапе также происходит разрешение ссылок между различными частями программы.

Пример использования компилятора GCC для сборки объектных файлов в исполняемый файл:

`g++ main.o -o hello` 

В данном примере:

-   `g++` - вызов компилятора GCC для языка C++.
-   `main.o` - объектный файл, который нужно включить в сборку.
-   `-o hello` - опция, указывающая имя выходного исполняемого файла.

Теперь у вас есть исполняемый файл `hello`, который можно запустить:
```shell
./hello
```

Данные инструкции будут удобны если у нас 1 файл который нужно компилировать, но если файлов больше и/или код использует библиотеки то удобнее использовать системы сборки например make*.

*  make не является  системой сборки только для C/C++ это утилита автоматизирующая процесс преобразования файлов из одной формы в другую. Но чаще всего используется как билд система для C/C++.

## Использование [make](https://www.gnu.org/software/make/manual/make.html) для сборки проектов
### Написание Makefile

`Makefile` - это текстовый файл, содержащий инструкции для утилиты `make`. Этот файл определяет зависимости между файлами и указывает, какие команды нужно выполнить для сборки проекта.

Пример простого `Makefile`:

```makefile
CC = g++
CFLAGS = -Wall

all: hello

hello: main.o
	$(CC) $(CFLAGS) main.o -o hello
main.o: main.cpp
	$(CC) $(CFLAGS) -c main.cpp
clean:
	rm -rf *.o hello
```

В данном примере:

-   `CC` - это переменная, содержащая имя компилятора (в данном случае `g++`).
-   `CFLAGS` - флаги компиляции (в данном случае `-Wall` для вывода предупреждений).
-   `all` - цель по умолчанию, которая собирает исполняемый файл `hello`.
-   `hello` - цель для сборки исполняемого файла.
-   `main.o` - цель для сборки объектного файла `main.o`.
-   `clean` - цель для удаления временных файлов.

### `MakeFile` для сборки нескольких файлов

```makefile
CC = clang
CFLAGS = -Wall -Wextra -O3
SRC = $(wildcard src/*.c)
HEADERS = $(wildcard include/*.h)
TARGET = lab2

$(TARGET): $(SRC) $(HEADERS)
	$(CC) $(SRC) -o $(TARGET) $(CFLAGS) -lm

clean:
	rm -f $(TARGET)

```
В данном примере:

- `CC = clang`: Эта строка определяет переменную CC, которая будет использоваться как компилятор. В данном случае, используется компилятор Clang.

- `CFLAGS = -Wall -Wextra -O3`: Эта строка определяет переменную CFLAGS, которая содержит флаги компиляции. Флаги включают в себя -Wall и -Wextra для вывода предупреждений и -O3 для уровня оптимизации.

- `SRC = $(wildcard src/*.c)`: Эта строка использует встроенную функцию wildcard, чтобы автоматически найти все файлы с расширением .c в директории src. Результат сохраняется в переменной SRC.

- `HEADERS = $(wildcard include/*.h)`: Аналогично строке выше, эта строка находит все файлы с расширением .h в директории include и сохраняет результат в переменной HEADERS.

- `TARGET = lab2`: Здесь определяется переменная TARGET, которая указывает на имя целевого исполняемого файла (программы). В данном случае, имя файла - `lab2`.

- `$(TARGET): $(SRC) $(HEADERS)`: Эта строка говорит о том, что цель lab2 зависит от файлов, перечисленных в переменных SRC и HEADERS.

- `$(CC) $(SRC) -o $(TARGET) $(CFLAGS) -lm`: Эта строка является правилом для построения цели lab2. Она использует компилятор $(CC) для компиляции файлов и создания исполняемого файла с именем, указанным в $(TARGET). Флаги компиляции указываются с использованием $(CFLAGS), а также добавляется флаг -lm, который связывает программу с библиотекой математических функций.

- `clean: rm -f $(TARGET)`: Эта строка определяет цель clean, которая удаляет файл, указанный в переменной TARGET, тем самым очищая рабочую директорию от созданных в процессе компиляции файлов.

### Запуск утилиты `make`

1.  Создайте файл с именем `Makefile` в корневой директории проекта.
2.  Добавьте необходимые инструкции в `Makefile`.
3.  Откройте терминал и перейдите в директорию проекта.
4.  Выполните команду `make` для сборки проекта.

### Запуск исполняемого файла

После успешной сборки проекта выполните следующую команду в терминале для запуска исполняемого файла:
`./hello`


## Линковка библиотек в Make
Пример с SFML
Для линковки SFML в Makefile нужно указать пути к заголовочным файлам и библиотекам, а также флаги компиляции.

Пример Makefile для проекта с SFML:

```makefile
CXX = g++
CXXFLAGS = -Wall -Wextra -std=c++17
LDFLAGS = -lsfml-system -lsfml-window -lsfml-graphics -lsfml-audio -lsfml-network

SRC = main.cpp
TARGET = MySFMLProject

$(TARGET): $(SRC)
	$(CXX) $(CXXFLAGS) -o $(TARGET) $(SRC) $(LDFLAGS)

clean:
	rm -f $(TARGET)
```

Объяснение:
```LDFLAGS = -lsfml-system -lsfml-window -lsfml-graphics -lsfml-audio -lsfml-network ```— флаги для линковки SFML.

```$(CXX) $(CXXFLAGS) -o $(TARGET) $(SRC) $(LDFLAGS)``` — команда для компиляции и линковки.


Пример с SDL2
Пример Makefile для проекта с SDL2:

```makefile
CXX = g++
CXXFLAGS = -Wall -Wextra -std=c++17
LDFLAGS = -lSDL2

SRC = main.cpp
TARGET = MySDLProject

$(TARGET): $(SRC)
	$(CXX) $(CXXFLAGS) -o $(TARGET) $(SRC) $(LDFLAGS)

clean:
	rm -f $(TARGET)
```

Объяснение:
LDFLAGS = -lSDL2 — флаг для линковки SDL2.

## Сборка проектов использующих [cmake](https://cmake.org/) 
Часто вы можете увидеть проекты в корне которых есть файл CMakeLists.txt это файл описания генерации файлов сборки при помощи cmake

Чаще всего используется для генерации файлов сборки для [make](https://www.gnu.org/software/make/manual/make.html) или [ninja](https://ninja-build.org/)

Использование cmake для проектов содержащих 4+ файлов вместо make имхо уже необходимо, для большего удобства

Для того чтобы собрать такие проекты нужно перейти в директорую с проектом `cd ProjectDir`

Там создать и перейти в папку build `mkdir build && cd build`

Выполнить ```cmake ..```

После будет выполнена генерация файлов систем сборки таких как `make`

Чтобы сгенерировать файлы сборки для определенной системы сборки например для `ninja` нужно выполнить `cmake ..-G=Ninja`

Чтобы сгенерировать файлы сборки для утилиты `make` нужно выполнить `cmake .. -G="Unix Makefiles"`

Чтобы начать сборку проекта нужно выполнить `ninja -j$(nproc)` для ninja или `make -j$(nproc)` для make

Чтобы запустить исполняемый файл нужно выполнить `./<название_исполняемого_файла>`

Название исполняемого файла можно узнать посмотрев в файл CMakeLists.txt на строчку ```project()```



## Простейший CMakeLists.txt для сборки проекта на C++
```cmake
cmake_minimum_required(VERSION 3.20)
project(<Название проекта>)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_executable(<Название проекта> main.cpp)
```


## Использование cmake в проекте с несколькими файлами
```cmake
cmake_minimum_required(VERSION 3.20)

project(project)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

file(GLOB SOURCE_FILE src/*.cpp) # на самом деле так лучше не делать но для пока и так норм
#лучше делать вот так:
add_executable(${CMAKE_PROJECT_NAME} main.cpp) # и перечеслять остльаные файлы тут

target_include_directories(project PRIVATE include)
target_compile_options(project PRIVATE -Wall -Wextra)

```

- `cmake_minimum_required(VERSION 3.20)`: Указывает минимальную версию CMake, необходимую для сборки проекта. Если установленная версия CMake ниже указанной, произойдет ошибка.

- `project(project)`: Задает имя проекта. В данном случае, проекту присвоено имя "project". Это также устанавливает переменные, такие как ${PROJECT_NAME}, которые могут использоваться в дальнейших настройках проекта.

- `set(CMAKE_CXX_STANDARD 23)`: Устанавливает стандарт языка C++ для компиляции проекта. В данном случае, используется стандарт C++23.

- `set(CMAKE_CXX_STANDARD_REQUIRED True)`: Указывает, что использование указанного стандарта C++ является обязательным для компилятора.

- `set(CMAKE_EXPORT_COMPILE_COMMANDS ON)`: Включает экспорт файла compile_commands.json, который может использоваться для интеграции с инструментами, такими как Clangd.

- `file(GLOB SOURCE_FILE src/*.cpp)`: Создает список исходных файлов, автоматически добавляя все файлы с расширением ".cpp" из директории "src" в переменную SOURCE_FILE.

- `add_executable(${CMAKE_PROJECT_NAME} <sources>.cpp main.cpp)`: Явное перечисление списка исходных файлов, ручное но это лучше чтобы не перегенерировать cmake при добавлении файла.
  
- `add_executable(project ${SOURCE_FILE})`: Создает исполняемый файл с именем "project" из перечисленных в SOURCE_FILE исходных файлов.

- `target_include_directories(project PRIVATE include)`: Добавляет директорию "include" к путям поиска заголовочных файлов для проекта. Все файлы в этой директории будут доступны только для компиляции этого проекта (PRIVATE).

- `target_compile_options(project PRIVATE -Wall -Wextra)`: Устанавливает опции компиляции для проекта. В данном случае, включаются предупреждения компилятора с уровнями -Wall и -Wextra, которые помогают выявлять проблемы в коде.

## Как использовать [clang-format](https://clang.llvm.org/docs/ClangFormat.html) - форматтер кода
```sh
clang-format -style=google -dump-config > .clang-format
clang-format -i *.cpp #если форматируете руками, не в редакторе кода
```

Обычно форматировать код в ручную не нужно этим занимается [vscode](https://marketplace.visualstudio.com/items?itemName=xaver.clang-format) и все популярные ide по сохранению файла

## CMake и статические библиотеки

Бывает так что вам нужно какую то часть вашего кода использовать как статическую библиотеку.
Чтобы это было удобно сделать можно создать примерно такую структуру файлов:
```
|---src
|    \
|     CMakeLists.txt
|      main.cpp
|---mystaticlib
|	\
|	 CMakeLists.txt
|	  MyStaticLibrary.cpp
|	  MyStaticLibrary.hpp
|
|---CMakeLists.txt
```

`src` содержит `main.cpp` в код которого будет включаться статически линкующуюся библиотека из поддиректории mystaticlib

`CMakeLists.txt` в находящийся в корне будет выглядеть примерно так:

```cmake
cmake_minimum_required (VERSION 3.20)
project (Example)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON) # это нужно чтобы ланг сервер в vscode мог лучше понять написаный вами код

add_subdirectory (mystaticlib)
add_subdirectory (src)
```
`CMakeLists.txt` в находящийся в папке `mystatilib` будет выглядеть примерно так:
```cmake
cmake_minimum_required (VERSION 3.20)
 
project(mystatilib)
 
set(SOURCE_FILES "MyStaticLibrary.cpp")
set(HEADER_FILES "MyStaticLibrary.h")
 
# Мы объявляем проект как статическую библиотеку и добавляем в нее все файлы исходного кода.
add_library(MyStaticLibrary STATIC ${HEADER_FILES} ${SOURCE_FILES})
```
`CMakeLists.txt` в находящийся в папке `src` будет выглядеть примерно так:
```cmake
cmake_minimum_required (VERSION 3.20)
 
project(exampleStatic)
 
set(SOURCE_FILES "main.cpp")
 
add_executable (exampleStatic ${SOURCE_FILES})
 
# Подключаем библиотеку, указываем откуда брать заголовочные файлы
include_directories("../MyStaticLibrary")
# А также указываем зависимость от статической библиотеки
target_link_libraries(exampleStatic mystatilib)
```
[Шаблон проекта использующего cmake](https://github.com/cppshizoidS/cmake_boilerplate)


Пример CMakeLists.txt для проекта с SFML:

```cmake
cmake_minimum_required(VERSION 3.20)
project(MySFMLProject)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Поиск SFML
find_package(SFML REQUIRED COMPONENTS system window graphics audio network)

# Добавление исполняемого файла
add_executable(MySFMLProject main.cpp)

# Линковка SFML
target_link_libraries(MySFMLProject PRIVATE sfml-system sfml-window sfml-graphics sfml-audio sfml-network)
Объяснение:
find_package(SFML REQUIRED COMPONENTS system window graphics audio network) # ищет SFML и его компоненты (system, window, graphics, audio, network). Если библиотека не найдена, CMake #выдаст ошибку.

```

Пример CMakeLists.txt для проекта с SDL2:

```cmake
cmake_minimum_required(VERSION 3.20)
project(MySDLProject)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Поиск SDL2
find_package(SDL2 REQUIRED)

# Добавление исполняемого файла
add_executable(MySDLProject main.cpp)

# Линковка SDL2
target_link_libraries(MySDLProject PRIVATE SDL2::SDL2)
```

Объяснение:
find_package(SDL2 REQUIRED) — ищет SDL2. Если библиотека не найдена, CMake выдаст ошибку.

target_link_libraries(MySDLProject PRIVATE SDL2::SDL2) — линкует SDL2 с вашим проектом.
