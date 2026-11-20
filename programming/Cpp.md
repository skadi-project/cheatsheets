# C++ — основы

## Быстрый старт

Сохраните пример как `main.cpp`. Примеры используют C++17. На Ubuntu/Debian установите `build-essential`, на Arch пакет `gcc`, на macOS используйте Clang из Xcode Command Line Tools, на Windows подходит GCC из MSYS2 UCRT64 или WSL.

```cpp
#include <iostream>
#include <vector>

int sum(const std::vector<int>& values) {
    int total = 0;
    for (int value : values) {
        total += value;
    }
    return total;
}

int main() {
    const std::vector<int> values{2, 3, 5};
    std::cout << sum(values) << '\n';
}
```

```bash
g++ -std=c++17 -Wall -Wextra -Wpedantic main.cpp -o demo.exe
```

```bash
./demo.exe
```

Для Clang замените `g++` на `clang++`. Программа выведет `10`.

## Типы данных

| Тип | Пример | Назначение |
|---|---|---|
| `bool` | `bool ready{true};` | Логическое значение |
| `char` | `char letter{'A'};` | Символ или байт |
| `short` / `int` / `long long` | `int count{42};` | Целые числа разных диапазонов |
| `float` / `double` | `double price{1.5};` | Числа с плавающей точкой |
| `std::string` | `std::string name{"Ada"};` | Изменяемая строка, заголовок `<string>` |
| `std::array<T, N>` | `std::array<int, 3> values{1, 2, 3};` | Массив фиксированного размера, заголовок `<array>` |
| `std::vector<T>` | `std::vector<int> values{1, 2};` | Динамический массив, заголовок `<vector>` |
| `std::pair<T, U>` | `std::pair<int, int> point{3, 4};` | Пара значений, заголовок `<utility>` |
| `std::map<K, V>` | `std::map<std::string, int> counts;` | Упорядоченное отображение ключей в значения, `<map>` |
| `std::optional<T>` | `std::optional<int> value{};` | Значение, которое может отсутствовать, `<optional>` |
| `struct` / `class` | `struct Point { int x{}; };` | Пользовательский тип с полями и методами |
| `enum class` | `enum class State { ready, done };` | Типобезопасный набор вариантов |
| `T*` | `int* pointer{nullptr};` | Указатель на объект или `nullptr` |

## Синтаксис

| Конструкция | Пример | Что делает |
|---|---|---|
| Инициализация | `int count{3};` | Создаёт переменную с начальным значением |
| Вывод типа | `auto price = 1.5;` | Выводит тип `double` из выражения |
| Условие | `if (count > 0) { --count; }` | Выполняет блок по условию |
| Цикл по диапазону | `for (int n : values) { std::cout << n; }` | Перебирает элементы |
| Строка | `std::string name{"Ada"};` | Владеет текстом, заголовок `<string>` |
| Контейнер | `std::vector<int> data{1, 2}; data.push_back(3);` | Хранит массив переменного размера, заголовок `<vector>` |
| Ссылка | `int& ref = count;` | Даёт другое имя существующему объекту |
| Константная ссылка | `const auto& ref = values;` | Позволяет читать объект без копирования |
| Указатель | `int* p = &count;` | Хранит адрес объекта. Пустое значение записывается `nullptr` |
| Структура | `struct Point { int x{}; int y{}; };` | Объединяет поля и может содержать методы |
| Перечисление | `enum class State { ready, done };` | Ограничивает имена значений областью `State` |
| Шаблон | `template<class T> T twice(T x) { return x + x; }` | Позволяет вызвать функцию с разными типами |

## Время жизни и типичные ошибки

- Объекты `std::string` и `std::vector` управляют выделенной памятью автоматически. Освобождение ресурса в деструкторе объекта называют RAII.
- Передача параметра `T value` создаёт отдельный объект. `const T& value` даёт доступ без копирования, `T& value` позволяет изменить объект вызывающего кода.
- Возвращайте локальные объекты по значению. Ссылки и указатели на них после выхода из функции становятся висячими.
- Для проверки границ `std::vector` используйте `.at(i)`. Рост вектора может сделать старые ссылки и указатели на его элементы недействительными.
- При ошибках линковки проверьте, что используете `g++` или `clang++` и передали все нужные `.cpp`.

## Ссылки

- [Стандартизация C++](https://isocpp.org/std)
- [Справочник по C++ и стандартной библиотеке](https://en.cppreference.com/w/cpp.html)
- [Каталог сторонних библиотек vcpkg](https://vcpkg.io/en/packages.html)
- [Каталог сторонних библиотек Conan Center](https://conan.io/center)
