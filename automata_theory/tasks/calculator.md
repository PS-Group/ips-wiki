## Задание: калькулятор

Требования:
- модуль калькулятора получает на вход строку и вычисляет её как выражение.
- на выходе &mdash; число типа ```double```.
- разбор выражений вида ```12445 + 125-5195 + 12``` с учётом пробелов
- учёт приоритетов операторов: ```9*0 + 124*125```
- поддержка чисел с плавающей точкой: ```12.512 + 0.125 + 2.125 + 9.1```
- для некорректных выражений возвращается ```NaN```

### Пример простого калькулятора

На вход подаются выражения в виде строки. Калькулятор вычисляет и выводит значение выражения. Требования, которые мы выполним в примере:
- вычислять выражения вида ```124+412+13```
- вычислять выражения вида ```124+412*13```
- поддерживать операторы ```+ - * /```
- для ошибочных выражений, таких как ```14++41```, возвращать ```NaN``` (not a number)

### Класс boost::string_ref

Чтобы разбирать выражение, нам придётся пробегать строку слева направо, и, возможно, делать это в нескольких случаях. Способы реализации:
- хранить в паре ```std::string text, size_t position```, для получения очередного символа использовать ```text[position]```, для сдвига вправо использовать ```++position``` и сравнение ```position``` с ```text.size()```
- использовать класс boost::string_ref, делающий то же самое
- использовать класс из стандарта C++ 2017 &mdash; [std::string_view](http://en.cppreference.com/w/cpp/experimental/basic_string_view)

В примере остановимся на ```boost::string_ref```. Вот так будет выглядеть функция для разброра чисел вида "124", "15", "012".

```cpp
#include <boost/utility/string_ref.hpp>
#include <cctype>

float parseFloat(boost::string_ref &ref)
{
    float value = 0;
    while (!ref.empty() && std::isdigit(ref[0]))
    {
        const int digit = ref[0] - '0';
        value = value * 10.0f + float(digit);
        ref.remove_prefix(1);
    }

    return value;
}
```

Протестировать код легко:
```cpp
int main()
{
    std::string expr = "152+42+512";
    boost::string_ref ref(expr);
    ref.remove_prefix(4);
    std::cout << parseFloat(ref) << std::endl;
    return 0;
}
```

Заметим, что мы не можем использовать [std::stof](http://en.cppreference.com/w/cpp/string/basic_string/stof), т.к. он не обрабатывает ```boost::string_ref```, а нам нужна ссылка для последовательного движения по выражению слева направо.

### Разбор сложения и вычитания

Пример выражения: ```1254+46-1200```. Можно вычислить его так:
- разобрать 1254 с parseFloat и сохранить в ```x```
- разобрать 46 c parseFloat и сохранить в ```y```
- разобрать 1200 с parseFloat и сохранить в ```z```
- вычислить ```x + y - z```

Чтобы сделать функцию универсальной, мы можем выбирать дальнейшее действие по очередному символу и использовать рекурсию. Пример ниже должен вывести ```100```.

```cpp
float parseExprSum(boost::string_ref &ref)
{
    float left = parseFloat(ref);
    if (!ref.empty() && ref[0] == '+')
    {
        ref.remove_prefix(1);
        return left + parseExprSum(ref);
    }
    if (!ref.empty() && ref[0] == '-')
    {
        ref.remove_prefix(1);
        return left - parseExprSum(ref);
    }
    return left;
}


int main()
{
    std::string expr = "1254+46-1200";
    boost::string_ref ref(expr);
    cout << parseExprSum(ref) << endl;
    return 0;
}
```

### Возврат NaN при ошибке

Для получения NaN в STL есть [std::numeric_limits<float>::quiet_NaN()](http://en.cppreference.com/w/cpp/types/numeric_limits/quiet_NaN).
Изменим функцию ```parseExprSum```:
```cpp
float parseExprSum(boost::string_ref &ref)
{
    // [...]
    return left;
}

/// transform to...

float parseExprSum(boost::string_ref &ref)
{
    // [...]
    if (!ref.empty())
    {
        return std::numeric_limits<float>::quiet_NaN();
    }
    return left;
}

// checking result...

int main()
{
    std::string expr = "1254+46-1200!";
    boost::string_ref ref(expr);
    cout << parseExprSum(ref) << endl;
    return 0;
}
```

Функция ```parseFloat``` должна проверять, что она хоть что-то распарсила. Для этого введём локальную переменную ```parsedAny```:

```cpp
float parseFloat(boost::string_ref &ref)
{
    float value = 0;
    bool parsedAny = false;
    while (!ref.empty() && std::isdigit(ref[0]))
    {
        parsedAny = true;
        const int digit = ref[0] - '0';
        value = value * 10.0f + float(digit);
        ref.remove_prefix(1);
    }
    if (!parsedAny)
    {
        return std::numeric_limits<float>::quiet_NaN();
    }

    return value;
}
```
