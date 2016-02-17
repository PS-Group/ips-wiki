# Проектирование API в современном C++

## Разработка удобных функций

А вы уверены, что написанные вами функции

- легко вызвать корректно?
- трудно вызвать некорректно?
    - невовремя
    - не с теми параметрами
- выполняются эффективно?
    - без лишнего копирования параметров
    - без разделения связанных параметров
    - без лишнего выделения ресурсов
- легко соединить с другими функциями?
- легко использовать в коде более высокого уровня?
    - там, где человек пишет код в более абстрактных терминах
    
#### Передача входных/выходных параметров

Как это было принято раньше:

- Входные параметры
    - данные по 4-8 байт: по значению
    - большие или трудно копируемые данные: по ссылке
- Выходные параметры
    - маленькие: возврат через ```return```
    - созданные объекты: возврат указателя через ```return```
    - большие или трудно копируемые: через ссылку
- Входные-выходные параметры
    - через ссылку
    
С современной семантикой перемещения (move semantics) принято так:

- Входные параметры
    - данные по 4-8 байт: по значению
    - объекты, меняющие владельца: по значению unique_ptr / shared_ptr
    - остальные: по ссылке
- Выходные параметры
    - по значению (должно произойти перемещение без копирования)
- Входные выходные параметры
    - ???

##### По меркам C++98, это хороший код:
```cpp
void DrawImage(Gdiplus::Image const& image, WTL::CRect const& bounds);

bool CFontMetrics::MeasureTextSize(std::string const& text, WTL::CRect & bounds);

float GetLength(sf::Vector2f const& v);

void LoadImage(std::string const& path, Gdiplus::Bitmap &* pImage);
```

##### По меркам C++98, это **плохой** код:
```cpp
// неконстантный указатель вместо константной ссылки
// 4 параметра x, y, w, h не соединены в один
void DrawImage(Gdiplus::Image *pImage, int x, int y, int w, int h);

// строка передаётся по значению
// а что, если функция упадёт при bounds=nullptr?
void CFontMetrics::MeasureTextSize(std::string text, WTL::CRect *pBounds);

// нет смысла передавать 4-х байтовый float по ссылке
float GetLength(float const& x, float const& y);

// const char* вместо std::string
// "**" вместо "*&"
void LoadImage(const char *szPath, Gdiplus::Bitmap **ppBitmap);

// будет утечка, если никто не сохранит результат вызова функции
Gdiplus::Bitmap *LoadImage(std::string const& path);
```

##### Благодаря C++11 функция LoadImage становится гораздо лучше!
```cpp
// если забыть сохранить результат, картинка тут же удалится
std::unique_ptr<Gdiplus::Bitmap> LoadImage(std::string const& path);
```

##### С учётом семантики перемещения, передача ```std::unique_ptr``` по значению означает смену владельца объекта:
```
auto icon = std::make_unique<CImage>(path);
auto decoratedIcon = std::make_unique<CGrayImage>(std::move(icon));
```

#### Отказ от выходных параметров

Функция std::getline в составе <iostream> была разработана для C++98. Вот её объявление:
```cpp
std::istream & getline(std::istream &, std::string &);
```

Так ей пользуются:
```cpp
std::string line;
if (std::getline(std::cin, line))
    use_line(line);
```

Допустим, мы "улучшили" getline с помощью семантики перемещения
```cpp
// Объявление
std::string getline(std::istream &);

// Применение
void f()
{
    use_line(std::getline(std::cin));
}
```

Появилась проблема: при повторных вызовах в цикле функция новая std::getline не переиспользует память, как это делала оригинальная функция:
```cpp
int main()
{
    std::string line;
    while (std::getline(std::cin, line)) {
        use_line(line);
    }
}
```

##### Решение: концепция диапазонов (ranges)

```cpp
// lines_range::iterator читает новую строку при вызове "++"
lines_range getlines(std::istream &);

void f()
{
    for (std::string const& line : getlines(std::cin))
        use_line(line);
}
```

#### Входные-выходные параметры

Входные-выходные параметры означают, что функция реализует алгоритм с состоянием. Параметр может переносить:
- текущее состояние
- кеш
- заранее вычисленные данные
- буферы

В любом случае, следует инкапсулировать эти данные в объекте, реализующем алгоритм. Примеры:
- string_view в составе STL
- boyer_moore в составе Boost

- Входные параметры
    - данные по 4-8 байт: по значению
    - объекты, меняющие владельца: по значению unique_ptr / shared_ptr
    - остальные: по ссылке
- Выходные параметры
    - по значению (должно произойти перемещение без копирования)
- Входные-выходные параметры
    - Хранить состояние в полях объекта, выполняющего алгоритм (в this)
    - Начальное состояние передаётся в объект по значению через конструктор
    
#### Rvalue-ссылки в шаблонных функциях

В шаблонных функциях вместо двух версий с передачей по значению и передачей по константной ссылке можно использовать одну версию функции с rvalue-ссылкой:
```cpp
// старый код
template <class TQueue, class TTask>
void Enqueue(TQueue &q, TTask t)
{
    q.Enqueue(t);
}

template <class TQueue, class TTask>
void Enqueue(TQueue &q, TTask const& t)
{
    q.Enqueue(t);
}

// новый код
template <class TQueue, class TTask>
void Enqueue(TQueue &q, TTask && t)
{
    q.Enqueue(t);
}

void f()
{
    TaskQueue q;
    Task t = MakeTask();
    Enqueue(q, t);
}
```

#### Perfect forwarding в шаблонных функциях
Функция std::inwoke позволяет избежать копирования аргументов везде, где возможно. Она появится в стандарте C++ 2017 и будет выглядеть примерно так:

template <class TFunction, class ...Args>
auto invoke( TFunction && fn, Args && ... args)
{
    return std::forward<TFunction>(fn)(std::forward<Args>(args)...);
}

## Разработка понятных классов

А вы уверены, что ваши классы

- учитывают семантику перемещения?
- хорошо сочетаются с другими типами данных?
- могут быть созданы везде?
    - на стеке
    - в куче
    - в статической области памяти
    - свёрнуты в константных выражениях во время компиляции
- хорошо сочетаются с алгоритмами на контейнерах в &lt;algorithm&gt;?
- помещаются в map и unordered_map как ключи?

#### TODO: закончить

## "Модули"

#### TODO: закончить

## Читать далее
- [Выходные параметры, семантика перемещения и алгоритмы с состояниями (ericniebler.com)](http://ericniebler.com/2013/10/13/out-parameters-vs-move-semantics/)
