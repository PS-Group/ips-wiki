## Программирование по контракту в языке C++

Design by Contract &mdash; принцип программирования. Его иногда ошибочно считают технологией и стремяться делать строго-как-в-учебниках, но это непрактичный подход.

### Контракты функций
При написании функций, имеющих математический смысл, следует помнить про необходимость вводить и соблюдать контракты, то есть спецификации функций и проверки этих спецификаций. Вот пример плохого кода (без контрактов):
- GameMath.h
```cpp
#pragma once

struct Math
{
    Math() = delete;
    static int Random(int a, int b);
}
```
- GameMath.cpp
```cpp
int Math::Random(int max)
{
    return rand() % max;
}
```
- код, использующий Math
```cpp
void foo()
{
    const int MIN_SPEED = 1;
    const int MAX_SPEED = 4;
    int sparkleSpeed = Math::Random(MIN_SPEED, MAX_SPEED);
    // ...
}
```

Читатель, анализирующий код функции ```foo()```, будет задаваться вопросами:
- Что означают a и b, почему туда передаются ```MIN_SPEED``` и ```MAX_SPEED```?
- Могу ли я через ```Math::Random(1, 10)``` получить число 10?
- Что вернёт ```Math::Random(0, -7)```?

В итоге читателю для анализа ```foo()``` придётся изучить код других функций. Проблема легко устраняется, если ввести:
- интуитивно понятные названия параметров
- контракт в виде комментария в заголовочном файле
- проверку соблюдения контракта с помощью макроса assert

```cpp
#pragma once

struct Math
{
    Math() = delete;
    // Returns value in range [min, max].
    // 'max' must be more than 'min'.
    static int Math::Random(int min, int max);
}
```
- GameMath.cpp
```cpp
int Math::Random(int min, int max)
{
    assert(min > max);
    return min + rand() % (max - min + 1);
}
```

### Немного теории
Контракты &mdash; часть формальных методов проверки корректности программ.

Опытные разработчики могут использовать другие реализации идеи контрактов:
- [перечисленные ниже способы не рекомендуются для новичков](cpp-limitations.md)
- для объектно-ориентированного кода: модульное тестирование (unit testing)
- для статического анализа кода: [контракты](http://www.open-std.org/JTC1/SC22/WG21/docs/papers/2015/n4415.pdf) из C++ 2017
- для метапрограммирования: концепты из стандарта C++ 2017
- грамотное проектирование API с применением ООП и ФП

### Читать далее
- макрос assert: http://en.cppreference.com/w/cpp/error/assert
- [Design by Contract](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D0%BD%D1%82%D1%80%D0%B0%D0%BA%D1%82%D0%BD%D0%BE%D0%B5_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5) на википедии.
