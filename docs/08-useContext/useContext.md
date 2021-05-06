
---

# ⬇️ useContext

Acepta un objeto de contexto (el valor devuelto de `React.createContext`) y devuelve el valor de contexto actual. El valor actual del contexto es determinado por la propiedad value del `<MyContext.Provider>` ascendentemente más cercano en el árbol al componente que hace la llamada.

`const value = useContext(MyContext);`

## Context

Un problema que se nos plantea en React es cuando tenemos que pasar handlers entre varios hijos de un componente.

[Problemas en React muchos subcomponentes](./componentsProblem.png)

Y una buena solución que nos brinda React es la posiblidad de usar context
[Context React](./contextReact.png)

Context provee una forma de pasar datos a través del árbol de componentes sin tener que pasar props manualmente en cada nivel.

### Create Context y useContext (functional Components)
Creamos UserContext

```javascript
// UserContext.js
import { createContext } from 'react';

export const UserContext = createContext(null);

// MainApp.js
import React, { useState } from 'react'
import { AppRouter } from './AppRouter'
import { UserContext } from './UserContext'

export const MainApp = () => {

    const [user, setUser] = useState({});

    return (
        <UserContext.Provider value={{
            user,
            setUser
        }}>
            <AppRouter/>
        </UserContext.Provider>
    )
}

// HomeScreen.js
import React, { useContext } from 'react'
import { UserContext } from './UserContext'

export const HomeScreen = () => {

    const {user, setUser} = useContext(UserContext);

    return (
        <div>
            <h1>HomeScreen</h1>
            <hr/>
        </div>
    )
}
```
---

### Class components
```javascript
// Crear un Context para el tema actual (con "light" como valor predeterminado).
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // Usa un Provider para pasar el tema actual al árbol de abajo.
    // Cualquier componente puede leerlo, sin importar qué tan profundo se encuentre.
    // En este ejemplo, estamos pasando "dark" como valor actual.
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// Un componente en el medio no tiene que
// pasar el tema hacia abajo explícitamente.
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // Asigna un contextType para leer el contexto del tema actual.
  // React encontrará el Provider superior más cercano y usará su valor.
  // En este ejemplo, el tema actual es "dark".
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}

// ThemedButton.contextType = ThemeContext; // esta seria una alternativa a static contextType
```
---

## Consumir múltiples Context

### Functional Component (multiple context)
```javascript
import React, { useContext } from 'react'
import { ThemeContext } from '../contexts/ThemeContext'
import { AuthContext } from '../contexts/AuthContext'

const Navbar = () => {
    const { isLightTheme, light, dark } = useContext(ThemeContext);
    const { isAuthenticated, toggleAuth } = useContext(AuthContext);
    const theme = isLightTheme ? light : dark;

    return(
        <nav style={{background: theme.ui, color: theme.syntax}}>
            <h1>Context App</h1>
            <div onClick={toggleAuth}>
                {isAuthenticated ? 'Logged in' : 'Logged out'}
            </div>
            <ul>
                <li>Home</li>
                <li>About</li>
                <li>Contact</li>
            </ul>
        </nav>
    )
}

export default Navbar

```
---

### Class Component (multiple context)
```javascript
// Theme context, default to light theme
const ThemeContext = React.createContext('light');

// Contexto de usuario registrado
const UserContext = React.createContext({
  name: 'Guest',
});

class App extends React.Component {
  render() {
    const {signedInUser, theme} = this.props;

    // Componente App que proporciona valores de contexto iniciales
    return (
      <ThemeContext.Provider value={theme}>
        <UserContext.Provider value={signedInUser}>
          <Layout />
        </UserContext.Provider>
      </ThemeContext.Provider>
    );
  }
}

function Layout() {
  return (
    <div>
      <Sidebar />
      <Content />
    </div>
  );
}

// Un componente puede consumir múltiples contextos.
function Content() {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => (
            <ProfilePage user={user} theme={theme} />
          )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  );
}
```

Otra forma de escribir un Context como class podria ser:
```javascript
import React, { createContext, Component } from 'react'

export const ThemeContext = createContext();

class ThemeContextProvider extends Component {
    state = {
        isLightTheme: true,
        light: {syntax: '#555', ui: '#ddd', bg: '#eee'},
        dark: {syntax: '#ddd', ui: '#333', bg: '#555'}
    }

    render() {
        return (
            <ThemeContext.Provider value={{...this.state}}>
                {this.props.children}
            </ThemeContext.Provider>
        )
    }
}

export default ThemeContextProvider;

```


---
# React Router
```javascript
// NavBar.js
import React from 'react'
import { Link, NavLink } from 'react-router-dom'

export const NavBar = () => {
    return (
        <nav>
            <ul>
                <NavLink exact activeClassName="active" to="./">Home</NavLink>
                <NavLink exact activeClassName="active" to="/about">About</NavLink>
                <NavLink exact activeClassName="active" to="/login">Login</NavLink>
            </ul>
        </nav>
    )
}

// AppRouter.js

import React from 'react'
import {    
    BrowserRouter as Router,
    Switch,
    Route,
    Redirect
} from 'react-router-dom'
import { AboutScreen } from './AboutScreen'
import { HomeScreen } from './HomeScreen'
import { LoginScreen } from './LoginScreen'
import { NavBar } from './NavBar'

export const AppRouter = () => {
    return (
        <Router>
            <div>
                <NavBar/>
                <Switch>
                    <Route exact path="/about" component={AboutScreen}/>
                    <Route exact path="/login" component={LoginScreen}/>
                    <Route exact path="/" component={HomeScreen}/>
                    <Redirect to="/"/>
                </Switch>
            </div>
        </Router>
    )
}

// MainApp.js
import React from 'react'
import { AppRouter } from './AppRouter'

export const MainApp = () => {
    return <AppRouter/>
}
```
---


# Pruebas

```javascript
import React from 'react';
import { mount } from 'enzyme';
import { HomeScreen } from '/../components/HomeScreen';
import { UserContext } from '/../components/UserContext';


describe('Pruebas en <HomeScreen />', () => {

    const user = {
        name: 'John',
        email: 'john@gmail.com'
    }

    const wrapper = mount(
        <UserContext.Provider value={{
            user
        }}>
            <HomeScreen />  
        </UserContext.Provider>
    );

    test('debe de mostrarse correctamente', () => {

        expect( wrapper ).toMatchSnapshot();
        
    })
    
    
})

```

---

```javascript
import React from 'react';
import { mount } from 'enzyme';
import { AppRouter } from '../components/AppRouter';
import { UserContext } from '../components/UserContext';


describe('Pruebas en <AppRouter />', () => {
    
    const user = {
        id: 1,
        name: 'Juan'
    }

    const wrapper = mount( 
        <UserContext.Provider value={{
            user
        }}>
            <AppRouter /> 
        </UserContext.Provider>
    );


    test('debe de mostrarse correctamente', () => {

        expect( wrapper ).toMatchSnapshot();
        
    })
    

})

```

```javascript
import React from 'react';
import { mount } from 'enzyme';
import { UserContext } from '../components/UserContext';
import { LoginScreen } from '../components/LoginScreen';


describe('Pruebas en <LoginScreen />', () => {s
    
    const setUser = jest.fn();
    
    const wrapper = mount(
        <UserContext.Provider value={{
            setUser
        }}>
            <LoginScreen />
        </UserContext.Provider>
    )

    test('debe de mostrarse correctamente', () => {
        expect( wrapper ).toMatchSnapshot();
    });


    test('debe de ejecutar el setUser con el argumento esperado', () => {
       
        wrapper.find('button').prop('onClick')();

        expect( setUser ).toHaveBeenCalledWith({
            id: 123,
            name: 'Fernando'
        })
        
    });
    
    

})


```