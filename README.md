#### [Typescript notes](ts.md)

---  
  
[Как мокать данные с сервера](#mockserver)  
[Ахитектура без длинных путей в импорте](#importsways)  
[fetch with useTransition](#usetransition)  
[props и children в пользовательский компонент](#propschildren)  
[почему хуки нельзя использовать в if и for](#hooksif)  
[Symbol](#symbol)
[javascript задачка f(x)](#jstask)
  
---  
[Access и refresh токены](tokens.md)  

---
  
<a id="mockserver"></a>
## Как мокать данные с сервера

_Шаг 1_: **создаем db.json**

```json

{
    "posts": [
      {
        "id": 1,
        "title": "Create a login form using formik in react js",
        "body": "Todays article will demonstrate how to develop a login form in react js using formik."
      },
      {
        "id": 2,
        "title": "How to parse or read CSV files in ReactJS",
        "body": "In this article, I will teach you how to parse or read CSV files in ReactJS in the simplest way possible. "
      }
    ]
  }
```
  
_Шаг 2_: **Устанавливаем json-server**  
  
```
npm install -g json-server
```
  
_Шаг 3_: **Стартуем JSON-сервер**  
  
```
json-server --watch db.json --port 3030
```  
Note: React utilizes the 3000 port, which json-server uses to run the server, thus we used — port 3030 to modify the port.
  
_Шаг 4:_ **Тестируем API**  
  
Тепер, если открыть http://localhost:3030/posts мы сможем увидеть data
![image](https://github.com/Highflyingexpress/frontend-tricks/assets/107925514/c6fc10a5-843e-4c2a-ab2b-b1cdc8a6b6a9)
_Шаг 5:_ **Делаем API запрос в React**   
например классический пример:  
```javascript
import React, { useState, useEffect } from "react";

const App = () => {
  const [posts, setPosts] = useState([]);

  const getData = () => {
    var requestOptions = {
      method: "GET",
      redirect: "follow",
    };

    fetch("http://localhost:3030/posts", requestOptions)
      .then((response) => response.json())
      .then((result) => setPosts(result))
      .catch((error) => console.log("error", error));
  };

  useEffect(() => {
    getData();
  }, []);

  return (
    <div>
      {posts.map((post) => (
        <div key={post.id}>
          <h3>
            <span>{post.id}</span> {post.title}
          </h3>
          <p>{post.body}</p>
        </div>
      ))}
    </div>
  );
};

export default App;
```

Доступны все операции  
**POST**  
![image](https://github.com/Highflyingexpress/frontend-tricks/assets/107925514/490cbe5b-92c5-498b-bb51-29dbb078d6ed)

**PUT**  
![image](https://github.com/Highflyingexpress/frontend-tricks/assets/107925514/c5b8d2b0-24e5-4792-b37d-aee1435f9240)  

**DELETE**  
![image](https://github.com/Highflyingexpress/frontend-tricks/assets/107925514/ab0a1cbb-0ea1-41eb-8c01-3ee93efa2257)
  
<a id="importsways"></a>
## Архитектура. Как не писать в импортах длинные пути к файлам.
  
**public-api.ts + index.ts**   
1) `index.ts` в каждой папке, который экспортирует все файлы из папки  
2) затем в `public-api.ts` собираем все экспорты из всех папок

Пример структуры проекта:
```
/src
  /moduleA
    - fileA1.ts
    - fileA2.ts
    - index.ts
  /moduleB
    - fileB1.ts
    - fileB2.ts
    - index.ts
  /moduleC
    - fileC1.ts
    - fileC2.ts
    - index.ts
  - public-api.ts
```
Пример содержимого файлов index.ts:
```
// moduleA/index.ts
export * from './fileA1';
export * from './fileA2';

// moduleB/index.ts
export * from './fileB1';
export * from './fileB2';

// moduleC/index.ts
export * from './fileC1';
export * from './fileC2';

// public-api.ts
export * from './moduleA';
export * from './moduleB';
export * from './moduleC';
```

`public-api.ts` собирает все экспорты в одном месте   
```
  // Другие файлы в проекте
import { someFunctionFromModuleA, someVariableFromModuleB } from './public-api';  
```
  
<a id="usetransition"></a>  
### Basic fetching data with useTransition  
today weather in Saint.P  
```typescript
import React, { useEffect, useState, useTransition } from "react";

interface Location {  country: string; }
interface Current {  temp_c: number; }
interface WeatherResponse {
  location: Location;
  current: Current;
}

const api = "b187fa2d8b7047e88f780241231411"; // api key

const App: React.FC = () => {
  const [data, setData] = useState<WeatherResponse | null>(null);
  const [isPending, startTransition] = useTransition(); // React 18' hook was born

  useEffect(() => {
    const fetchData = async (): Promise<void> => {
      try {
        await startTransition(async () => {
          const response = await fetch(
            `https://api.weatherapi.com/v1/current.json?key=${api}&q=Saint-Petersburg&aqi=no`
          );

          if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
          }

          const responseData: WeatherResponse = await response.json();
          setData(responseData);
        });
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    };

    fetchData();
  }, []); // empty dependency array for initial render

  return (
    <div>
      {isPending ? (
        <p>Loading...</p>
      ) : (
        data && (
          <>
            <h1>{data.location.country}</h1>
            <p>Temperature: {data.current.temp_c}°C</p>
          </>
        )
      )}
    </div>
  );
};
export default App;
```  
  
<a id="propschildren"></a>  
## props и children от родителя к чаилду  
  
```javascript
import React, { useState } from 'react';

const Counter = ({children, ...props}) => {
  const [count, setCount] = useState(0);
  const handleChange = () => {
    setCount((prev) => prev + 1);
  };

  return ( 
    <div {...props}>
      <span> {children} </span>
      <button onClick={handleChange}>Click me</button>
      <strong>{count}</strong>
    </div>
  );
};

const App = () => {
  return (
    <div className="App">
      <Counter style={{ color: 'red' }} />
      <Counter className="myclass" />
      <Counter>Counter:</Counter>
      <Counter />
    </div>
  );
};

export default App;
```

<a id="hooksif"></a> 
## Почему хуки не используют в условия и циклах  
  
   Правила использования хуков гарантируют стабильность порядка вызовов хуков в компоненте, что важно для корректной работы React и избежания потенциальных проблем.  
Concurrent Mode в React представляет собой новую функциональность, которая призвана улучшить производительность приложений, обеспечивая более гладкую работу приложения в условиях высокой нагрузки или медленного рендеринга. Одним из аспектов Concurrent Mode является возможность прерывания и возобновления процесса рендеринга, что позволяет React более гибко распределять свои ресурсы.    
Это правило обеспечивает стабильный порядок вызова хуков, что позволяет React корректно отслеживать состояние компонента и предотвращает возможные ошибки и неопределенности.  
Кратко говоря, **React использует порядок вызова хуков, чтобы правильно ассоциировать состояние и другие данные с конкретным вызовом хука в компоненте. Если хуки будут вызваны внутри условных операторов или циклов, порядок вызова может измениться в зависимости от логики выполнения, и это может привести к непредсказуемому поведению.**
  
## Symbol  
  
// читаем символ из глобального реестра и записываем его в переменную  
let id = `Symbol.for("id")`; // если символа не существует, он будет создан  
  
// читаем его снова и записываем в другую переменную (возможно, из другого места кода)  
let idAgain = `Symbol.for("id")`;  
  
// проверяем -- это один и тот же символ  
alert( id === idAgain ); // true  

 Object.assign, в отличие от цикла for..in, копирует и строковые, и символьные свойства  

  Технически символы скрыты не на 100%. Существует встроенный метод `Object.getOwnPropertySymbols(obj)` – с его помощью можно получить все свойства объекта с ключами-символами. Также существует метод `Reflect.ownKeys(obj)`, который возвращает все ключи объекта, включая символьные.

Если метод Symbol.toPrimitive существует, он используется для всех хинтов, и больше никаких методов не требуется.  
  
```javascript
let user = {
  name: "John",
  money: 1000,

  [Symbol.toPrimitive](hint) {
    alert(`hint: ${hint}`);
    return hint == "string" ? `{name: "${this.name}"}` : this.money;
  }
};

// демонстрация результатов преобразований:
alert(user); // hint: string -> {name: "John"}
alert(+user); // hint: number -> 1000
alert(user + 500); // hint: default -> 1500
```
  
<a id="jstask"></a> 
```
let x = {a: 1, b: 2};
function fn1(x) {
 x.a = 5;
}
function fn2() {
  x.a = 5;
}
function fn3(x) {
 x = 5;
}
function fn4() {
 x = 5;
}

fn1(x);
console.log(x);

fn2(); 
console.log(x);

fn3(x);
console.log(x);

fn4(); 
console.log(x);
```
