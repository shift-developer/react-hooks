[Volver a inicio](../../README.md)
---

# 🛠️ customHooks

Cuando queremos compartir lógica entre dos funciones de JavaScript, lo extraemos en una tercera función. Un Hook personalizado es una función de JavaScript cuyo nombre comienza con ”use” y que puede llamar a otros Hooks. Por ejemplo, a continuación useFriendStatus:

```javascript
// useFriendStatus (custom hook)
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}

function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}

// <FriendListItem/> component
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}

```
---
# Ejemplos

## useCounter
```javascript
// useCounter
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
import { useCounter } from '../hooks/useCounter';

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

### Pruebas sobre useCounter
```javascript
import { renderHook, act } from '@testing-library/react-hooks';
import { useCounter } from '../components/hooks/useCounter';

describe('Pruebas en useCounter', () => {

    test('debe de retornar valores por defecto', () => {
        const { result } = renderHook( () => useCounter() );
        expect(result.current.counter).toBe(10);
        expect(typeof result.current.increment).toBe('function');
        expect(typeof result.current.decrement).toBe('function');
        expect(typeof result.current.reset).toBe('function');
    }) 

    test('debe de tener el counter en 100', () => {
        const { result } = renderHook( () => useCounter(100) );
        expect(result.current.counter).toBe(100);
        
    }) 

    test('debe de incrementar el counter en 1 ', () => {
        const {result} = renderHook( () => useCounter(100));
        const { increment } = result.current;

        act( () => {
            increment();
        });

        const { counter } = result.current;
        expect(counter).toBe(101);
    })

    test('debe decrementar el counter en 1', () => {
        const {result} = renderHook( () => useCounter(200));
        const {decrement} = result.current;

        act( () => {
            decrement();
        })

        const {counter} = result.current;
        expect(counter).toBe(199);
    })

    test('debe de reiniciar el counter al valor inicial', () => {
        const initialValue = 1000;
        const {result} = renderHook( () => useCounter(initialValue));
        const {reset, decrement} = result.current;

        act( () => {
            decrement();
            decrement();
            reset()
        })

        const {counter} = result.current;
        expect(counter).toBe(initialValue);
    })
})
```

---

## useForm

```javascript
// useForm
import { useState } from "react"

export const useForm = ( initialState = {} ) => {
    
    const [values, setValues] = useState(initialState);

    const reset = () => {
        setValues( initialState );
    }

    const handleInputChange = ({ target }) => {

        setValues({
            ...values,
            [ target.name ]: target.value
        });

    }

    return [ values, handleInputChange, reset ];
}


// <FormWithCustomHook/>
import React, { useEffect } from 'react';
import { useForm } from '../hooks/useForm';

export const FormWithCustomHook = () => {

    const [ formValues, handleInputChange ] = useForm({
        name: '',
        email: '',
        password: ''
    });

    const { name, email, password } = formValues;

    return (
        <form>
            <h1>FormWithCustomHook</h1>
            <hr />

            <div>
                <input 
                    type="text"
                    name="name"
                    placeholder="Tu nombre"
                    autoComplete="off"
                    value={ name }
                    onChange={ handleInputChange }
                />
            </div>


            <div className="form-group">
                <input 
                    type="text"
                    name="email"
                    placeholder="email@gmail.com"
                    autoComplete="off"
                    value={ email }
                    onChange={ handleInputChange }
                />
            </div>

            <div className="form-group">
                <input 
                    type="password"
                    name="password"
                    placeholder="*****"
                    value={ password }
                    onChange={ handleInputChange }
                />
            </div>


            <button type="submit">
                Guardar
            </button>

        </form>
    )
}
```

### Pruebas sobre useForm
```javascript
import { renderHook, act } from '@testing-library/react-hooks';
import {useForm} from '../components/hooks/useForm'

describe('Pruebas en useForm', () => {
    const initialForm = {
        name: "Juan",
        email: "juan@gmail.com"
    }

    test('debe regresar un formulario por defecto', () => {
        const {result} = renderHook(() => useForm(initialForm))
        const [formValues, handleInputChange, reset] = result.current;
        expect(formValues).toEqual(initialForm)
        expect(typeof handleInputChange).toBe('function')
        expect(typeof reset).toBe('function')
    });

    test('debe de cambiar el valor del formulario (cambiar name)', () => {
        const {result} = renderHook(() => useForm(initialForm))
        const [, handleInputChange] = result.current;
        act( () => {
            handleInputChange({
                target: {
                    name: 'name',
                    value: 'Joe'
                }
            })
        })
        const [formValues] = result.current;
        expect(formValues).toEqual({...initialForm, name: 'Joe'})
    });

    test('debe de restablecer el formulario con RESET', () => {
        const {result} = renderHook(() => useForm(initialForm))
        const [, handleInputChange, reset] = result.current;
        act( () => {
            handleInputChange({
                target: {
                    name: 'name',
                    value: 'Joe'
                }
            })
            reset();
        })
        const [formValues] = result.current;
        expect(formValues).toEqual(initialForm)

    })
})

```

---

## useFetch
```javascript
import { useState, useEffect, useRef } from 'react';

export const useFetch = ( url ) => {
    
    const isMounted = useRef(true);
    const [state, setState] = useState({ data: null, loading: true, error: null });

    useEffect( () => {
        return () => {
            isMounted.current = false;
        }
    }, [])


    useEffect( () => {

        setState({ data: null, loading: true, error: null });

        fetch( url )
            .then( resp => resp.json() )
            .then( data => {

                if ( isMounted.current ) {
                    setState({
                        loading: false,
                        error: null,
                        data
                    });
                }

            })
            .catch( () => {
                setState({
                    data: null,
                    loading: false,
                    error: 'No se pudo cargar la info'
                })
            })

    },[url])

    return state;
}

```

### Pruebas sobre useFetch
```javascript
import { renderHook } from "@testing-library/react-hooks"
import { useFetch } from "../components/hooks/useFetch"

describe('Pruebas en useFetch', () => {
    test('debe de retornar la info por defecto', () => {
        const {result} = renderHook( () => useFetch('https://www.breakingbadapi.com/api/quotes/1'));

        const {data, loading, error} = result.current;
        expect(data).toBe(null);
        expect(loading).toBe(true);
        expect(error).toBe(null)
    })

    test('debe de tener la info deseada, loading false, error false', async () => {
        const {result, waitForNextUpdate} = renderHook( () => useFetch('https://www.breakingbadapi.com/api/quotes/1'));
        await waitForNextUpdate({timeout: 5000});

        const {data, loading, error} = result.current;
       
        expect(data.length).toBe(1)
        expect(loading).toBe(false)
        expect(error).toBe(null)
    })

    test('debe de manejar el error', async () => {
        const {result, waitForNextUpdate} = renderHook( () => useFetch('https://www.reqres.in/apid/users?page=2'));
        await waitForNextUpdate({timeout: 5000});

        const {data, loading, error} = result.current;
       
        expect(data).toBe(null)
        expect(loading).toBe(false)
        expect(error).toBe('No se pudo cargar la info')
    })
    
    
})

```

---

## Multiple Custom Hooks
```javascript
import React from 'react'
import { useFetch } from '../hooks/useFetch'
import { useCounter } from '../hooks/useCounter'

export const MultipleCustomHooks = () => {

    const { counter, increment } =  useCounter(1);
    const { loading, data } = useFetch( `https://www.breakingbadapi.com/api/quotes/${ counter }` );
    const { author, quote } = !!data && data[0];

    return (
        <div>
            <h1>BreakingBad Quotes</h1>
            <hr />

            {
                loading 
                ?
                    (
                        <div className="alert alert-info text-center">
                            Loading...
                        </div>
                    )
                :
                    (
                        <blockquote className="blockquote text-right">
                            <p className="mb-0"> { quote } </p>
                            <footer className="blockquote-footer"> { author } </footer>
                        </blockquote>
                    )
            }


            <button 
                className="btn btn-primary"
                onClick={ increment }
            >
                Siguiente quote
            </button>

        </div>
    )
}

```

### Pruebas sobre Multiple Custom Hooks
```javascript
import React from 'react'
import { shallow } from 'enzyme'
import { MultipleCustomHooks } from '../components/MultipleCustomHooks'
import { useFetch } from '../components/hooks/useFetch'
import { useCounter } from '../components/hooks/useCounter'
jest.mock('../components/hooks/useFetch')
jest.mock('../components/hooks/useCounter')

describe('Pruebas en <MultipleCustomHooks />', () => {

    beforeEach( () => {
        useCounter.mockReturnValue({
            counter: 10,
            increment: () => {}
        });
    })
    
    test('debe de mostrarse correctamente', () => {

        useFetch.mockReturnValue({
            data: null,
            loading: true,
            error: null
        })
        
        const wrapper = shallow(<MultipleCustomHooks />);
        expect(wrapper).toMatchSnapshot();
    })

    test('debe de mostrar la informacion', () => {
        
        useFetch.mockReturnValue({
            data: [{
                author: "Lenny",
                quote: "Hello World"
            }],
            loading: false,
            error: null
        });

        const wrapper = shallow(<MultipleCustomHooks />);
        expect(wrapper.find('.alert').exists()).toBe(false)
        expect(wrapper.find('.mb-0').text().trim()).toBe('Hello World')
        expect(wrapper.find('footer').text().trim()).toBe('Lenny')
    })
    
    
})

```

---
[ir a => 🔗 useRef](../04-useRef/useRef.md)




