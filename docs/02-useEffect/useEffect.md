[Volver a inicio](../../README.md)

# 🔂 useEffect

## Efectos sin saneamiento

En ciertas ocasiones, queremos ejecutar código adicional después de que React haya actualizado el DOM. Peticiones de red, mutaciones manuales del DOM y registros, son ejemplos comunes de efectos que no requieren una acción de saneamiento. Decimos esto porque podemos ejecutarlos y olvidarnos de ellos inmediatamente. Vamos a comparar cómo las clases y los Hooks nos permiten expresar dichos efectos.

En las clases de React ponemos los efectos secundarios en `componentDidMount` y `componentDidUpdate`. Volviendo a nuestro ejemplo, aquí tenemos el componente clase contador de React que actualiza el título del documento justo después de que React haga cambios en el DOM:

### Class component

```javascript
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

### Functional Component

El Hook de efecto (useEffect) te permite llevar a cabo efectos secundarios en componentes funcionales:

```javascript
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // De forma similar a componentDidMount y componentDidUpdate
  useEffect(() => {
    // Actualiza el título del documento usando la API del navegador
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

#### ⚠️ A tener en cuenta sobre el hook:
- Al usar este Hook, le estamos indicando a React que el componente tiene que hacer algo después de renderizarse.
- Poner useEffect dentro del componente nos permite acceder a la variable de estado count (o a cualquier prop) directamente desde el efecto.
- Por defecto se ejecuta después del primer renderizado y después de cada actualización. 


## Efectos con saneamiento

Por ejemplo, si queremos establecer una suscripción a alguna fuente de datos externa. En ese caso, ¡es importante sanear el efecto para no introducir una fuga de memoria!

### Class component
Fíjate en como la lógica que asigna `document.title` se divide entre `componentDidMount` y `componentDidUpdate`. La lógica de la suscripción a ChatAPI también se reparte entre `componentDidMount` y `componentWillUnmount`. Y `componentDidMount` contiene código de ambas tareas.

```javascript
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate(prevProps) {
    document.title = `You clicked ${this.state.count} times`;

    // Cancela la suscripción del friend.id anterior
    ChatAPI.unsubscribeFromFriendStatus(
      prevProps.friend.id,
      this.handleStatusChange
    );
    // Se suscribe al siguiente friend.id
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
```

### Functional component

```javascript
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```

#### ⚠️ A tener en cuenta sobre el hook:
- Todos los efectos pueden devolver una función que los sanea más tarde. Esto nos permite mantener la lógica de añadir y eliminar suscripciones cerca la una de la otra.
- React sanea el efecto cuando el componente se desmonta y también sanea los efectos de renderizados anteriores antes de ejecutar los efectos del renderizado actual.

```javascript
// Se monta con las props { friend: { id: 100 } }
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // Ejecuta el primer efecto

// Se actualiza con las props { friend: { id: 200 } }
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // Sanea el efecto anterior
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // Ejecuta el siguiente efecto

// Se actualiza con las props { friend: { id: 300 } }
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // Sanea el efecto anterior
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // Ejecuta el siguiente efecto

// Se desmonta
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // Sanea el último efecto
```

## Omitir efectos para optimizar el rendimiento

En algunos casos, sanear o aplicar el efecto después de cada renderizado puede crear problemas de rendimiento. En los componentes de clase podemos solucionarlos escribiendo una comparación extra con `prevProps` o `prevState` dentro de `componentDidUpdate`:

### Class component
```javascript
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

### Functional component
```javascript
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Solo se vuelve a ejecutar si count cambia
```

```javascript
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // Solo se vuelve a suscribir si la propiedad props.friend.id cambia
```

**Para ejecutar un efecto y sanearlo solamente una vez (al montar y desmontar), puedes pasar un array vacío ([]) como segundo argumento.**


