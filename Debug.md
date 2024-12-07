# Дебаггер
Бывает так что программа ведет себя не так как вы предполагали. И тогда нужно отладить конечную программу чтобы найти ошибки .
Для дебага(отладки) C и C++ существуют специальные программы -  дебаггеры.
На linux существуют 2 чаще используемых дебаггера это [GDB](https://sourceware.org/gdb/current/onlinedocs/)/[LLDB](https://lldb.llvm.org/use/tutorial.html) и мемори дебаггер [valgrind](https://valgrind.org/docs/manual/manual.html)

## Как пользоваться дебаггерами([lldb](https://lldb.llvm.org/use/tutorial.html) краткий экскурс)
1 **Для начала нужно скомпилировать проект с флагом `-g` чтобы компилятор сохранил информацию о компиляции которая будет полезна для дебаггера**

``` clang++ -g main.cpp -o main ```

2 **Далее нужно запустить программу используя `lldb`**

``` lldb ./main ```

Так же можно добавить условие в CMakeLists.txt чтобы если сборка будет производится в дебаг режиме флаг подставлялся автоматически
```cmake
if (DEBUG)
    add_compile_options(-g3 -ggdb)
    add_compile_definitions(DEBUG)
endif()
```

3 **Основные комманды LLDB:**
* Запустить программу
```run```

* Остановка выполнения
```process interrupt```

* Получить информацию о стеке вызовов
```thread backtrace```

* Установить брейкпоинт(точку останова):
```breakpoint set --name functionName```

* Установить условный брейкопинт:
```breakpoint set --name functionName --condition 'expression'```

* Использование вочпоинтов
Watchpoints позволяют останавливаться при изменении значения переменной:
```watchpoint set variable varName ```

* Продолжение выполнения
```process continue```

* Вывод значений переменной
```expr varName```

* Вывод информации о переменных и типах
```image lookup -a address```

* Сохранение дебаг сессии
```save``` и ее загрузка ```target create```

Так же в редакторах кода и ide(VSCode и Clion например) есть gui для создания брейк поинтов, отслеживания значения переменной в данный момент и подстановки выражений

## Как пользоваться дебаггерами([gdb](https://sourceware.org/gdb/current/onlinedocs/) краткий экскурс)
1  **Компиляция с флагом `-g`:**
    `g++ -g main.cpp -o main` 
   
    Или для использования GCC:

    `gcc -g main.c -o main` 
    
2  **Запуск GDB:**
    `gdb ./main` 
    
3  **Условия в CMakeLists.txt:**
    `if (DEBUG)
        add_compile_options(-g3 -ggdb)
        add_compile_definitions(DEBUG)
    endif()` 
    
4  **Основные команды GDB:**
* Запуск программы:
        ```run``` 
        
*   Прерывание выполнения:
        ```Ctrl+C``` 
        
*   Информация о стеке вызовов:
        ```bt``` 
        
*   Установка брейкпоинта:
        ```break functionName``` 
        
*   Установка условного брейкпоинта:
        ```break functionName if expression``` 
        
*   Использование watchpoints:
        ```watch varName``` 
        
*   Продолжение выполнения:
        ```continue```
        
*   Вывод значения переменной:
        ```print varName``` 
        
* Вывод информации о переменных и типах:
        ``` info locals```
        или
        ```ptype typeName ```

*   Сохранение и загрузка сессии:
        ```save breakpoints filename```
        и
        ```source filename```

## Как пользоваться дебаггерами([valgrind](https://valgrind.org/docs/manual/manual.html) краткий экскурс)
Так же может быть такое что ваша программа может иметь ошибки с памятью(течь по памяти например)
Для начала нужно скопилировать проект с флагом -g чтобы компилятор сохранил информацию о компиляции которая будет полезна для дебаггера

``` clang++ -g main.cpp -o main ```

Далее нужно запустить программу используя valrind

``` valgrind --leak-check=full -s ./main ```

Флаг --leak-check=full включает проверку утечек памяти.

* Valgrind выдаст общий отчет об утечках памяти после завершения выполнения программы.
* Для получения более подробной информации о местах утечек, смотрите выходные данные Valgrind.
* Valgrind также может обнаруживать ошибки чтения и записи в неинициализированные области памяти.

Как будет выглядеть утечка памяти в valgrind

Например у нас есть код в котором мы забыли освободить память:
```cpp
#include <cstdlib>
int main() {
    int *myArray = new int[5]; // выделение памяти для массива
    // забыли удалить память
    return 0;
}
```

```sh
==2581524== Memcheck, a memory error detector
==2581524== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==2581524== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==2581524== Command: ./main --leak-check=full -s
==2581524== 
==2581524== 
==2581524== HEAP SUMMARY:
==2581524==     in use at exit: 20 bytes in 1 blocks
==2581524==   total heap usage: 2 allocs, 1 frees, 73,748 bytes allocated
==2581524== 
==2581524== LEAK SUMMARY:
==2581524==    definitely lost: 20 bytes in 1 blocks <-  ͟п͟о͟т͟е͟р͟я͟н͟о͟ ͟2͟0͟ ͟б͟а͟й͟т͟
==2581524==    indirectly lost: 0 bytes in 0 blocks
==2581524==      possibly lost: 0 bytes in 0 blocks
==2581524==    still reachable: 0 bytes in 0 blocks
==2581524==         suppressed: 0 bytes in 0 blocks
```
Как видно из вывода valgrind у нас есть 2 аллокации и всего 1 free, так же valgrind указал что мы потеряли 20 байт 

что означает вывод valgrind:
```definitely lost: количество байт, которые были выделены и потеряны, и на которые нет указателей.```
```indirectly lost: количество байт, которые были выделены, потеряны и не могут быть восстановлены.```
```possibly lost: количество байт, которые, возможно, утрачены, но есть указатели, ведущие к этой утраченной памяти.```
```still reachable: количество байт, которые еще доступны через указатели, но не были освобождены.```
```suppressed: количество байт, которые были подавлены и не учитываются в отчете.```

Как будет выглядеть процесс дебага кода в lldb:
```cpp
#include <iostream>
int main() {
    int x = 10;
    int y = 0;
    int result = x / y;  // ошибка: деление на ноль

    std::cout << "Result: " << result << std::endl;

    return 0;
}
```
* Компиляция с флагом -g для сохранения отладочной информации ``` clang++ -g main.cpp -o main ```
* Запуск программы в lldb ```lldb ./main```
* Установка брейк поинт на строке 7 ```breakpoint set --file main.cpp --line 7```
* Запуск программы ```run``` ![](https://i.imgur.com/IxZokWx.png)
* Произойдет остановка на точке останова. Теперь вы можете выполнять пошаговую инструкцию. ```thread step-inst```
* Продолжайте выполнение программы пошагово и наблюдайте за ее состоянием ``` continue ```
