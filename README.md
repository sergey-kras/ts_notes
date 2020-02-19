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

interface Dog extends Animal {
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

Т.к. Мы можем получить доступ к тем методам/свойствам объекта, которые гарантированно у него есть, нужно как - то проверять наличие этих сущностей. Делается это следующим образом.

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

Простое решение. В ts версии 3.7 появились необязательные цепочки, которые можно использовать вместо цепочки условий в if-ах.

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

## Numeric Literal Types (Числовые литеральные типы)

1 в 1, как и строковые, поэтому только пример, чтобы знать, что такое есть.

```typescript
type DogCount = 0 | 1 | 1 | 2 | 3 | 5 | 8;

if (dog: DogCount === 'Bobic') {
    // do something
}
```

## Enum Member Types (Перечисления типов)

Используются, когда нужно при помощи констант задать набор каких либо значений. Например полезно, когда нам нужно распарсить объект и мы заранее знаем его ключи. 

Пример:

```typescript
enum DogName {
    Bobic = 'Bobic',
    Juchka = 'Juchka',
    Barbos = 'Barbos',
    Kashtanka = 'Kashtanka'
}

function foo(dog: keyof typeof DogName) {
    if (dog === 'Bobic') {
        // do something
    }
}
```

Аналогично с числовыми. Но по-умолчанию значения ключей инкрементируются автоматически.

```typescript
enum LogLevel {
    Error = 0,
    Warning, // 1 
    Info, // 2
}

function makeLog(logLevel: LogLevel) {
    if (logLevel === LogLevel.Error) {
        // do something
    }
}
```

## Discriminated Unions

Выше в примерах уже встречалось. По сути это просто тип с логическим оператором "или", который говорит, что сущность является одним из какого - то набора типов.

```typescript
type Pet = Animal | Dog;

function isAnimal(pet: Pet): pet is Animal {
    return 'weight' in pet;
}
```

## Index types (Индексные типы)

Полезны, когда нужно протипизировать сущности, имена которых задаются динамически.

```typescript
function pluck<T, K extends keyof T>(o: T, propertyNames: K[]): T[K][] {
    return propertyNames.map(n => o[n]);
}

interface Car {
    manufacturer: string;
    model: string;
    year: number;
}
let taxi: Car = {
    manufacturer: 'Toyota',
    model: 'Camry',
    year: 2014
};

let makeAndModel: string[] = pluck(taxi, ['manufacturer', 'model']);
```

Что круто. Теперб в функцию pick мы можем передавать только определенные строки в массиве, которые зависят от передаваемого первым аргументом объекта. Минус непредсказуемое поведение.

## Mapped types (Сопоставленные типы)

Стоит использовать, когда нам нужно взять какой либо интерфейс и немного его поменять. Например сделать все поля необязательными или приватными / публичными / защищенными и т д. Либо у всех полей сделать другой тип. Выглядит все это следующим образом.

```typescript
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}

type Partial<T> = {
    [P in keyof T]?: T[P];
}
```

Так же данный механизм может помочь создавать свои типы.

```typescript
type PipelineStages = 'build' | 'pushArtifacts' | 'unitTests' | 'integrationTests' | 'uiTests';

type PipelineStatus = {
    [K in PipelineStages]: boolean
}

const pipeline: PipelineStatus = {
    build: true,
    pushArtifacts: true,
    unitTests: false,
    integrationTests: true,
    uiTests: true
};
```

В итоге переменная `pipeline` типа `PipelineStatus` должна содержать только те свойства, которые строчно описаны в `PipelineStages` и должны быть либо `true`, либо `false`.

Так, а если мы в другой части приложения хотим, чтобы pipeline хранил какую то текстовую информацию вместе true и false и хотим добавить статус деплоя?

Делаем следующее:
```typescript
type PipelineStages = 'build' | 'pushArtifacts' | 'unitTests' | 'integrationTests' | 'uiTests';

type PipelineStatus = {
    [K in PipelineStages]: boolean
}

type PipelineStatusExtra = {
    [K in keyof PipelineStatus]: string
} & { deploy: string }

const pipeline: PipelineStatusExtra = {
    build: 'string',
    pushArtifacts: 'string',
    unitTests: 'string',
    integrationTests: 'string',
    uiTests: 'string',
    deploy: 'string'
};
```

Теперь в `PipelineStatusExtra` все свойства должны быть строками и есть статус деплоя.

Таким образом можно собирать свои собственные типы, не трогая уже существующие.

## Conditional Types (Условные типы)

Conditional Types умеет определять тип сущности по какому либо условию.

```typescript
enum CatNames {
    Barsik = 'Barsik',
    Murka = 'Murka'
}

enum DogNames {
    Juchka = 'Juchka',
    Barbos = 'Barbos'
}

interface Cat {
    name: keyof typeof CatNames,
    makeMau: () => void
}

interface Dog {
    name: keyof typeof DogNames,
    makeGaf: () => void
}

class MyDog implements Dog {
    constructor(public name: keyof typeof DogNames) {}

    makeGaf() {
        console.log('Gaf');
    }
}

class MyCat implements Cat {
    constructor(public name: keyof typeof CatNames) {}

    makeMau() {
        console.log('Mau');
    }
}

declare function getPetName
    <T extends Dog | Cat>
    (pet: T): T extends Cat ? keyof typeof CatNames : keyof typeof DogNames

getPetName(new MyCat('Barsik'));
getPetName(new MyDog('Juchka'));
```

Наваял код выше, теперь с этим нужно как-то разобраться.

Что есть.
- Перчисление имен собак
- Перчисление имен котов
- Интерфейс котов
- Интерфейс собак
- Отельный кот, принадлежащий интерфейсу котов
- Отельная собака, принадлежащая интерфейсу собак
- Функция, которая возвращает имя петомца.

Самое интересное тут - это функция, которая может частично предсказать, что она вернет нам в зависимости от объекта, который мы ей передаем. Она вернет нам одно из множества имен котов, если мы передадим ей кота, либо одно из множества имен собак, если мы передадим ей собаку.

Еще интересно. Если мы захотим создать собаку с именем кота - то TS нам не даст это сделать. Т.к. в конструкторе мы гарантируем, что собака будет создана только с именем собаки. Аналогично и с котами.

### Distributive conditional types (Распределительные условные типы)

По сути просто перемешивание объединения типов с условными типами.

```typescript
type ArrayFilter<T> = T extends any[] ? T : never;

type StringsOrNumbers = ArrayFilter<string | number | string[] | number[]>;
```

В итоге в `StringsOrNumbers` будет либо массив строк, либо массив чисел. Т.к для всех, что не является массивом, переденных в generic, будет never.


## Полезная фигня, которую я нарыл
- Автоген типов https://github.com/Microsoft/dts-gen

## Ресурсы
- Оф.документация по TS https://www.typescriptlang.org/docs/handbook/advanced-types.html
- Про TS 3.7 https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#optional-chaining
- Про utility types https://www.typescriptlang.org/docs/handbook/utility-types.html
- Про Enum https://www.typescriptlang.org/docs/handbook/enums.html#union-enums-and-enum-member-types
- Нашел интересный сайт со статьями по фронту https://frontend-stuff.com/blog/conditional-types/