## Обработка коллекции разнородных объектов

Допустим, стоит задача &mdash; все компоненты UI, сохранённые в контейнере std::vector, обойти и нарисовать путём вызова метода ```Draw(sf::Window &window)```. Типовое решение:
```cpp
class UiComponent
{
public:
  virtual void Draw(sf::Window &window);
};

class UiButton : public UiComponent
{
public:
  virtual void Draw(sf::Window &window);
};

class UiLabel : public UiComponent
{
public:
  virtual void Draw(sf::Window *window);
};

void DrawAll(std::vector<UiComponent> &components, sf::Window &window)
{
  for (UiComponent &x : components)
    x.Draw(window);
}
```

В данном решении кроется ошибка: функция UiLabel::DrawAll не вызовет функцию UiLabel::Draw. Функция UiLabel::Draw не перегружает функцию UiComponent::Draw, потому что у них *разные сигнатуры*, т.е. они принимают разный список параметров. С точки зрения языка, это объявление нового метода, а не полиморфная перегрузка UiComponent::Draw).
Есть два решения:
- полумера &mdash; объявить UiComponent::Draw() чисто виртуальным методом, что сделает класс UiComponent абстрактным. Компилятор будет выдавать ошибки компиляции при попытке создать объект абстрактного класса.
- чистое решение &mdash; добавить спецификатор override ко всем перегрузкам Draw. Компилятор проверит, что метод с пометкой override что-то перегружает. Если вы забыли добавить virtual в базовом классе или объявили разный список параметров в базовом классе и подклассе, компилятор выдаст ошибку, что метод с пометкой override ничего не перегрузил.

Исправленный вариант с чисто виртуальной UiComponent::Draw и спецификаторами override у перегрузок:
```cpp
class UiComponent
{
public:
  virtual void Draw(sf::Window &window) = 0;
};

class UiButton : public UiComponent
{
public:
  void Draw(sf::Window &window) override;
};

class UiLabel : public UiComponent
{
public:
  void Draw(sf::Window *window) override;
};

void DrawAll(std::vector<UiComponent> &components, sf::Window &window)
{
  for (UiComponent &x : components)
    x.Draw(window);
}
```

### Читать далее
- [спецификаторы override, final, default, delete](vk.com/im?sel=47855698)
