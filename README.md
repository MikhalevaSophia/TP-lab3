## Homework

Представьте, что вы стажер в компании "Formatter Inc.".
### Задание 1
Вам поручили перейти на систему автоматизированной сборки **CMake**.
Исходные файлы находятся в директории [formatter_lib](formatter_lib).
В этой директории находятся файлы для статической библиотеки *formatter*.
Создайте `CMakeList.txt` в директории [formatter_lib](formatter_lib),
с помощью которого можно будет собирать статическую библиотеку *formatter*.

> Создаём **в директории** `formatter_lib` файл с названием `CMakeLists.txt`
>
> Приведу готовый файл:
> ```cmake
> cmake_minimum_required(VERSION 3.4)
> project(formatter_lib)
> set(CMAKE_CXX_STANDARD 11)
> set(CMAKE_CXX_STANDARD_REQUIRED ON)
> # Выше - стандартная фигня, она везде будет одинаковой, кроме имени проекта.
> add_library(formatter_lib STATIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter.cpp)
> # Добавляем таргет-статичную библиотеку formatter_lib из файла с кодом formatter.cpp
> ```
> Проверим, что проект собирается. Для этого **в директории** `formatter_lib` введём команды:
> ```sh
> $ cmake -B build
> $ cmake --build build
> ```

### Задание 2
У компании "Formatter Inc." есть перспективная библиотека,
которая является расширением предыдущей библиотеки. Т.к. вы уже овладели
навыком созданием `CMakeList.txt` для статической библиотеки *formatter*, ваш 
руководитель поручает заняться созданием `CMakeList.txt` для библиотеки 
*formatter_ex*, которая в свою очередь использует библиотеку *formatter*.

> Аналогично, создаём `CMakeLists.txt` в папке `formatter_ex_lib`:
> ```cmake
> cmake_minimum_required(VERSION 3.4)
> project(formatter_ex_lib)
> set(CMAKE_CXX_STANDARD 11)
> set(CMAKE_CXX_STANDARD_REQUIRED ON)
> 
> add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib formatter_lib_dir)
> # Собираем то, что лежит в formatter_lib и пихаем это в папочку formatter_lib_dir
> add_library(formatter_ex_lib STATIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter_ex.cpp)
> # Добавляем статичную библиотеку formatter_ex_lib из файла formatter_ex.cpp
> target_include_directories(formatter_ex_lib PUBLIC
> ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib
> )
> # Открываем таргету formatter_ex_lib доступ к директории formatter_lib
> target_link_libraries(formatter_ex_lib formatter_lib)
> # Привязываем библиотеку formatter_lib к библиотеке formatter_ex_lib.
> ```
> После чего собираем командами:
> ```
> $ cmake -B build
> $ cmake --build build
> ```

### Задание 3
Конечно же ваша компания предоставляет примеры использования своих библиотек.
Чтобы продемонстрировать как работать с библиотекой *formatter_ex*,
вам необходимо создать два `CMakeList.txt` для двух простых приложений:
* *hello_world*, которое использует библиотеку *formatter_ex*;

> Процесс такой же. Директория `hello_world_application`, файл `CMakeLists.txt`:
> ```cmake
> cmake_minimum_required(VERSION 3.4)
> 
> project(hello_world)
> set(CMAKE_CXX_STANDARD 11)
> set(CMAKE_CXX_STANDARD_REQUIRED ON)
> 
> include_directories("../formatter_lib")
> include_directories("../formatter_ex_lib")
> 
> add_library(formatter_lib STATIC "../formatter_lib/formatter.cpp")
> add_library(formatter_ex_lib STATIC "../formatter_ex_lib/formatter_ex.cpp")
> 
> 
> 
> add_executable(lab03 "hello_world.cpp")
> target_link_libraries(lab03 formatter_ex_lib formatter_lib)

> # Привязываем к таргету hello_world библиотеку formatter_ex_lib
> ```
> Сборка:
> ```sh
> $ cmake -B build
> $ cmake --build build
> ```
> Теперь можно проверить, что всё работает как надо.
> 
> В папке `build` появится исполняемый файл `hello_world`. Запустим его:
> ```sh
> $ build/hello_world
> ```


* *solver*, приложение которое испольует статические библиотеки *formatter_ex* и *solver_lib*.

> Сначала соберём библиотеку `solver_lib`. 
> 
> В коде библиотеки есть **ошибка**: не хватает библиотеки `cmath`, а `sqrtf` не лежит в `std`. Исправим это. В файле `solver_lib/solver.cpp`:
> ```cpp
> #include "solver.h"
> 
> #include <stdexcept>
> #include <cmath>
> 
> void solve(float a, float b, float c, float& x1, float& x2)
> {
>     float d = (b * b) - (4 * a * c);
> 
>     if (d < 0)
>     {
>         throw std::logic_error{"error: discriminant < 0"};
>     }
> 
>     x1 = (-b - sqrtf(d)) / (2 * a);
>     x2 = (-b + sqrtf(d)) / (2 * a);
> }
> 
> ```
> Далее соберём библиотеку.
> 
> Директория `solver_lib`, файл `CMakeLists.txt`:
> ```cmake
> cmake_minimum_required(VERSION 3.4)
> project(solver_lib)
> set(CMAKE_CXX_STANDARD 11)
> set(CMAKE_CXX_STANDARD_REQUIRED ON)
> add_library(solver_lib ${CMAKE_CURRENT_SOURCE_DIR}/solver.cpp)
> ```
> Директория `solver_application`, файл `CMakeLists.txt`:
> ```cmake
> cmake_minimum_required(VERSION 3.4)
> project(solver)
> set(CMAKE_CXX_STANDARD 11)
> set(CMAKE_CXX_STANDARD_REQUIRED ON)
> add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../formatter_ex_lib formatter_ex_lib_dir)
> add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../solver_lib solver_lib_dir)
> add_executable(solver ${CMAKE_CURRENT_SOURCE_DIR}/equation.cpp)
> target_include_directories(formatter_ex_lib PUBLIC
> ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_ex_lib
> ${CMAKE_CURRENT_SOURCE_DIR}/../solver_lib
> )
> target_link_libraries(solver formatter_ex_lib solver_lib)
> ```
> Разница лишь в том, что к таргету `solver` подключаются две библиотеки.
> 
> Сборка:
> ```sh
> $ cmake -B build
> $ cmake --build build
> ```
> Проверим, что всё работает:
> ```sh
> $ build/solver
> ```
> От нас потребуется ввести три числа — коэффициенты квадратного уравнения. На выходе получим решение уравнения.
