# Чем функциональные компоненты React отличаются от компонентов, основанных на классах?

# Давайте напишем вот такое приложение 

![Анимация](https://user-images.githubusercontent.com/16369478/121025279-aa939380-c7ad-11eb-9221-f4fca023710e.gif)

# Классовый компонент

```
import React from 'react';

class ProfilePage extends React.Component {
    showMessage = () => {
        alert('Followed ' + this.props.user);
    };

    handleClick = () => {
        setTimeout(this.showMessage, 3000);
    };

    render() {
        return <button onClick={this.handleClick}>Follow</button>;
    }
}

export default ProfilePage;


```

# Компонент функция

```
import React from 'react';

function ProfilePage(props) {
    const showMessage = () => {
        alert('Followed ' + props.user);
    };

    const handleClick = () => {
        setTimeout(showMessage, 3000);
    };

    return (
        <button onClick={handleClick}>Follow</button>
    );
}

export default ProfilePage;

```

# В чем же разница?

Ошибка, которая часто встречается в React-приложениях

- Щёлкните по кнопке.
- Измените выбранный профиль до того, как после нажатия на кнопку прошли 3 секунды.
- Прочтите текст, выведенный в окне сообщения.

Сделав это, вы заметите следующие особенности:

1. При щелчке по кнопке, сформированной функциональным компонентом, при выбранном профиле Dan и при последующем переключении на профиль Sophie, в окне сообщения будет выведено 'Followed Dan'.

2. Если сделать то же самое с кнопкой, сформированной компонентом, основанном на классе, будет выведено 'Followed Sophie'.

![386a449110202d5140d67336a0ade5a0](https://user-images.githubusercontent.com/16369478/121025687-1a098300-c7ae-11eb-9219-ae7694e9e877.gif)

# Причины неправильного поведения компонента, основанного на классе

Если наш компонент выполняет повторный рендеринг во время выполнения запроса, this.props изменится. После этого метод showMessage прочтёт значение user из «слишком новой» сущности props.

Пропсы в React иммутабельны, поэтому они не меняются. Однако this, является мутабельной сущностью.

Мы выполняем чтение данных из this.props слишком поздно

# Как решить эту проблему в классе?

Один из способов сделать это заключается в том, чтобы заблаговременно прочитать this.props в обработчике события

```
handleClick = () => {
    const {user} = this.props;
    setTimeout(() => this.showMessage(user), 3000);
};

```

# Почему это работает в компоненте функции?

Обработчик события, который уже вызван, «принадлежит» предыдущему вызову этой функции, в этом вызове используется его собственное значение user и его собственный коллбэк showMessage, который это значение читает. Всё это остаётся нетронутым.

Теперь мы выяснили — в чём заключается большое различие между функциями и классами в React. Как уже было сказано, речь идёт о том, что функциональные компоненты захватывают значения.

# Жизненный цикл

У нас есть 3 этапа ЖЦ

- Монтирование
- Обновление
- Размонтирование

# Этап монтирования

При создании экземпляра компонента и его вставке в DOM, следующие методы вызываются в установленном порядке:

- constructor()
- static getDerivedStateFromProps(props, state)
- render()
- componentDidMount()

static getDerivedStateFromProps(props, state) - используется когда у вас есть зависимоть состояние компонента от его пропсов.

# Почему нельзя писать пропсы напрямую в стейт?

При изменении этих пропсов у вас не будет перерендера компонента

# Пример

В пропсах мы получаем значение для count

```
class Component extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0,
            error: ''
        }
    }

    static getDerivedStateFromProps(props) {
        if (props.count < 5) { // Проверям 
            return { // Возвращаем новое состояние для компонента
                count: props.count
            }
        } else {
            return { // Возвращаем новое состояние для компонента
                error: 'This is the end'
            }
        }
    }

    render() {
        return (
            <>
                { this.state.count }
            </>
        )
    }
}

```

# Обновление

- static getDerivedStateFromProps()
- shouldComponentUpdate(nextProps, nextState)
- render()
- getSnapshotBeforeUpdate(prevProps, prevState)
- componentDidUpdate(prevProps, prevState, snapshot)

# shouldComponentUpdate - метод, которіе определяем надо ли нашему компоненту перерендериваться
Должен вернуть или true (перерендериваемся), или false (не перерендериваемся)

```
class Component extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0,
            error: ''
        }
    }

    static getDerivedStateFromProps(props) {
        if (props.count < 5) {
            return {
                count: props.count
            }
        } else {
            return {
                error: 'This is end'
            }
        }
    }

    shouldComponentUpdate(nextProps, nextState) {
        if (this.state.error) { // если есть ошибка, то переренд уже не нужен
            return false
        }
        return true
    }

    render() {
        console.log('error: ', this.state.error)
        return (
            <>
                { this.state.count }
            </>
        )
    }
}

```

- getSnapshotBeforeUpdate() - вызывается прямо перед этапом «фиксирования» (например, перед добавлением в DOM). Он позволяет вашему компоненту брать некоторую информацию из DOM (например, положение прокрутки) перед её возможным изменением.

Может быть полезно в таких интерфейсах, как цепочка сообщений в чатах, в которых позиция прокрутки обрабатывается особым образом.


- componentDidUpdate(prevProps, prevState, snapshot)

вызывается сразу после обновления. Не вызывается при первом рендере.

В этом методе часто делаются перезапросы на сервер с новыми данными, которые мы получили в результате обновления

```
const myFetch = () => {
    setTimeout(() => {
        console.log('New data from fetch')
    }, 2000)
}

class LifeCycle extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0,
            error: ''
        }
    }

    componentDidUpdate(prevProps, prevState) {
        if (prevProps.count !== this.state.count) { // Если данные поменялись, то делаем запрос
            myFetch();
        } else {
            console.log('Мы не делаем перезапрос')
        }
    }

    render() {
        return (
            <>
                { this.state.count }
            </>
        )
    }
}

```

# Размонтирование
Этот метод вызывается при удалении компонента из DOM:

- componentWillUnmount() 


# Хуки

Хуки — нововведение в React 16.8, которое позволяет использовать состояние и другие возможности React без написания классов.


Какие проблемы в классах?

- Трудно повторно использовать логику состояний между компонентами (раньше использовали HOC, render prop и т.д.)
- Сложные компоненты становятся трудными для понимания


# Правила хуков

- Хуки следует вызывать только на верхнем уровне. Не вызывайте хуки внутри циклов, условий или вложенных функций.

- Хуки следует вызывать только из функциональных компонентов React. Не вызывайте хуки из обычных JavaScript-функций


# Краткрий обзор хуков

- Хук состояния

useState

принимает 1 аргумент, который является initialState

```
import React, { useState } from 'react';

export const Component = (props) => {
    const [count, setCount] = useState(0); // 0 - initialState, count - и есть ваше состояние, setCount - функция, которая будет менять ваше состояние

    const handleClick = () => {
        setCount(prevCount => prevCount + 1)
    }

    return (
        <>
            <button onClick={handleClick}>Click me</button>
            <span>{count}</span>
        </>
    )
}

```
useState может принимать все что угодно

```
const [...] = useState('hi')
const [...] = useState([{ name: 'Test' }])

```

#  Хук эффекта

С помощью хука эффекта useEffect вы можете выполнять побочные эффекты из функционального компонента. 

Он выполняет ту же роль, что и componentDidMount, componentDidUpdate и componentWillUnmount в React-классах

```
export const Component = (props) => {
    const [count, setCount] = useState(0);

    const handleClick = () => {
        setCount(prevCount => prevCount + 1)
    }

    useEffect(() => {
        //componentDidMount and componentDidUpdate
        console.log('Стработает при первом рендере')
        return () => {
            // componentWillUnmount
            console.log('Стработает когда компонент удалится')
        }
    })

    return (
        <>
            <button onClick={handleClick}>Click me</button>
            <span>{count}</span>
        </>
    )
}

```

useEffect так же принимает в себя второй аргумент - массив зависимостей

```
import React, {useEffect, useState} from 'react';

export const Component = (props) => {
    const [count, setCount] = useState(0);

    const handleClick = () => {
        setCount(prevCount => prevCount + 1)
    }

    useEffect(() => {
        console.log('Стработает при первом рендере')
        console.log('Стработает при изменении count')
    }, [count])

    return (
        <>
            <button onClick={handleClick}>Click me</button>
            <span>{count}</span>
        </>
    )
}

```

# useCallback

Хук useCallback вернёт мемоизированную версию колбэка, который изменяется только, если изменяются значения одной из зависимостей

Используется для отпимизации

Но так ли это?

На самом деле не всегда

# В чем проблема

```
const fn1 = useCallback(() => {
    console.log('count: ', count)
}, [count])
    
// VS
    
const fn2 = () => {
    console.log('count: ', count)
}

```

Функция useCallback как и любая другая функция вызывается на каждый рендер и в качестве параметра callback каждый рендер приходит новая функция и новый массив зависимостей.


# Когда же использовать этот хук??

Получается основная идея не в улучшении перформанса в конкретном компоненте, а скорее использование useCallback выгодно только в случае передачи функции как props.

Допустим у нас есть массив машин

```
const Cars = ({ cars }) => {
  return cars.map((car) => {
    return (
      <Car key={car.id} car={car} />
    )
  });
}

```

И нам надо добавить обработчик клика на машину

```
const Cars = ({ cars }) => {
  const onCarClick = (car) => {
    console.log(car.model);
  }
  
  return cars.map((car) => {
    return (
      <Car key={car.id} car={car} onCarClick={onCarClick} />
    )
  });
}

```

В такой ситуации на каждый рендер компонента Cars у нас создается новая функция onCarClick

Для этого и нужен useCallback, если мы завернем функцию в хук, то у нас в переменной onCarClick будет уже возвращаться мемоизированая функция, хоть мы в нее на каждый рендер и передаем новую функцию

```
const Cars = ({ cars }) => {
  const onCarClick = useCallback((car) => {
    console.log(car.model);
  }, []);
  
  return cars.map((car) => {
    return (
      <Car key={car.id} car={car} onCarClick={onCarClick} />
    )
  });
}

```

Таким образом все компоненты Car не будут рендериться лишний раз, т.к. ссылка на функцию останется прежней.

А если заглянуть внутрь компонента Car. Там мы создадим еще одну функцию, которая свяжет onCarClick и объект car.

```
const Car = ({ car, onCarClick }) => {
  const onClick = () => onCarClick(car);
  
  return (
    <button onClick={onClick}>{car.model}</button>
  )
}

```
