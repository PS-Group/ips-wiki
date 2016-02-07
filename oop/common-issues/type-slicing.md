### Срезка типа (type slicing)

### Срезка типа при поимке исключения

В примере ниже исключение ловится по значению, а не по ссылке. В результате информация о типе теряется, и невозможно определить тип объекта с помощью RTTI либо путём вызова виртуальных методов.
```cpp
try
{
	throw std::runtime_error("Runtime error");
}
catch (std::exception e)	// The information about runtime_error has been sliced
{
	std::cout << e.what() << ", type name: " << typeid(e).name() << "\n\n";
}
```

### Читать далее
- [пример срезки типа при поимке исключения](https://github.com/alexey-malov/oop2015/blob/master/slicing/slicing/main.cpp)