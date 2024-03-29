+++
title = 'Изучая язык C в 2024. Часть 1'
date = 2024-02-20T22:31:05+03:00
+++

## Введение

Вероятно, каждого программиста на скриптовых ~~ненастоящих~~ языках программирования рано или поздно снедает чувство неполноценности.

Данный материал, вероятно, будет скучным для того, кто давно пишет программы на Си, или изучал его в университете.
Но также мне кажется, что среди людей, чей путь в программировании начался с **JavaScript** или **Python** (а может и с **Java**),
кто-то, так же как и я, воможно испытывает интерес к тому, как все устроено там, внизу, гораздо ближе к реальной кремниевой машине.

Все это (плюс, возможно, [интервью Джоржа Хоца у Лекса Фридмана](https://www.youtube.com/watch?v=XlvfHOrF26M), мотивировало меня к изучению Си. Все мои потуги я буду стараться задокументировать в этом блоге в виде серии заметок при изучении языка Си.

## Часть 1. Различие между указателями и массивами

Первое, что отмечают Керниган и Ритчи в своей знаменитой "C Programming Language", массивы и указатели очень близки по смыслу друг к другу.
Поэтому, у меня сразу воник вопрос, в чем именно заключается различие между этими двумя фундаментальными понятиями языка Си, и когда нужно использовать указатели вместо массивов?
Мои заметки ниже являются попыткой разобраться в этом вопросе.

### Определение и объявление

Массивы - это коллекции элементов, хранящихся в памяти в одном блоке друг за другом.
При объявлении массивов, мы специфицируем тип и количесво хранимых элементов
(хотя в отдельных случаях, например при объявлении строк, значение размера можно опустить):

```c
int arr[5]; // Обявление массива из 5 элементов типа Integer
```

Указатель - это переменная, которая содержит адрес памяти, и тип хранимоного по этому адресу значения.

```c
int *ptr; // Объявление указателя на Integer
```

### Выделение памяти

При стандартном объявлении массивов, память для них выделяется во время компиляции. 
Если массив объявляется внутри функции, то память для него будет выделена в стеке.

```c
void function() 
{
    int arr[10]; /* Память выделена в стеке */
}
```

Если массив был объявлен за пределами функций, или с ключевым словом `static`

```c
int globalArr[10]; /* Память выделена в global/static области памяти */

void function()
{
    static int staticArr[10]; /* Память также выделена в global/static области памяти */
}
```

Массивы также могут быть объявлены в рантайме, при использовании функций для динамического выделения памяти в куче.
Под кучей (**heap**) обычно понимается область памяти за пределами стека, доступ к которой осуществляется при помощи 
функций вроде `malloc` и `calloc`, которые запрашивают у операционной системы выделение необходимого объема этой самой памяти.
Это позволяет программе не знать заранее точный размер массива, и выделять необходимое место в ходе выполнения программы.
При этом выделенная память должна быть освобождена после использования, и программист должен позаботиться об этом самостоятельно.
При таком подходе, мы получаем указатель на первый адрес ячейки, где память была выделена:

```c
int *arr = (int*)malloc(10 * sizeof(int)); // Аллокация в куче
```

### Использование

В целом, указатели являеются более обобщенным и универсальным иструментом. По факту, все операции, которые 
проводятся с массивами, подразумевают использование программой указателей. И также все действия, производимые с массивами,
можно воспроизвести, обращаясь к массиву как к указателю, используя при этом адресную арифметику.

Важно отметить, что по факту массив и является указателем, т.е. переменная `arr` из примера выше указывает на начало блока памяти,
выделенного под массив.

```c
void function() 
{
    int arr[10];
    arr == &a[0] /* переменная массива равна адресу памяти первого элемента */
}
```

Указатели важны, когда мы не можем определить заранее необходимый объём памяти, который необходимо использовать.
Например, при реализации связных списов, графов и деревьев.
Во всех случаях, когда мы можем определить размеры используемых данных заранее (или хотя бы оценить необходимое место "с запасом"), лучше использовать 
массивы.

Еще одной важной особенностью является то, что массивы, при передачи их в качестве аргумента в функцию, они "деграгируют" до указателя и
теряют информацию о размерности (что, при вызове `sizeof` даст нам размер указателя вместо размера массива). Поэтому следить за размером массива
при передаче его в функцию является задачей программиста (стандартным решением будет передавать размер массива в виде дополнительного аргумента функции).

### Заключение

В целом можно заключить, что массивы являются, по сути, просто удобным представлением,
своего рода надстройкой для облегчения работы с указателями и адресной арифметикой.
При этом, хотя любое продвинутое использование языка Си почти неизбежно подразумевает использование указателей,
в большинстве случаев, когда можно использовать массивы, их стоит препочесть использованию указателей на массив.
