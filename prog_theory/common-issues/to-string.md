## Строки

### Число в строку

Для склеивания строки из нескольких элементов, включающих подстроки и числа, есть оператор '+' и удобный метод 'std::string', именно они представляют современные методы:
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
