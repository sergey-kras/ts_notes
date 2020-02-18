# Заметки по TS

## Intersection Types (пересеченные типы).

Если нужно задать какой либо сущьности свойства нескольких интерфейсов - то используем или пересечение интерфейсов, или наследование.

Пример пересечения:
```typescript
interface Animal {
    weight: number;
    height: number;
}

interface Dog {
    tailSize: number;
    makeGaf: () => {};
}

const bobic: Dog & Animal = {
    weight: 10,
    height: 1.6,
    tailSize: 1,
    makeGaf: () => {
        console.log('gaf');
    }
};
```

Пример наследования:
```typescript
interface Animal {
    ...
}

interface Dog extends Animal{
    ...
}

const bobic: Dog = {
    ...
};
```

## Union Types (объединение типов).

В случае, когда сущность может быть одним из нескольких типов интерфесов, исползуем Union Types.

```typescript
interface Animal {
    ...
}

interface Dog {
    ...
}

const bobic: Dog | Animal = {
    /*
        Тут будет либо
            weight: 10,
            height: 1.6,
        либо 
            tailSize: 1,
            makeGaf: () => console.log('gaf'),
    */
};
```

## Type Guards and Differentiating Types

Т.к. Мы можем получить доступ к тем методам/свойствам объекта, которые гарантированно у него есть, нужно как - то проверять наличие этих сущность. Делается это следующим образом.

```typescript
interface Animal {
    weight: number;
    height: number;
}

interface Dog {
    tailSize: number;
    makeGaf: () => {};
}

function foo(bar: Animal | Dog) {
    if((bar as Dog).makeGaf) {
        (bar as Dog).makeGaf();
    }
}
```

### User-Defined Type Guards (определяемые пользователем проверки)

В проверках выше мы "определяем" тип переменной прям в самой функции, где помимо определения типа есть еще какая - то логика. Выглядит как - то сильно громоздко, если у нас будет больше 3 свойств/методов в объекте.

#### Using type predicates (Использование предикаты типов)

В примере ниже предикат - это `pet is Animal`. Он говорит линтеру, каким типом является переменна `bar`. Более того в данной записи если `bar` не `Animal`, то `bar` - это `Dog` и обращаться с ним дальше нужно как с объектом типа `Dog`.

```typescript
function foo(bar: Animal | Dog) {
    if(isAnimal(bar)) {
        bar.weight;
    } else {
        bar.makeGaf();
    }
}

function isAnimal(pet: Animal | Dog): pet is Animal {
    return (pet as Animal).weight !== undefined;
}
```

#### Using the in operator (Использование in)

Одним из способов определить тип объекта является оператор in. Используется он следующим способом.

```typescript
function isAnimal(pet: Animal | Dog): pet is Animal {
    return 'weight' in pet; //true or false
}
```

### ==typeof== type guards (==typeof== в защите типов)

==typeof== Возвращает только примитивные типы. И работает только в случае, если у нас объединение типов number | boolean, boolean | string и т д.

Применяется вот так:

```typescript
function some(foo: number | string): number {
    let result: number;
    
    if(typeof foo === 'string') {
        result = Number(foo.replace(/\D+/g,""));
    } else {
        result = foo;
    }
    
    return result;
}
```

### instanceof type guards (Защита типов через instanceof)
instanceof применяется, когда нам ужно узнать, к какому классу принадлежит объект. instanceof использует функцию конструктора.

```typescript
class Bobic implements Dog {
    constructor(public tailSize: number){} 
    //Краткая запись this.tailSize = tailSize

    makeGaf() {
        console.log('gaf');
    }
}

function isBobic(pet: any): pet is Bobic {
    return pet instanceof Bobic;
}
```

## Nullable types

### Optional Chaining (Необязательные цепочки в TypeScript 3.7)
Бывает, что возникают ошибки, когда мы пытаемся получить доступ к свойству/методу, по пути через необязательное свойство и можем наткнуться на ошибку.

Например:

```typescript
interface Some {
    foo?: {
        bar: () => void;
    }
}

function(some: Some) {
    const doIt = some.foo.bar; // Вот тут будет ошибка, т.к foo может и не существовать
}
```

Простое решение. В ts версии 3.7 появились необязательные цепочки, которые можно использовать вместо цепочки условий в if.

```typescript
interface Some {
    foo?: {
        bar: () => void;
    }
}

function(some: Some) {
    const doIt = some.foo?.bar; // если foo все таки не существует - то в doIt запишется просто undefined
}
```

## Type Aliases

Можно создавать свои типы. Это нужно для объединения примитивов, создания объединений или пересечений типов.

Возьмем готовый пример выше:

```typescript
function isAnimal(pet: Animal | Dog): pet is Animal {
    return 'weight' in pet;
}
```

Для типа аргумента функции можно задать алиас и переиспользовать его по всему проекту.

```typescript
type Pet = Animal | Dog;

function isAnimal(pet: Pet): pet is Animal {
    return 'weight' in pet;
}
```

Более того можно сделать тип, который будет ссылаться сам на себя. Нужно, когда у нас сложные вложенные объекты с одинаковыми вложенными свойствами.

```typescript
type Pet = {
    value: Pet & Animal
};

function isAndimal(pet: Pet) {
    console.log(pet.value.value.value.weight);
}
```

### Interfaces vs. Type Aliases (Отличия интерфейсов от Алиасов)

Отличаются тем, что когда мы использует псевдоним типа - в IDE он будет показывать литерал объекта, а интерфейс покажет только имя интерфейса. При ошибках имя алиаса типа так же опказываться не будет.

## String Literal Types (Строковые литералы типов)

Строковые литералы гарантируют, что в переменной будет храниться одна из перечисленных в типе строк.

```typescript
type DogName = 'Bobic' | 'Juchka' | 'Barbos' | 'Kashtanka';

if (dog: DogName === 'Bobic') {
    // do something
}
```

Если в условии будет имя не из литерала - то ts ругнется, т.к. вы типе гарантировано должно быть что - то из списка строк.


## Полезная фигня, которую я нарыл
- Автоген типов https://github.com/Microsoft/dts-gen