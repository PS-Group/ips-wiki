# OpenGL приложение с SDL2

В данном уроке мы научимся создавать окно видеоигры с помощью SDL2, и очищать окно путём заливки одним цветом средствами OpenGL. Также мы напишем основной цикл игры, использующей мультимедийную библиотеку SDL2.

## Введение

Для программирования 3D-графики вам потребуется программный интерфейс 3D-рисования. За прошедшие десятилетия наибольшее распространение получили два стандарта &mdash; OpenGL и DirectX. OpenGL даёт возможность писать код, работающий на любой платформе и любом устройстве, поэтому мы сосредоточимся на изучении OpenGL.

## Что такое OpenGL

OpenGL &mdash; это спецификация API для рисования трёхмерной графики. Он позволяет переслать данные для рисования одного кадра, которые затем будут растеризованы в двумерный кадр силами конкретной реализации OpenGL. После растеризации кадр можно отправить для вывода в рамке окна или получить его как изображение.

В основном OpenGL оперирует треугольниками, изображениями и состояниями драйвера рисования.

![Иллюстрация](figures/triangle_rasterization.png)

На современных платформах API OpenGL реализует либо драйвер видеокарты, либо программный растеризатор в составе операционной системы. Реализации в драйвере работают быстрее, потому что все сложные вычислительные задачи они переносят на видеокарту. Видеокарта содержит множество процессорных ядер, хорошо приспособленных для параллельной обработки множества векторов и матриц, поэтому рисование трёхмерного мира она выполняет гораздо быстрее центрального процессора.

Возможности OpenGL сильно зависят от операционной системы и от производителя драйвера. OpenGL на Linux и на Windows имеют разные возможности. OpenGL в драйверах от NVIDIA и в драйверах от Intel также различаются. Тем не менее, можно писать код, одинаково качественно работающий на любой реализации OpenGL.

## Версии OpenGL

Разработка стандарта OpenGL началась в 1992 году на основе существовавших ранее разработок компании Silicon Graphics. С тех пор вышли версии OpenGL 1.1, OpenGL 1.2, а затем OpenGL 2.x, OpenGL 3.x и серия OpenGL 4.x: от 4.0 до 4.6.

За это время взгляды на 3D-графику изменились. Было обнаружено, что программный интерфейс, разработанный для OpenGL 1.0, имеет недостаточную гибкость и потворствует потерям производительности при рисовании графики. Начиная с OpenGL 3.0, была представлена полностью новая модель программирования с использованием OpenGL, и старый способ был объявлен устаревшим, но оставлен ради обратной совместимости.

В наших уроках мы будем опираться на возможности OpenGL 3.0 и выше, в том числе применим новую модель рисования 3D-графики. Следует учесть, что на платформе Windows функции версий OpenGL выше 1.1 нужно получать через механизм расширений. Это ограничение не действует на остальных платформах и может быть прозрачно скрыто от программиста, как мы покажем позднее.

## От SDL2 к OpenGL

OpenGL работает с графическим контекстом OpenGL. Проблема в том, что OpenGL сам не умеет создавать свой контекст &mdash; это ограничение наложено для максимальной кроссплатформенности и гибкости. Для создания контекста OpenGL нам нужно:

- создать окно в оконной операционной системе
- попросить операционную систему создать для этого окна контекст OpenGL, реализуемый драйвером или программно &mdash; на усмотрение ОС

Детали взаимодействия с системой может спрятать мультимедийная библиотека, такая как SDL2, SFML, Cocos2d-x, Cinder или OpenScehegraph. Мы будем использовать SDL2, потому что она

- гибкая и простая, разработанная экспертами в компьютерной графике и разработке игр
- хорошо проверенная и надежная, широко применяемая в индустрии
- имеет крайне либеральную открытую лицензию

Базовый пример кода на SDL2 приведён в документации функции [SDL_GL_CreateContext](wiki.libsdl.org/SDL_GL_CreateContext):

```cpp
// Специальное значение SDL_WINDOWPOS_CENTERED для x и y заставит SDL2
// разместить окно в центре монитора по осям x и y.
// Здесь для примера используется 0, т.е. окно появится в левом верхнем углу экрана.
// Для использования OpenGL вы ДОЛЖНЫ указать флаг SDL_WINDOW_OPENGL.
SDL_Window *window = SDL_CreateWindow(
    "SDL2/OpenGL Demo", 0, 0, 640, 480, SDL_WINDOW_OPENGL);
    
// Создаём контекст OpenGL, связанный с окном.
SDL_GLContext glcontext = SDL_GL_CreateContext(window);
// Теперь мы можем очистить область окна средствами OpenGL
glClearColor(0,0,0,1);
glClear(GL_COLOR_BUFFER_BIT);
// В конце - вывод нарисованного кадра в окно на экране
SDL_GL_SwapWindow(window);
    
// После окончания работы программы, SDL_GLContext должен быть удалён
SDL_GL_DeleteContext(glcontext);
```

## Основной цикл программы

Будучи библиотекой, а не фреймворком, SDL2 не навязывает программисту ни стиль программирования, ни архитектуру программы. Поэтому, основной цикл следует написать самостоятельно. Простейший вариант выглядит так:

```cpp
SDL_Event event;
bool running = true;
while (running)
{
    while (SDL_PollEvent(&event) != 0)
    {
        if (event.type == SDL_QUIT)
        {
            running = false;
        }
    }
    // Заливка кадра чёрным цветом средствами OpenGL
    glClearColor(0,0,0,1);
    glClear(GL_COLOR_BUFFER_BIT);
    // Обновление и рисование сцены
    UpdateCurrentScene();
    DrawCurrentScene();
    // В конце - вывод нарисованного кадра в окно на экране
    SDL_GL_SwapWindow(window);
}
```

## Отслеживаем интервалы времени

Обновление игрового состояния не должно зависеть от реальной частоты кадров, которую может обеспечить устройство. Поэтому функция `UpdateCurrentScene` должна принимать время, прошедшее с момента предыдущего обновления. Реализовать это можно с помощью замера интервалов времени через [std::chrono::system_clock](http://en.cppreference.com/w/cpp/chrono):

```cpp
// Через системные часы получаем объект типа time_point.
auto lastTimePoint = std::chrono::system_clock::now();

while (running)
{
    // ... выполняем предшествующие обновлению операции
    
    // Получаем второй момент времени (после итерации).
    auto newTimePoint = std::chrono::system_clock::now();
    auto dtMsec = std::chrono::duration_cast<std::chrono::milliseconds>(newTimePoint - lastTimePoint);
    lastTimePoint = newTimePoint;
    float dtSeconds = 0.001f * float(dtMsec.count());
    
    // Обновление и рисование сцены
    UpdateCurrentScene(dtSeconds);
    DrawCurrentScene();
    // В конце - вывод нарисованного кадра в окно на экране
    SDL_GL_SwapWindow(window);
}
```

## Библиотека линейной алгебры GLM

В компьютерной графике широко применяется линейная алгебра с векторами из 2-х, 3-х, 4-х элементов, матрицам 3x3 и 4x4, квантерионами. При этом вычисления с применением линейной алгебры происходят как на центральном процессоре в приложении, так и на видеокарте, управляемой видеодрайвером. OpenGL 3.0 не содержит в себе полноценного API для работы с векторами и матрицами, но существует библиотека GLM (OpenGL Mathematics library), которая предоставляет удобные C++ классы.

Как мы увидим позже, классы библиотеки GLM похожи на типы данных специального языка GLSL, используемый вместе с OpenGL для написания шейдеров. Сейчас мы просто будем использовать GLM как удобную и надёжную библиотеку:

```cpp
#include <glm/vec2.hpp>

void ShowWindow(glm::vec2 const& size)
{
    // Реализация показа окна.
}
```

## Соединяем вместе

- Для абстрагирования создания окна и контекста мы заведём класс `CAbstractWindow`, который будет предоставлять приложению "шаблонные методы" OnUpdateWindow() и OnDrawWindow().
- Также мы применим идиому [pointer to implementation](https://habrahabr.ru/post/118010/), чтобы спрятать структуры SDL2 от пользователя класса `CAbstractWindow`.
- Класс `CAbstractWindow` унаследован приватно от `boost::noncopyable`, чтобы запретить ненамеренное копирование объекта окна.
- Для игр и трёхмерных приложений обычно удобнее фиксировать размер окна или даже раскрыть его на весь экран. Поэтому, в классе `CAbstractWindow` мы пока не будем думать об изменении размера окна.

#### Листинг AbstractWindow.h

```cpp
#pragma once
    
#include <memory>
#include <boost/noncopyable.hpp>
#include <glm/fwd.hpp>
#include <SDL2/SDL_events.h>
    
class CAbstractWindow : private boost::noncopyable
{s
public:
    CAbstractWindow();
    virtual ~CAbstractWindow();
    
    void ShowFixedSize(glm::vec2 const& size);
    void DoGameLoop();
    
protected:
    virtual void OnWindowEvent(const SDL_Event &event) = 0;
    virtual void OnUpdateWindow(float deltaSeconds) = 0;
    virtual void OnDrawWindow() = 0;
    
private:
    class Impl;
    std::unique_ptr<Impl> m_pImpl;
};
```

#### Листинг AbstractWindow.cpp

```cpp
#include "AbstractWindow.h"
#include <SDL2/SDL_video.h>
#include <GL/gl.h>
#include <glm/vec2.hpp>
#include <chrono>
#include <type_traits>

namespace
{
const char WINDOW_TITLE[] = "SDL2/OpenGL Demo";

// Используем unique_ptr с явно заданной функцией удаления вместо delete.
using SDLWindowPtr = std::unique_ptr<SDL_Window, void(*)(SDL_Window*)>;
using SDLGLContextPtr = std::unique_ptr<void, void(*)(SDL_GLContext)>;

class CChronometer
{
public:
    CChronometer()
        : m_lastTime(std::chrono::system_clock::now())
    {
    }

    float GrabDeltaTime()
    {
        auto newTime = std::chrono::system_clock::now();
        auto timePassed = std::chrono::duration_cast<std::chrono::milliseconds>(newTime - m_lastTime);
        m_lastTime = newTime;
        return 0.001f * float(timePassed.count());
    }

private:
    std::chrono::system_clock::time_point m_lastTime;
};
}

class CAbstractWindow::Impl
{
public:
    Impl()
        : m_pWindow(nullptr, SDL_DestroyWindow)
        , m_pGLContext(nullptr, SDL_GL_DeleteContext)
    {
    }

    void ShowFixedSize(glm::vec2 const& size)
    {
        // Специальное значение SDL_WINDOWPOS_CENTERED вместо x и y заставит SDL2
        // разместить окно в центре монитора по осям x и y.
        // Для использования OpenGL вы ДОЛЖНЫ указать флаг SDL_WINDOW_OPENGL.
        m_pWindow.reset(SDL_CreateWindow(WINDOW_TITLE, SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED,
                                         size.x, size.y, SDL_WINDOW_OPENGL));

        // Создаём контекст OpenGL, связанный с окном.
        m_pGLContext.reset(SDL_GL_CreateContext(m_pWindow.get()));
    }

    void Clear()
    {
        // Заливка кадра чёрным цветом средствами OpenGL
        glClearColor(0,0,0,1);
        glClear(GL_COLOR_BUFFER_BIT);
    }

    void SwapBuffers()
    {
        // Вывод нарисованного кадра в окно на экране.
        // При этом система отдаёт старый буфер для рисования нового кадра.
        // Обмен двух буферов вместо создания новых позволяет не тратить ресурсы впустую.
        SDL_GL_SwapWindow(m_pWindow.get());
    }

private:
    SDLWindowPtr m_pWindow;
    SDLGLContextPtr m_pGLContext;
};

CAbstractWindow::CAbstractWindow()
    : m_pImpl(new Impl)
{
}

CAbstractWindow::~CAbstractWindow()
{
}

void CAbstractWindow::ShowFixedSize(const glm::vec2 &size)
{
    m_pImpl->ShowFixedSize(size);
}

void CAbstractWindow::DoGameLoop()
{
    SDL_Event event;
    CChronometer chronometer;
    bool running = true;
    while (running)
    {
        while (SDL_PollEvent(&event) != 0)
        {
            if (event.type == SDL_QUIT)
            {
                running = false;
            }
            else
            {
                OnWindowEvent(event);
            }
        }
        // Очистка буфера кадра, обновление и рисование сцены, вывод буфера кадра.
        if (running)
        {
            m_pImpl->Clear();
            const float deltaSeconds = chronometer.GrabDeltaTime();
            OnUpdateWindow(deltaSeconds);
            OnDrawWindow();
            m_pImpl->SwapBuffers();
        }
    }
}
```

#### Листинг main.cpp
```cpp
#include "AbstractWindow.h"
#include <glm/vec2.hpp>

class CWindow : public CAbstractWindow
{
protected:
    void OnWindowEvent(const SDL_Event &event) override
    {
        (void)event;
    }

    void OnUpdateWindow(float deltaSeconds) override
    {
        (void)deltaSeconds;
    }

    void OnDrawWindow() override
    {
    }
};

int main()
{
    CWindow window;
    window.ShowFixedSize({800, 600});
    window.DoGameLoop();
}
```

