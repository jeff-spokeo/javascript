# Spokeo React/Redux Guidelines

*A mostly reasonable approach to React Development at Spokeo, maintained by the Spokeo JavaScript Guild.*

## Table of Contents

  1. [Component Overview](#component-overview)
  1. [Action Creators](#action-creators)
  1. [Reducers](#reducers)
  1. [Connecting Components](#connecting-components)
  1. [Store](#store)
  1. [Middleware](#middleware)
  1. [Resources](#resources)


## Component Overview

For Reduxified components, create a directory under `/components` using the name of the component in `PascalCase`.

  ```
  /MyComponent
      /actions.ts
      /index.tsx
      /models.ts
      /MyComponent.tsx
      /reducer.ts
      /SubComponent.tsx
  ```

Within the component directory, you will typically have the following files:

1. `actions.ts` - defines the [action creators](#action-creators).
1. `index.tsx` - exports the public-facing API of this component. this MUST include the main component as the `default` export, but can optionally include any interfaces or sub components that might be shared with other components.

    ```jsx
    // components/MyComponent/index.tsx

    import MyComponent from './MyComponent'
    import { ILocation, IPerson } from './models'

    export {
      ILocation,
      IPerson
    }

    export default MyComponent
    ```

    then, in some other component, import the component and any other dependencies from the root component directory.  Avoid referencing any of the component's helper files directly.
    ```jsx
    // components/SomeOtherComponent.tsx

    // bad - do NOT reference any files under MyComponent directly
    import { ILocation, IPerson } from '../MyComponent/models'

    // good - imports interfaces from MyComponent's public API
    import MyComponent, { ILocation, IPerson } from '../MyComponent'
    ```
1. `models.ts` - any models or interfaces used by this component.
1. `MyComponent.tsx` - the definition of the main component, [connected to Redux](#connecting-components).
1. `reducer.ts` - defines the [reducers](#reducers) and action enums.
1. `SubComponent.tsx` - any sub components used by the main component.
1. any other files that are germain to the main component.


## Action Creators

Filename: `actions.ts`

In addition to standard Redux actions, we also use `redux-thunk` middleware for managing async operations (e.g. API calls).

```js
// MyComponent/actions.ts

// simple action creator
export const notifyUser = (id: number) => ({
  type: NOTIFY_USER,
  id: id
})

// with redux-thunk middleware
export const updateUser = (id: number, name: string) => {
  return (dispatch, getState) => {
    axios.post('/api/v1/user', { id, name })
      .then(response => {
        dispatch(handleSuccess(response))
      })
      .catch(error => {
        dispatch(handleError(error))
      })
  }
}

const handleSuccess = (response) => {
  //...
}

const handleError = (error) => {
  //...
}
```

## Reducers

Filename: `reducer.ts`

exports the following items: 
- root reducer
- action enums
- selectors

```js
// components/MyComponent/reducer.ts
import { MyComponentData } from "./models"

// export the action types enum (used by the action creators in actions.ts)
export enum MyComponentActionsEnum {
  CHANGE_NAME = 'CHANGE_NAME'
}

// export the root reducer
export const myComponentReducer = (state: MyComponentData, action) => {

  switch (action.type) {
    case MyComponentActionsEnum.CHANGE_NAME:
      return handleChangeName(state, action)
  }

  return state
}

// define one or more private reducers to handle specific actions
const handleChangeName = (state: MyProfileUI, action)=> {
  return {
    ...state,
    name: action.name

  }
}

// selectors encapsulate the shape of your state data. the less your components know about the shape of your state, the better.
export const fullNameSelector = (state: any) => {
  return `${state.firstName} ${state.lastName}`
}
```


## Connecting Components

  These examples show two ways to connect a React component to Redux.
  
  > *NOTE: Both approaches are valid, so developer preference wins.*

  - Classic Implementation (verbose)

    This first example defines the `mapStateToProps` and `mapDispatchToProps` methods explicitly, creates a reference to a connected component, and then exports the connected component.

    ```jsx
    // MyComponent.tsx
    import { notifyUser } from './actions'

    // ...component "MyComponent" defined or imported here

    function mapStateToProps(state) {
      return {
        id: state.profile.id,
        name: state.profile.firstName + ' ' + state.profile.lastName
      }
    }

    function mapDispatchToProps(dispatch) {
      return {
        notify: notifyUser
      }
    }

    const MyReduxifiedComponent = connect(mapStateToProps, mapDispatchToProps)(MyComponent)

    export default MyReduxifiedComponent
    ```

  - Alternative Implementation (concise)

    This second example accomplishes the same thing in a "single" line of code.

    ```jsx
    // MyComponent.tsx
    import { notifyUser } from './actions'

    // ...component "MyComponent" defined or imported here
    
    export default connect(
      state => ({
        id: state.profile.id,
        name: state.profile.firstName + ' ' + state.profile.lastName      
      }),
      dispatch => ({
        notify: notifyUser      
      })
    )(MyComponent)
    ```

  - Connecting Redux to an Existing Component

    Sometimes you want to use Redux with another component that's shared by non-Redux apps. Simply create a new component that connects the existing component to your Redux store.

    ```jsx
    // my_app/components/MyFooComponent/index.tsx
    import { connect } from 'react-redux'
    import FooComponent from 'core/components/FooComponent' // <-- import existing component
    import { notifyUser } from './actions'

    export default connect({  // <-- export the connected component
      state => ({
        id: state.profile.id,
        name: state.profile.name
      }),
      dispatch => ({
        notify: notifyUser
      }, dispatch)
    })(FooComponent)
    ```
    
  - TODO: add example that uses re-usable interfaces (randy)

## Store

TODO...

## Middleware

TODO...

## Resources

TODO...


**[⬆ back to top](#table-of-contents)**
