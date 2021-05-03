[Volver a inicio](../../README.md)
---

# 🔂 useLayoutEffect

La firma es idéntica a `useEffect`, pero se dispara de forma síncrona después de todas las mutaciones de DOM. Use esto para leer el diseño del DOM y volver a renderizar de forma sincrónica. Las actualizaciones programadas dentro de `useLayoutEffect` se vaciarán sincrónicamente, antes de que el navegador tenga la oportunidad de pintar.

```javascript
import React, { useLayoutEffect, useRef, useState } from 'react';
import { useFetch } from '../../hooks/useFetch';
import { useCounter } from '../../hooks/useCounter';

import './layout.css';

export const Layout = () => {

    const { counter, increment } =  useCounter(1);
    const { data } = useFetch( `https://www.breakingbadapi.com/api/quotes/${ counter }` );
    
    const { quote } = !!data && data[0];

    const pTag = useRef();
    const [boxSize, setBoxSize] = useState({});

    useLayoutEffect( () =>  {

        setBoxSize( pTag.current.getBoundingClientRect() );

    },[quote])


    return (
        <div>
            <h1>LayoutEffect</h1>
            <hr />

            <blockquote className="blockquote text-right">
                <p 
                    className="mb-0"
                    ref={ pTag }
                > 
                    { quote } 
                </p>
            </blockquote>


            <pre>
                { JSON.stringify( boxSize, null, 3 ) }
            </pre>


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

---
[ir a => 💾 useCallback-useMemo](../06-useCallback-useMemo/useCallback-useMemo.md)