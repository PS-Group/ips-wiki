## Строки

### Число в строку

Для склеивания строки из нескольких элементов, включающих подстроки и числа, есть оператор '+' и удобный метод 'std::to_string', именно они представляют современные методы:
```cpp
void DisplayStats(Player &player, sf::RenderWindow &window, sf::Font fontMain)
{
    const std::string name = player.getName();
    const int points = player.getPoints();
    std::string message = name + " got " + std::to_string(points) + " points.";
    sf::Text text(message, fontMain);
    window.draw(text);
}
```

В прежние времена для такой простой задачи использовались менее удачные методы. Например, создавался std::stringstream
- для читающего код появляется лишняя сущность, ненужная по принципу [бритвы Оккама](https://ru.wikipedia.org/wiki/%D0%91%D1%80%D0%B8%D1%82%D0%B2%D0%B0_%D0%9E%D0%BA%D0%BA%D0%B0%D0%BC%D0%B0)
```cpp
void DisplayStats(Player &player, sf::RenderWindow &window, sf::Font fontMain)
{
    const std::string name = player.getName();
    const int points = player.getPoints();
    std::stringstream message;
    message << name << " got " << points << " points.";
    sf::Text text(message.str(), fontMain);
    window.draw(text);
}
```

Другим вариантом был sprintf - небезопасный в неопытных руках, но мощный и порой очень удобный инструмент
```cpp
void DisplayStats(Player &player, sf::RenderWindow &window, sf::Font fontMain)
{
    const size_t MAX_SUFFIX_SIZE = 100;
    const std::string name = player.getName();
    char suffix[MAX_SUFFIX_SIZE];
    sprintf(suffix, " got %d points.", player.getPoints());
    sf::Text text(name + suffix, fontMain);
    window.draw(text);
}
```

### Строку в число

Простейший способ &mdash; std::stoi, std::stol, std::stoll
```cpp
#include <string>

std::wstring ws = L"456";
int i = std::stoi(ws); // convert to int
std::wstring ws2 = std::to_wstring(i); // and back to wstring
```

Следует помнить, что std::stoi может бросить два разных исключения

```cpp
#include <string>

std::string versionValid = L"1.456";
std::string versionInvalid = L".456";
std::string versionOutOfRange = L"45645254125.412";
int versionMajor = std::stoi(versionValid); // ok
int versionMajor2 = std::stoi(versionInvalid); // throws 'std::invalid_argument'
int versionMajor3 = std::stoi(versionOutOfRange); // throws 'std::out_of_range'
```

Более гибкий вариант сканирования - функция sscanf.
```cpp
namespace {
int parseInt(std::string const& text)
{
     int result = 0;
     sscanf(text.c_str(), "%d", &result);
     return result;
}

float parseFloat(std::string const& text)
{
    float result = 0;
    sscanf(text.c_str(), "%f", &result);
    return result;
}

double parseDouble(std::string const& text)
{
    double result = 0;
    sscanf(text.c_str(), "%lf", &result);
    return result;
}
}
```

Обратите внимание, что сканировать строку небезопасно &mdash; можно получить переполнение буфера. Чтобы переполнения не происходило, следует указывать максимальное число символов (на 1 меньше размера буфера)
```cpp
void security_hole(std::string const& text)
{
    char result[20];
    sscanf(text.c_str(), "%s", result);
}

void security_ok(std::string const& text)
{
    char result[20];
    sscanf(text.c_str(), "%19s", result);
}
```

С помощью sscanf можно разобрать, например, дату в формате "31-12-2014". Для этого придётся проверять число, возвращённое функцией sscanf, потому что sscanf возвращает число реально считанных переменных.
```cpp
struct Date
{
    int year = 0;
    int month = 0;
    int day = 0;
};

bool parseDate(std::string const& text, Date &date)
{
    Date temp;
    if (3 == sscanf(text.c_str(), "%d-%d-%d", &temp.day, &temp.month, &temp.year))
    {
        date = temp;
        return true;
    }
    return false;
}
```
