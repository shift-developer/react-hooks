[Volver a inicio](../../README.md)
---

# 🔗 useRef

Tiene dos usos:
- Interactuar con el DOM
- Ser una referencia mutable (una variable que mantiene su estado entre renderizaciones y que al cambiar no genera un nuevo render)

useRef devuelve un objeto ref mutable cuya propiedad .current se inicializa con el argumento pasado (initialValue). El objeto devuelto **se mantendrá persistente durante la vida completa del componente.**

- Ten en cuenta que useRef no notifica cuando su contenido cambia. Mutar la propiedad .current no causa otro renderizado. 

Un caso de uso común es para acceder a un hijo imperativamente:
```javascript
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` apunta al elemento de entrada de texto montado
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

El otro caso común es como referencia mutable en peticiones asíncronas
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

---

Si quieres correr algún código cuando React agregue o quite una referencia de un nodo del DOM, puede que quieras utilizar en su lugar una referencia mediante callback. Puedes ver más acerca de useCallback presionando [aqui](../06-useCallback-useMemo/useCallback-useMemo.md).

```javascript
function MeasureExample() {
  const [height, setHeight] = useState(0);

  const measuredRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}


// Custom Hooks reutilizable
function MeasureExample() {
  const [rect, ref] = useClientRect();
  return (
    <>
      <h1 ref={ref}>Hello, world</h1>
      {rect !== null &&
        <h2>The above header is {Math.round(rect.height)}px tall</h2>
      }
    </>
  );
}

function useClientRect() {
  const [rect, setRect] = useState(null);
  const ref = useCallback(node => {
    if (node !== null) {
      setRect(node.getBoundingClientRect());
    }
  }, []);
  return [rect, ref];
}
```

# 🔻 useImperativeHandle

`useImperativeHandle` personaliza el valor de instancia que se expone a los componentes padres cuando se usa `ref`. Como siempre, el código imperativo que usa refs debe evitarse en la mayoría de los casos. useImperativeHandle debe usarse con `forwardRef`

## React.forwardRef
Crea un componente React que envía el atributo `ref` que recibe a otro componente más abajo en el árbol. Esta técnica no es muy común, pero es particularmente útil en dos escenarios:

- Enviar refs a componentes DOM
- Enviar refs en componentes de orden superior

`React.forwardRef` acepta una función de renderizado como un argumento. React llamará esta función con props y ref como dos argumentos. Esta función debe retornar un nodo React.

```javascript
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// Ahora puedes obtener un ref directamente al botón del DOM
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```
---

### Ejemplo

```javascript
// <Counter/>
import React, { useState, forwardRef, useImperativeHandle } from 'react'

export const Counter = forwardRef((props, ref) => {
  const [counter, setCounter] = useState(0);
  const increment = () => setCount(count+1);
  useImperativeHandle(ref, () => ({
    increment
  }));

  return(
    <div>
      <button onClick={increment}>
        click
      </button>
      <h2>Count: {count}</h2>
    </div>
  );
})

// <Example/>

import React, { useRef } from 'react'
import { Counter } from './Counter'

export const Example = () => {
  const ref = useRef();
  return(
    <>
      <Counter ref={ref}/>
      <button onClick={() => ref.current.increment()}>
        Another button
      </button>
    </>
  )
}
```


---
[Ir a => 🔂 useLayoutEffect](../05-useLayoutEffect/useLayoutEffect.md)