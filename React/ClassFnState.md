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

