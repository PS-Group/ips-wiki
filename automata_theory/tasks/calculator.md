## Задание: калькулятор

Базовые требования, выполненные в примере:
- модуль калькулятора получает на вход строку и вычисляет её как выражение, возвращая число
- разбор выражений вида ```12445 + 125-5195 + 12``` с учётом пробелов
- учёт приоритетов операторов: ```9*0 + 124*125```
- поддержка чисел с плавающей точкой: ```12.512 + 0.125 + 2.125 + 9.1```
- для некорректных выражений, таких как ```14 ++ 41```, возвращается ```NaN```

Расширенные требования (выполнять самостоятельно):
- возврат ```double``` вместо ```float```.
- кроме возврата double, в режиме отладки печатать выражения в [польской](https://ru.wikipedia.org/wiki/%D0%9F%D0%BE%D0%BB%D1%8C%D1%81%D0%BA%D0%B0%D1%8F_%D0%BD%D0%BE%D1%82%D0%B0%D1%86%D0%B8%D1%8F) или [обратной польской](https://ru.wikipedia.org/wiki/%D0%9E%D0%B1%D1%80%D0%B0%D1%82%D0%BD%D0%B0%D1%8F_%D0%BF%D0%BE%D0%BB%D1%8C%D1%81%D0%BA%D0%B0%D1%8F_%D0%B7%D0%B0%D0%BF%D0%B8%D1%81%D1%8C) нотации.
- вместо ```boost::string_ref``` можно использовать экспериментальный ```std::string_view```. Этот компонент STL пока реализован не во всех компиляторах, работоспособную замену можно взять тут ```https://github.com/sergey-shambir/string_view```.

Требуемая документация:
- зарисовка [диаграммы состояний](https://ru.wikipedia.org/wiki/%D0%94%D0%B8%D0%B0%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B0_%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D0%B9_%28%D1%82%D0%B5%D0%BE%D1%80%D0%B8%D1%8F_%D0%B0%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%BE%D0%B2%29) конечного автомата для функции parseFloat
- аналогично для функций skipSpaces, parseExprSum, parseExprMul
- общая диаграмма состояний конечного автомата калькулятора, где отдельные функции используются для перехода между состояниями

### Класс boost::string_ref

Чтобы разбирать выражение, нам придётся пробегать строку слева направо, и, возможно, делать это повторно в разных функциях. Способы реализации:
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
	float value = parseFloat(ref);
	while (true)
	{
		if (!ref.empty() && ref[0] == '+')
		{
			ref.remove_prefix(1);
			value += parseFloat(ref);
		}
		else if (!ref.empty() && ref[0] == '-')
		{
			ref.remove_prefix(1);
			value -= parseFloat(ref);
		}
		else
		{
			break;
		}
	}

	return value;
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

Для получения NaN в STL есть [std::numeric_limits<float>::quiet_NaN()](http://en.cppreference.com/w/cpp/types/numeric_limits/quiet_NaN). Для проверки наличия ошибки функция ```parseFloat``` должна проверять, что она хоть что-то распарсила. Для этого введём локальную переменную ```parsedAny```:

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

### Пропуск пробелов

Для пропуска пробелов напишем функцию skipSpaces, воспользуемся функцией [std::isspace](http://en.cppreference.com/w/cpp/string/byte/isspace):

```cpp
void skipSpaces(boost::string_ref &ref)
{
    size_t i = 0;
    while (i < ref.size() && std::isspace(ref[i]))
        ++i;
    ref.remove_prefix(i);
}

float parseFloat(boost::string_ref &ref)
{
    skipSpaces(ref);
    float value = 0;
    // [...]
}

float parseExprSum(boost::string_ref &ref)
{
	float value = parseFloat(ref);
	while (true)
	{
		skipSpaces(ref);
    // [...]
}


int main()
{
    std::string expr = "1254 + 46 - 1200";
    boost::string_ref ref(expr);
    cout << parseExprSum(ref) << endl;
    return 0;
}
```

### Обработка умножения и деления

Для обработки двух новых операций мы можем вставить посредника между parseExprSum и parseFloat. Посредник вставляется между ними, так как приоритет у операций умножения/деления выше.
- в parseExprSum заменяем вызовы parseFloat на parseExprMul
- функцию parseExprMul пишем аналогично функции parseExprSum

```cpp
float Calculator::parseExprMul(boost::string_ref &ref)
{
	float value = parseFloat(ref);
	while (true)
	{
		skipSpaces(ref);
		if (!ref.empty() && ref[0] == '*')
		{
			ref.remove_prefix(1);
			value *= parseFloat(ref);
		}
		else if (!ref.empty() && ref[0] == '/')
		{
			ref.remove_prefix(1);
			value /= parseFloat(ref);
		}
		else
		{
			break;
		}
	}

	return value;
}
```

### Обработка точки в parseFloat
Чтобы поддерживать разбор чисел вида ```1.24```, мы можем добавить второй, слегка изменённый, цикл разбора в parseFloat:
```cpp
float Calculator::parseFloat(boost::string_ref &ref)
{
	// [...]
	if (!parsedAny)
	{
		return std::numeric_limits<float>::quiet_NaN();
	}
	if (ref.empty() || (ref[0] != '.'))
	{
		return value;
	}
	ref.remove_prefix(1);
	float factor = 1.f;
	while (!ref.empty() && std::isdigit(ref[0]))
	{
		const int digit = ref[0] - '0';
		factor *= 0.1f;
		value += factor * float(digit);
		ref.remove_prefix(1);
	}

	return value;
}
```

### Прячем реализацию

Чтобы скрыть детали реализации калькулятора, мы можем создать класс (или структуру) Calculator согласно идиоме [Хранилище Функций](../../prog_theory/common-issues/vector-math.md). Новый класс должен иметь минимальный интерфейс, с документацией в виде комментариев. Пример:
```cpp
struct Calculator
{
	Calculator() = delete;

	// parses expressions like "7 / 2 + 12 - 3 * 4 + 17 - 2 * 7"
	// calculates and returns result.
	static float parseExpr(boost::string_ref &ref);

private:
	static float parseFloat(boost::string_ref &ref);
	static float parseExprMul(boost::string_ref &ref);
	static float parseExprSum(boost::string_ref &ref);
	static void skipSpaces(boost::string_ref &ref);
};
```

Реализация функции parseExpr проста: она вызывает parseExprSum. Кроме этого, она может проверить, достигнут ли конец ```ref```.
- Если не достигнут, значит, какие-то символы не обработаны, и можно вернуть NaN
- В таком случае придётся обрабатывать ещё и пробелы в конце: "1 + 2  "

### Читать далее

Остальные требования придётся выполнить самостоятельно.
- [диаграмма состояний](https://ru.wikipedia.org/wiki/%D0%94%D0%B8%D0%B0%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B0_%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D0%B9_%28%D1%82%D0%B5%D0%BE%D1%80%D0%B8%D1%8F_%D0%B0%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%BE%D0%B2%29)
- [польская нотация](https://ru.wikipedia.org/wiki/%D0%9F%D0%BE%D0%BB%D1%8C%D1%81%D0%BA%D0%B0%D1%8F_%D0%BD%D0%BE%D1%82%D0%B0%D1%86%D0%B8%D1%8F)
- [обратная польская нотация](https://ru.wikipedia.org/wiki/%D0%9E%D0%B1%D1%80%D0%B0%D1%82%D0%BD%D0%B0%D1%8F_%D0%BF%D0%BE%D0%BB%D1%8C%D1%81%D0%BA%D0%B0%D1%8F_%D0%B7%D0%B0%D0%BF%D0%B8%D1%81%D1%8C)
