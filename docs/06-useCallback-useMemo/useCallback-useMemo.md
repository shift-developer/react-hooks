[Volver a inicio](../../README.md)
---

# 💾 useCallback-useMemo

## React.memo HOC

***Memoizar*** (y no memorizar) es un concepto informático o **técnica de optimización** para que dados los mismos argumentos de una función que ejecuta un "trabajo pesado" y el mismo resultado, guardemos este resultado y lo devolvamos evitando re-ejecutarla.

En React, si el componente renderiza el mismo resultado dadas las mismas props, se puede envolver en una llamada a `React.memo` para una mejora en el desempeño en algunos casos *memoizando* el resultado. Esto significa que React omitirá renderizar el componente y reusará el último resultado renderizado. 

`React.memo` solamente verifica los cambios en las props. Si tu componente de función envuelto en React.memo tiene un Hook `useState`, `useReducer` o `useContext` en su implementación, continuará volviéndose a renderizar cuando el estado o el contexto cambien.

Por defecto solo comparará superficialmente objetos complejos en el objeto de props. Si se desea controlar la comparación, se puede proporcionar también una función de comparación personalizada como el segundo argumento.

```javascript
function MyComponent(props) {
  /* renderiza usando props */
}
function areEqual(prevProps, nextProps) {
  /*
  retorna true si al pasar los nextProps a renderizar retorna
  el mismo resultado que al pasar los prevProps a renderizar,
  de otro modo retorna false
  */
}
export default React.memo(MyComponent, areEqual);
```

---
### ⚠️ A diferencia del método shouldComponentUpdate() en los componentes de clases, la función areEqual retorna true si los props son iguales y false si los props no son iguales. Esto es lo opuesto a shouldComponentUpdate.
---

### Ejemplo

```javascript
// <Small/>
import React from 'react'

export const Small = React.memo(({ value }) => {
    console.log(' Me volví a llamar :(  '); // solamente se va ejecutar si la prop cambia y no si el componente padre re-renderiza
    return (
        <small> { value } </small>
    )
});

// <Memorize/>
import React, { useState } from 'react';
import { useCounter } from '../hooks/useCounter';
import { Small } from './Small';

export const Memorize = () => {

    const { counter, increment } =  useCouter( 10 );
    const [ show, setShow ] = useState(true);

    return (
        <div>
            <h1>Counter: <Small value={ counter } />  </h1>
            <hr />


            <button 
                className="btn btn-primary"
                onClick={ increment }
            >
                +1
            </button>

            <button
                className="btn btn-outline-primary ml-3"
                onClick={ () => {
                    setShow( !show ); // si no usaramos React.memo en <Small/> al presionar este boton, se volveria a renderizar
                }}
            >
                Show/Hide { JSON.stringify( show ) }
            </button>

        </div>
    )
}

```

---

## useCallback

**Devuelve un callback memorizado.**

Pasa un callback en línea y un arreglo de dependencias. `useCallback` devolverá una versión memorizada del callback que solo cambia si una de las dependencias ha cambiado. Esto es útil cuando se transfieren callbacks a componentes hijos optimizados que dependen de la igualdad de referencia para evitar renders innecesarias (por ejemplo, `shouldComponentUpdate`).

`useCallback(fn, deps)` es igual a `useMemo(() => fn, deps)`.

```javascript
// <ShowIncrement/>
import React from 'react'

export const ShowIncrement = React.memo(({ increment }) => {

    console.log(' Me genero solo una vez :) '); 
    // al usar useCallback la funcion no cambia la referencia en memoria
    // junto con React.memo permite evitar renderizaciones innecesarias en este mismo componente ya que increment no cambia

    return (
        <button
            className="btn btn-primary"
            onClick={ () => {
                increment( 5 );
            }}
        >
            Incrementar
        </button>
    )
})

// <CallbackHook/>
import React, { useState, useCallback, useEffect } from 'react';
import { ShowIncrement } from './ShowIncrement';

export const CallbackHook = () => {

    const [counter, setCounter] = useState( 10 );

    const increment = useCallback( (num) => {
        setCounter( c => c + num );
    }, [ setCounter ] );

    
    useEffect( () => {
        // algun efecto que dependa del cambio en el increment
    }, [increment] )


    return (
        <div>
            <h1>useCallback Hook:  { counter }  </h1>
            <hr />

            <ShowIncrement increment={ increment } />

        </div>
    )
}

```

---

## useMemo

**Devuelve un valor memorizado.**

Pasa una función de “crear” y un arreglo de dependencias. `useMemo` solo volverá a calcular el valor memorizado cuando una de las dependencias haya cambiado. Esta optimización ayuda a evitar cálculos costosos en cada render. 

`const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);`

### Ejemplos
```javascript
// procesoPesado
export const procesoPesado = ( iteraciones ) => {

    for( let i = 0; i < iteraciones; i ++ ){
        console.log('Ahí vamos....');
    }

    return `${ iteraciones } iteraciones realizadas.`;
}

// <MemoHook/>
import React, { useState, useMemo } from 'react';
import { useCounter } from '../../hooks/useCounter';
import { procesoPesado } from '../../helpers/procesoPesado';

export const MemoHook = () => {

    const { counter, increment } =  useCounter( 50000 );
    const [ show, setShow ] = useState(true);
    
    const memoProcesoPesado = useMemo(() => procesoPesado(counter), [ counter ]);

    return (
        <div>
            <h1>MemoHook</h1>
            <h3>Counter: <small>{ counter }</small>  </h3>
            <hr />

            <p> { memoProcesoPesado } </p>

            <button 
                className="btn btn-primary"
                onClick={ increment }
            >
                +1
            </button>

            <button
                className="btn btn-outline-primary ml-3"
                onClick={ () => {
                    setShow( !show );
                }}
            >
                Show/Hide { JSON.stringify( show ) }
            </button>

        </div>
    )
}
```
---
```javascript
// <Hijo/>
import React from 'react'

export const Hijo = React.memo(({ numero, incrementar }) => {

    console.log('  Solo me genero una vez :) ');

    return (
        <button
            className="btn btn-primary mr-3"
            onClick={ () => incrementar( numero ) }
        >
            { numero }
        </button>
    )
})


// <Padre/>
import React, { useCallback } from 'react';
import { Hijo } from './Hijo';
import { useState } from 'react';

export const Padre = () => {

    const numeros = [2,4,6,8,10];
    const [valor, setValor] = useState(0);

    const incrementar = useCallback( (num) => {
        setValor( v => v + num )
    }, [ setValor ]);

    return (
        <div>
            <h1>Padre</h1>
            <p> Total: { valor } </p>

            <hr />

            {
                numeros.map( n => (
                    <Hijo 
                        key={ n }
                        numero={ n }
                        incrementar={ incrementar }
                    />
                ))
            }
        </div>
    )
}

```

---
[ir a => 🔀 useReducer](../07-useReducer/useReducer.md)