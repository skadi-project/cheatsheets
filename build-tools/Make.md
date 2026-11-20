# GNU Make — шпаргалка

## Что это и зачем

Make строит цели по зависимостям и пересобирает только устаревшие файлы. Он удобен для простых и средних проектов C/C++. Для сложных кроссплатформенных проектов используйте CMake, который генерирует файлы сборки.

## Установка

В Debian/Ubuntu установите `build-essential`, в Arch — `base-devel` и `gcc`. В macOS GNU Make доступен через Homebrew (`brew install make`, команда `gmake`). В Windows используйте WSL или MSYS2 UCRT64 с GNU Make и GCC. Проверьте версию через `make --version` или `gmake --version`.

## Быстрый старт

Структура:

```text
project/
├── Makefile
├── main.cpp
└── math.cpp
```

`main.cpp`:

```cpp
#include <iostream>

int add(int a, int b);

int main() {
    const int result = add(2, 3);
    std::cout << result << '\n';
    return result == 5 ? 0 : 1;
}
```

`math.cpp`:

```cpp
int add(int a, int b) {
    return a + b;
}
```

`Makefile` должен содержать TAB перед командами рецепта.

```make
CXX := g++
CXXFLAGS := -std=c++17 -Wall -Wextra -Wpedantic
TARGET := app
OBJECTS := main.o math.o

.PHONY: all clean

all: $(TARGET)

$(TARGET): $(OBJECTS)
	$(CXX) $(CXXFLAGS) $^ -o $@

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

clean:
	rm -f $(TARGET) $(OBJECTS)
```

```bash
make
./app
touch math.cpp
make -n
make
make clean
```

## Синтаксис

| Элемент | Назначение | Пример |
|---|---|---|
| Цель | Имя результата | `app: main.o` |
| Зависимость | Файл или цель, нужная для цели | `app: main.o math.o` |
| Рецепт | Команды сборки | TAB + `g++ main.o -o app` |
| Переменная | Переиспользуемое значение | `CXX := g++` |
| `$@` | Имя текущей цели | `-o $@` |
| `$<` | Первая зависимость | `-c $<` |
| `$^` | Все зависимости без повторов | `g++ $^ -o $@` |
| `%` | Шаблон правила | `%.o: %.cpp` |
| `.PHONY` | Цель не является именем файла | `.PHONY: clean` |
| `?=` | Значение по умолчанию | `CXXFLAGS ?= -Wall` |
| `include` | Подключает другой make-файл | `include config.mk` |

## Важные команды

```bash
make                 # первая цель
make clean           # очистить результаты
make -n               # показать команды без выполнения
make -B               # пересобрать всё
make -j2              # до двух параллельных заданий
make -f Makefile.dev  # выбрать файл
make -C subproject    # выполнять в другом каталоге
make --debug=b        # диагностика выбора целей
```

После `touch math.cpp` Make покажет пересборку `math.o` и повторную линковку `app`. `main.o` уже актуален.

Запускайте `make -n` перед незнакомой целью. `make -B` может занять много времени. Параллельная сборка требует корректных зависимостей.

## Конфигурация и варианты сборки

```make
BUILD ?= debug
ifeq ($(BUILD),release)
    CXXFLAGS := -std=c++17 -O2 -DNDEBUG
else
    CXXFLAGS := -std=c++17 -Wall -Wextra -g
endif
```

```bash
make clean
make BUILD=release
```

Секреты не храните в Makefile и не передавайте через командную строку в CI без защищённых переменных.

## Ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `missing separator` | Пробелы вместо TAB в рецепте | Замените отступ на TAB |
| `No rule to make target` | Нет зависимости или правила для файла | Проверьте имя и путь |
| Всегда пересобирается | Файл-цель не создан или время файлов неверное | Проверьте рецепт и зависимости |
| Параллельная сборка ломается | Пропущена зависимость между целями | Опишите реальный порядок зависимостей |
| `command not found` | Компилятор не установлен или не в `PATH` | Проверьте `command -v g++` |

## Документация

- [GNU Make manual](https://www.gnu.org/software/make/manual/)
- [GNU Make](https://www.gnu.org/software/make/)
- [CMake](CMake.md)
- [Основы C++](../programming/Cpp.md)
