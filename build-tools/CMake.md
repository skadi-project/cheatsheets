# CMake — шпаргалка

## Что это и зачем

CMake описывает проект и генерирует файлы сборки для Make, Ninja, Visual Studio и других генераторов. Он отделяет описание проекта от конкретной среды сборки. Для маленького проекта может хватить Makefile.

## Установка

В Debian/Ubuntu установите `cmake` и `build-essential`, в Arch — `cmake`, `make` и `gcc`. В macOS используйте Homebrew (`brew install cmake`) и Xcode Command Line Tools. В Windows доступны официальный установщик CMake и Visual Studio Build Tools с поддержкой C++. Проверьте версию через `cmake --version`.

## Быстрый старт

Структура:

```text
project/
├── CMakeLists.txt
├── main.cpp
└── math.cpp
```

Возьмите `main.cpp` и `math.cpp` из [примера GNU Make](Make.md). Сохраните следующий файл как `CMakeLists.txt`.

```cmake
cmake_minimum_required(VERSION 3.20)
project(Calculator LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_library(math_lib math.cpp)

add_executable(calculator main.cpp)
target_link_libraries(calculator PRIVATE math_lib)
```

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
./build/calculator
```

На Windows с Visual Studio исполняемый файл может находиться в `build/Debug/`. Путь зависит от генератора.

## Основные команды CMake

| Команда | Назначение | Пример |
|---|---|---|
| `cmake -S -B` | Настроить отдельный каталог сборки | `cmake -S . -B build` |
| `cmake --build` | Запустить сборку | `cmake --build build` |
| `--config` | Выбрать конфигурацию multi-config | `cmake --build build --config Release` |
| `--target` | Собрать конкретную цель | `cmake --build build --target calculator` |
| `--parallel` | Включить параллельную сборку | `cmake --build build --parallel 4` |
| `--install` | Установить результат | `cmake --install build --prefix install` |
| `-D` | Передать переменную | `-DCMAKE_BUILD_TYPE=Release` |
| `-G` | Выбрать генератор | `cmake -S . -B build -G Ninja` |
| `--fresh` | Очистить кэш и настроить сборку заново (CMake 3.24+) | `cmake --fresh -S . -B build` |

## Команды CMakeLists.txt

| Команда | Назначение | Пример |
|---|---|---|
| `cmake_minimum_required` | Минимальная версия | `cmake_minimum_required(VERSION 3.20)` |
| `project` | Имя и языки проекта | `project(App LANGUAGES CXX)` |
| `add_executable` | Исполняемая цель | `add_executable(app src/main.cpp)` |
| `add_library` | Библиотека | `add_library(core src/core.cpp)` |
| `target_link_libraries` | Зависимости цели | `target_link_libraries(app PRIVATE core)` |
| `target_include_directories` | Пути заголовков | `target_include_directories(core PUBLIC include)` |
| `target_compile_features` | Требуемые возможности языка | `target_compile_features(core PUBLIC cxx_std_17)` |
| `target_compile_options` | Опции компилятора | `target_compile_options(core PRIVATE -Wall)` |
| `add_subdirectory` | Подключает подкаталог | `add_subdirectory(src)` |
| `install` | Описывает установку | `install(TARGETS app DESTINATION bin)` |
| `enable_testing` | Включает CTest | `enable_testing()` |
| `add_test` | Добавляет тест | `add_test(NAME smoke COMMAND app)` |

## Debug, Release и зависимости

```bash
cmake -S . -B build-debug -DCMAKE_BUILD_TYPE=Debug
cmake --build build-debug
cmake -S . -B build-release -DCMAKE_BUILD_TYPE=Release
cmake --build build-release
```

Для зависимости, которую ищет система, используйте `find_package`. Не добавляйте абсолютные пути конкретного компьютера в репозиторий.

```cmake
find_package(Threads REQUIRED)
target_link_libraries(calculator PRIVATE Threads::Threads)
```

## Тесты

Добавьте блок в конец `CMakeLists.txt`. Программа возвращает `0`, если сумма равна `5`. Перед запуском CTest повторите настройку и сборку. Для Visual Studio добавьте `-C Debug` к вызову CTest.

```cmake
enable_testing()
add_test(NAME calculator_smoke COMMAND calculator)
```

```bash
ctest --test-dir build --output-on-failure
```

`ctest` запускайте после сборки. Если программа зависит от рабочей директории или переменных окружения, задайте их в тестовой конфигурации.

## Ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `CMake was unable to find a build program` | Нет Make, Ninja или генератора IDE | Установите генератор или явно задайте `-G` |
| `No CMAKE_CXX_COMPILER could be found` | Нет компилятора или он не в `PATH` | Проверьте `g++ --version`, Clang или Visual Studio |
| `target_link_libraries` не находит цель | Имя цели отличается или библиотека отсутствует | Объявите цель и проверьте её имя |
| Заголовок не найден | Не задан include-директория | Используйте `target_include_directories` |
| Старые настройки после изменения | В каталоге остался старый кэш | Используйте новый каталог или `cmake --fresh` |

## Документация

- [Официальная документация CMake](https://cmake.org/documentation/)
- [CMake tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)
- [GNU Make](Make.md)
- [Основы C++](../programming/Cpp.md)
