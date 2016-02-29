# Наследование

### Зачем?

Наследование решает узкие проблемы:
- даёт путь реализации полиморфизма и паттернов проектирования
- иногда сжимает код без потери качества

Прямая альтернатива &mdash; композиция. В реальности композиция используется намного чаще, и не требует ООП: любой структурный язык уровня C уже поддерживает композицию. То есть, наследование &mdash; объекто-ориентированная замена композиции, направленная на решение узких проблем в конкретных случаях. Во всех остальных случаях композиция лучше наследования.

### Наследование класса

Представим три класса:

В псевдокоде:
```cpp
class CheckButton
{
    void DoOnClick(function<void()> handler);
    void Resize(Vector2f size);
}

class PushButton
{
    void Resize();
}
```

- мы передаём ```handler``` по ссылке для избавления от лишнего копирования, потому тип ```std::function``` внутри своей реализации хранит указатель на область динамической памяти, где хранится _замыкание_ функции.
- мы могли бы передать размер по константной ссылке, чтобы избежать лишнего копирования. Но размер указателя (и ссылки) на 64-битной платформе &mdash; 8 байт, размер ```sf::Vector2f``` равен размеру 2 чисел float, то есть те же 8 байт. То есть выигрыша нет.




```cpp
class Button
{
    void DoOnClick(std::function<void()> const& handler);
    void Resize(sf::Vector2f size);
}
```

### Наследование интерфейса

##### в псевдокоде
```cpp
interface Shape
{
    void Draw(sf::RenderTarget &window);
    void TestHit(sf::Vector2f point);
}

class RectangleShape : Shape
{
    void Draw(sf::RenderTarget &window) override;
    void TestHit(sf::Vector2f point) override;
}
```
Нюансы:

- мы могли бы передать точку по константной ссылке, чтобы избежать лишнего копирования. Но размер указателя (и ссылки) на 64-битной платформе &mdash; 8 байт, размер ```sf::Vector2f``` равен размеру 2 чисел float, то есть те же 8 байт. То есть выигрыша нет.
- мы могли бы передать ```sf::RenderWindow``` в функцию Draw. Но если ```sf::RenderWindow``` наследуется от ```sf::RenderTarget```, то мы вполне можем передать ```sf::RenderTarget```. Это может дать нам бесплатную поддержку дополнительных способов вывода графики &mdash; например, вывод в Bitmap вместо вывода на дисплей.

##### на языке C++

```cpp
class Shape
{
public:
    virtual ~Shape() = default;
    virtual void Draw(sf::RenderTarget &window) = 0;
    virtual void TestHit(sf::Vector2f point) = 0;
};

class RectangleShape : public Shape
{
public:
    void Draw(sf::RenderTarget &window) override;
    void TestHit(sf::Vector2f point) override;
};
```
Нюансы:

- виртуальный деструктор гарантирует, что удаление указателя на Shape приведёт к вызову деструкторов всех подклассов, поскольку вызов виртуального деструктора косвенный, а не прямой.

### Композиция

Наследование &mdash; это альтернатива композиции, направленная на решение узких проблем. Для решения большинства проблем вполне хватит композиции, то есть включения одного типа данных в состав другого:
```cpp
class MenuScene
{
public:
    MenuScene(sf::Vector2f size);

private:
    Button m_btnPause;
    LabelButton m_lblScore;
    sf::Sprite m_background;
};
```


