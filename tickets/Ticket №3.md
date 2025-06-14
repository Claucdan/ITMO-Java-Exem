# Иерархия интерфейсов для работы с коллекциями. Особенности Stream API.
## Иерархия интерфейсов для работы с коллекциями.
### Краткая информация
![photo1](../photos/Pasted%20image%2020250609235553.png)

### Расширенная информация
![Pasted image 20250609235635.png](../photos/Pasted%20image%2020250609235635.png)
### Объяснение

#### Корневой интерфейс `Iterable<T>`
- Находится в вершине иерархии
- Определяет метод `iterator()` для получения итератора
- Добавлен в Java 5 для поддержки for-each цикла
### Интерфейс `Collection<E>` (расширяет `Iterable<E>`)
Базовый интерфейс для всех коллекций, основные методы:
- `add(E e)`, `addAll(Collection<? extends E> c)`
- `remove(Object o)`, `removeAll(Collection<?> c)`
- `contains(Object o)`, `containsAll(Collection<?> c)`
- `size()`, `isEmpty()`, `clear()`
- `toArray()`, `stream()`, `parallelStream()`
### Основные ветви иерархии
1. `List<E>` (упорядоченные коллекции)
	*Характеристики:*
	- Сохраняют порядок добавления элементов
	- Допускают дубликаты
	- Доступ по индексу
	*Основные реализации:*
	- `ArrayList` - на основе массива
	- `LinkedList` - на основе связанного списка
	- `Vector` (устаревшая, потокбезопасная версия ArrayList)
	- `Stack` (расширяет Vector)
2. `Set<E>` (множества)
	*Характеристики:*
	- Не допускают дубликаты
	- Не гарантируют порядок (кроме LinkedHashSet)
	*Основные реализации:*
	- `HashSet` - на основе хэш-таблицы
	- `LinkedHashSet` - сохраняет порядок добавления
	- `TreeSet` - отсортированное множество (на основе TreeMap)
3. `Queue<E>` (очереди)
	*Характеристики:*
	- FIFO (First-In-First-Out) или приоритетный порядок
	- Специальные методы для работы с головой очереди
## Особенности Stream API
> Его задача - упростить работу с наборами данных, в частности, упростить операции фильтрации, сортировки и другие манипуляции с данными.
> Ключевым понятием в Stream API является поток данных.
### Отличия от коллекций
- **Храенение данных**
	- не хранит данные
	- источние - коллекция, массив, I/O
- **Изменяемость**
	- неизменяемы (как String)
- **Порядок обработки**
	- Lazy 
	- Вычисления только при терминальной операции
- **Параллелизм**
	- Автоматическое распараллеливание
### 2. Основные особенности Stream API
 1. Функциональный стиль программирования
	- Используются **лямбда-выражения** и **методные ссылки**.
	- Нет side-эффектов (потоки не изменяют исходные данные).
  
2. Ленивые и терминальные операции
	- **Промежуточные (lazy) операции**:
	  - `filter()`, `map()`, `flatMap()`, `distinct()`, `sorted()`, `peek()`, `limit()`, `skip()`
	  - Не выполняются без терминальной операции.
	- **Терминальные операции**:
	  - `forEach()`, `collect()`, `reduce()`, `count()`, `min()`, `max()`, `anyMatch()`, `allMatch()`, `noneMatch()`, `findFirst()`, `findAny()`
	  - Запускают выполнение всего конвейера.
3. Потоки одноразовые
	- После вызова терминальной операции поток **закрывается**.
	- Повторное использование вызовет `IllegalStateException`.
4. Поддержка параллелизма
	- Простое распараллеливание через `.parallel()` или `parallelStream()`.
	- Внутренняя оптимизация (ForkJoinPool).
5. Примитивные специализации
	- `IntStream`, `LongStream`, `DoubleStream` — для работы с примитивами (избегают боксинга).
	- Методы: `mapToInt()`, `sum()`, `average()`, `range()`, `boxed()`.

### Преимущества Stream API
- **Удобство** — лаконичный код (меньше шаблонных конструкций).  
- **Производительность** — оптимизация под капотом (ленивые вычисления, параллелизм).  
- **Читаемость** — декларативный стиль (описание "что сделать", а не "как").  
- **Безопасность** — отсутствие side-эффектов (исходные данные не меняются).  