# Тестирование

## Введение в тестирование
Модульное тестирование в C++ с использованием [Google Test](https://google.github.io/googletest) и CMake:

  
Модульное тестирование в C++ является важной практикой разработки программного обеспечения, позволяющей проверить корректность работы отдельных функций или модулей кода. 
В контексте модульного тестирования, минимальной тестируемой единицей является функция.
Писать тесты удобно в отдельных файлах, где для каждой тестируемой функции создается соответствующий файл с тестами.


## Шаблон проекта с использованием CMake:

Создайте структуру каталогов для вашего проекта. Предположим, у вас есть следующая структура каталогов:

```sh
├── CMakeLists.txt
├── src
│ ├── my_code.cpp
│ └── CMakeLists.txt
├── test
│ ├── test_my_code.cpp
│ └── CMakeLists.txt

```

CMakeLists.txt в корневом каталоге:
```cmake
cmake_minimum_required(VERSION 3.20)
project(MyProject)

add_subdirectory(src)
add_subdirectory(test)
```

CMakeLists.txt в каталоге src:
```cmake
add_library(my_code my_code.cpp)
```

CMakeLists.txt в каталоге test:
```cmake
find_package(GTest REQUIRED)
include_directories(${GTEST_INCLUDE_DIRS})

add_executable(test_my_code test_my_code.cpp)
target_link_libraries(test_my_code my_code ${GTEST_LIBRARIES} pthread)
```

Пример `my_code.cpp`:

```cpp
// src/my_code.cpp
int  add(int  a,  int  b)  {
return a + b;
}
```

Пример `test_my_code.cpp`:
```cpp
// test/test_my_code.cpp
#include  <gtest/gtest.h>
#include  "my_code.cpp"

TEST(AddTest, PositiveNumbers)  {
EXPECT_EQ(add(2,  3),  5);
}

TEST(AddTest, NegativeNumbers)  {
EXPECT_EQ(add(-2,  -3),  -5);
}

int  main(int  argc,  char  **argv)  {
::testing::InitGoogleTest(&argc, argv);
return  RUN_ALL_TESTS();
}
```


## Основные макросы [GTest](https://google.github.io/googletest/primer.html):

Google Test (GTest) предоставляет различные макросы для написания тестов в C++. Вот основные макросы, которые часто используются при написании тестов с использованием GTest:

```cpp
TEST(test_case_name, test_name)
```

* Определяет новый тест.
test_case_name: Имя тестового случая (группы тестов).
test_name: Имя теста внутри тестового случая.
```cpp

TEST(MathTest, Addition)  {
// Тестирование сложения
// ASSERT_ или EXPECT_ макросы используются для проверки условий
}
```
* ASSERT_ и EXPECT_ макросы:
Используются для проверки условий в тестах. Если условие не выполняется, ASSERT_ вызывает завершение теста, а EXPECT_ продолжает выполнение теста.

```cpp

ASSERT_EQ(expected, actual); // Проверка на равенство

ASSERT_NE(expected, actual); // Проверка на неравенство

ASSERT_TRUE(condition); // Проверка на true

ASSERT_FALSE(condition); // Проверка на false

ASSERT_EQ(expected, actual) и EXPECT_EQ(expected, actual)

```

* Проверка на равенство
```cpp
ASSERT_EQ(val1, val2) и EXPECT_EQ(val1, val2)

```

* Проверка на неравенство.

```cpp
ASSERT_NE(val1, val2) и EXPECT_NE(val1, val2)
```


* ASSERT_TRUE(condition):
Если условие `condition` оценивается как ложное (false), то тест завершается сразу, и остальные проверки внутри теста не выполняются.
Если условие истинное (true), то тест продолжается.

```cpp
ASSERT_TRUE(condition) и EXPECT_TRUE(condition)
```

* Проверка на ложность.

```cpp
ASSERT_FALSE(condition) и EXPECT_FALSE(condition)
```

* Проверка на ложность.

```cpp
ASSERT_THROW(statement, exception_type) и EXPECT_THROW(statement, exception_type)
```

* Проверка, что выполнение statement генерирует исключение определенного типа.

```cpp
ASSERT_THROW(statement, exception_type) и EXPECT_THROW(statement, exception_type)
```

* Проверка, что выполнение statement не генерирует исключение.

```cpp
ASSERT_THROW(throwAnException(), MyException);

ASSERT_NO_THROW(statement) и EXPECT_NO_THROW(statement)
ASSERT_NO_THROW(doSomething());
```

* Проверка, что две строки равны.

ASSERT_STREQ и EXPECT_STREQ - это макросы в библиотеке тестирования GTest (Google Test), предназначенные для сравнения двух строк.

```cpp
ASSERT_STREQ(expected_str, actual_str) 
EXPECT_STREQ(expected_str, actual_str)
```

* Соответсвие строк

Этот код используется для утверждения, что строка, представленная как myString.c_str(), точно соответствует строке "hello".
Этот код используется для утверждения, что строка, представленная как myString.c_str(), точно соответствует строке "hello".

```cpp
ASSERT_STREQ("hello", myString.c_str());

ASSERT_LT(val1, val2) и EXPECT_LT(val1, val2)
```

* Проверка, что val1 меньше val2.

```cpp
ASSERT_LT(2,  5);

ASSERT_GT(val1, val2) и EXPECT_GT(val1, val2)
```

* Проверка, что val1 больше val2.

```cpp
ASSERT_GT(5,  2);
```

Эти макросы помогают создавать тесты с использованием GTest.


## Использование [CTest](https://cmake.org/cmake/help/latest/manual/ctest.1.html) для автоматизации тестирования

CTest — это инструмент, входящий в комплект поставки CMake, который предоставляет средства для автоматизации и управления тестированием ваших проектов. Добавление поддержки CTest к вашему проекту позволяет удобно интегрировать тестирование в процесс сборки, а также легко использовать его в системах непрерывной интеграции.
* Автоматическое обнаружение тестов: CTest способен автоматически обнаруживать и запускать тесты в вашем проекте. Это упрощает процесс добавления новых тестов и поддержки существующих.
* Параметризация тестов: CTest позволяет параметризовать тесты, что облегчает создание вариантов тестов для различных сценариев и входных данных.
* Интеграция с CD/CI: Автоматическое выполнение тестов при каждом изменении кода в системе непрерывной интеграции обеспечивает постоянную проверку работоспособности вашего проекта.

.gitlab-ci.yml:
```yaml
stages:
-  test

test:

script:
-  mkdir build
-  cd build
-  cmake ..
-  make
-  ctest  # Запуск тестов с использованием CTest
```
Этот блок в файле .gitlab-ci.yml интегрирует CTest в процесс непрерывной интеграции. После построения проекта, команда ctest запускает тесты, а результаты могут быть легко анализированы в системе CI/CD.
