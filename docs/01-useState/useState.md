[Volver a inicio](../../README.md)

# 🔄 useState

El estado manejado en React con clases:

```javascript
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
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


Manejado con el hook de estado:

```javascript
import React, { useState } from 'react';

function Example() {
  // Declaración de una variable de estado que llamaremos "count"
  const [count, setCount] = useState(0);

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

## Counter App

```javascript
import React, { useState } from 'react';

export const CounterApp = () => {

    const [ state, setState] = useState({
        counter1: 10,
        counter2: 20,
        counter3: 30,
        counter4: 40,
    });

    const { counter1, counter2 } = state;


    return (
        <>
         <h1>Counter1 { counter1 } </h1>
         <h1>Counter2 { counter2 } </h1>
         <hr />

         <button 
            className="btn btn-primary"
            onClick={ () => {
                setState({
                    ...state,
                    counter1: counter1 + 1
                });
            }}
        >
             +1
         </button>
        </>
    )
}
```

### CounterApp (custom hook useCounter)
```javascript
// hooks/useCounter
import { useState } from 'react';

export const useCounter = ( initialState = 10 ) => {
    
    const [counter, setCounter] = useState(initialState);

    const reset = () => {
        setCounter( initialState );
    }

    const increment = () => {
        setCounter( counter + 1 );
    }

    const decrement = () => {
        setCounter( counter - 1 );
    }

    return {
        counter,
        increment,
        decrement,
        reset
    };
}


// <CounterWithCustomHook/>
import React from 'react';
import { useCounter } from '../../hooks/useCounter';

export const CounterWithCustomHook = () => {

    const { state, increment, decrement, reset } = useCouter( 100 );

    return (
        <>
          <h1>Counter with Hook: { state } </h1>
          <hr />

          <button onClick={ () => increment(2) } className="btn"> + 1</button>
          <button onClick={ reset } className="btn"> Reset </button>
          <button onClick={ () => decrement(2) } className="btn"> - 1</button>
        </>
    )
}
```


[Ir a => 🔂 useEffect](../02-useEffect/useEffect.md)