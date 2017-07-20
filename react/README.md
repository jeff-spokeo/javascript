# Spokeo React Guidelines

*A mostly reasonable approach to React*

These guidelines are meant to define how we structure our React projects at Spokeo, as well as identify best practices, recommended patterns, and anti-patterns.


## Table of Contents

  1. [Directory Structure/Naming Conventions](#directory-structure-naming-conventions)
  1. [Class vs Stateless Components](#class-vs-stateless-components)
  1. [Naming](#naming)
  1. [Binding](#binding)
  1. [Methods](#methods)
  1. [React and TypeScript `interfaces`](#react-and-typescript-interfaces)
  1. [Ordering](#ordering)
  1. [Redux](#redux)
  1. [Things to Avoid](#things-to-avoid)


## Directory Structure / Naming Conventions

```
  /<root-dir | react-app>
      /components
          /Component1.tsx --> contains Component1
          /Component2
              /SubComponent1.tsx
              /SubComponent2.tsx
              /actions.ts
              /reducers.ts
              /index.tsx  --> contains Component2 (the root)
          /Navigation.tsx --> contains one or more stateless navigation components
      /index.tsx --> entry point if it's a react app
      /services
          /api_client.ts
      /shared
          /device_helper.ts
          /string_utils.ts --> what's the different between a helper and a util?
```
  - **Directories**: Use snake_case (all lower case) for directories, unless it's a React component (see below).
    > Why? to be consistent with Rails naming conventions

  - **Modules**: Use snake_case with a `.ts` extension.
    > Why? we've established this standard elsewhere in the code 

  - **Classes**: Use PascalCase with a `.ts` extension.
    ```js
    // Foo.ts
    export default class Foo {
      ...
    }
    ```

  - **Components**: place all React components under a `/components` directory
    - **Single File/Single Component**: Use PascalCase with a `.tsx` extension.
      - should contain a default export for the main component
      ```jsx
      // Header.tsx

      interface IHeader = {
        title: string
      }

      export default class Header extends React.Component<IHeader, IHeader> {
        constructor(props) {
          super(props)
          this.state = {
            title: props.title
          }
        }
        render() {
          return  (
            <h1>{this.state.title}</h1>
          )
        }
      } 
      ```

    - **Single File/Multiple Components**: Use PascalCase with a `.tsx` extension.
      - should NOT contain a default export
      ```jsx
      // Navigation.tsx

      const NavItem = ({ href, text }) => (
        <a href={ href }>{ text }</a>
      )

      const NavList = ({ children }) => (
        <ul>
          { children }
        </ul>
      )

      const NavListItem = ({ href, text }) => (
        <li>
          <NavItem href text />
        </li>
      )

      export {
        NavItem,
        NavList,
        NavListItem
      }
      ```

    - **Directory**: Use PascalCase on the directory name with an `index.tsx` file.
      ```
      /UserProfile
          /Avatar.tsx
          /DetailedInfo.tsx
          /actions.ts
          /reducers.ts
          /index.tsx --> export default the UserProfile component
      ```


## Class vs Stateless Components

  - Class (or Stateful) Components
    - If you have internal state and/or refs, or you need to hook into component lifecyle methods, prefer `class extends React.Component` over [`React.createClass`](#avoid-react-createclass).
    - include only one React component per file.
    - eslint: [`react/prefer-es6-class`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/prefer-es6-class.md) 
    
    ```jsx
    // bad
    const Listing = React.createClass({
      // ...
      render() {
        return <div>{this.state.hello}</div>;
      }
    });

    // good
    class Listing extends React.Component<any, any> {
      // ...
      render() {
        return <div>{this.state.hello}</div>;
      }
    }
    ```

  - [Stateless Functional (or Pure) Components](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions)
    - If you don't have state or refs, prefer normal functions over classes.
    - multiple SFC's are allowed per file. 
      - eslint: [`react/no-multi-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-multi-comp.md#ignorestateless)
    - eslint: [`react/prefer-stateless-function`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/prefer-stateless-function.md)
    
    ```jsx
    // bad
    class Listing extends React.Component<any, any> {
      render() {
        return <div>{this.props.hello}</div>;
      }
    }
    ```

    ```jsx
    // good
    function Listing({ hello }) {
      return <div>{hello}</div>;
    }

    // good (NOTE: AirBnb discourages function name inference, but we like it)
    const Listing = ({ hello }) => (
      <div>{hello}</div>
    );

    // ok (it works, but is less clear what props the component is expecting)
    const Listing = (props) => (
      <div>{props.hello}</div>
    );
    ```

## Naming
  
  - **Reference Naming**: Use PascalCase for React components and camelCase for their instances. eslint: [`react/jsx-pascal-case`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-pascal-case.md)

    - References (PascalCase)

    ```jsx
    // bad
    import myProfile from './MyProfile';

    // good
    import MyProfile from './MyProfile';
    ```

    - Instances (camelCase)
    
    ```jsx
    // bad
    const MyProfile = <MyProfile />;

    // good
    const myProfile = <MyProfile />;
    ```

  - **Component Naming**: Use the filename as the component name. For example, `Footer.tsx` should have a reference name of `Footer`. However, for root components of a directory, use `index.tsx` as the filename and use the directory name as the component name:

    ```jsx
    // bad
    import Footer from './Footer/Footer';

    // bad
    import Footer from './Footer/index';

    // good
    import Footer from './Footer';
    ```
  - **Higher-order Component Naming**: Use a composite of the higher-order component's name and the passed-in component's name as the `displayName` on the generated component. For example, the higher-order component `withFoo()`, when passed a component `Bar` should produce a component with a `displayName` of `withFoo(Bar)`.

    > Why? A component's `displayName` may be used by developer tools or in error messages, and having a value that clearly expresses this relationship helps people understand what is happening.

    ```jsx
    // bad
    export default function withFoo(WrappedComponent) {
      return function WithFoo(props) {
        return <WrappedComponent {...props} foo />;
      }
    }

    // good
    export default function withFoo(WrappedComponent) {
      function WithFoo(props) {
        return <WrappedComponent {...props} foo />;
      }

      const wrappedComponentName = WrappedComponent.displayName
        || WrappedComponent.name
        || 'Component';

      WithFoo.displayName = `withFoo(${wrappedComponentName})`;
      return WithFoo;
    }
    ```

## Binding

  - AVOID: binding event handlers within the render method
    - eslint: [`react/jsx-no-bind`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md)
    > Why? A bind call in the render path creates a brand new function on every single render.

    ```jsx
    // bad (binding in the render method)
    class extends React.Component {
      onClickDiv() {
        // do stuff
      }

      render() {
        return <div onClick={this.onClickDiv.bind(this)} />;
      }
    }
    ```

  - AVOID: lodash decorators (`BindAll()`, `Bind()`)

    > Why? We want to avoid importing the lodash library in order to keep our file size down.  There are other suitable alternatives defined below.

    ```jsx
    // bad (lodash class decorator)
    @BindAll() // does NOT work with preact!
    class extends React.Component<any, any> {
      onClickDiv() {
        // do stuff
      }

      render() {
        return <div onClick={this.onClickDiv} />;
      }
    }
    ```

    ```jsx
    // bad (lodash method decorator)
    class extends React.Component<any, any> {

      @Bind()
      onClickDiv() {
        // do stuff
      }

      render() {
        return <div onClick={this.onClickDiv} />;
      }
    }
    ```

  - DO: Bind via class arrow functions
    ```jsx
    // good (class arrow functions)
    class extends React.Component {
      onClickDiv = () => {
        // do stuff
      }

      render() {
        return <div onClick={this.onClickDiv} />;
      }
    }
    ```

  - DO: Bind in the constructor
    ```jsx
    // good (constructor)
    class extends React.Component {
      constructor(props) {
        super(props);

        this.onClickDiv = this.onClickDiv.bind(this);
      }

      onClickDiv() {
        // do stuff
      }

      render() {
        return <div onClick={this.onClickDiv} />;
      }
    }
    ```


## Methods

  - Use arrow functions to close over local variables.

    ```jsx
    function ItemList(props) {
      return (
        <ul>
          {props.items.map((item, index) => (
            <Item
              key={item.key}
              onClick={() => doSomethingWith(item.name, index)}
            />
          ))}
        </ul>
      );
    }
    ```

## TypeScript vs PropTypes

Prefer TypeScript's `interface` and `generics` API over React's `propTypes` and `defaultProps` for defining the component props interface.

  > Why? TypeScript provides robust real-type type checking and intellisense capabilities.  Also, PropTypes have been extracted out of react core as of v15.5

  - Strongly-typed Class Component

    ```jsx
    interface IProps {
      id: number;
      url: string;
      text?: string;
    };

    interface IState {
      foo: string;
      bar?: string;
    }

    class Box extends React.Component<IProps, IState> {

      static defaultProps = {
        text: 'Hello World!'
      }

      constructor(props) {
        super(props)

        this.state = {
          foo: 'this is foo',
          bar: 'this is bar'
        }
      }

      render() {
        return (
          <div>
            <label>{ this.state.foo }</label>
            <a href={ this.props.url } data-id={ this.props.id }>{ this.props.text }</a>
            <span>{ this.state.bar }</span>
          </div>
        )
      }
    }
    ```

  - Strongly-typed Stateless Functional Component

    ```jsx
    // good (destructure the props argument and define the types directly on the argument list)
    const Link = ({ id: number, url: string, text?: string }) => (
      <a href={ url } data-id={ id }>{text}</a>
    )
    ```

    ```jsx
    // good (define an interface. suitable when there are many props.)
    interface ILink = {
      id: number;
      url: string;
      text?: string;
      foo: string;
      bar: string;
    }
    const Link: React.SFC<ILink> = (props) => (
      <div>
        <label>{ props.foo }</label>
        <a href={ props.url } data-id={ props.id }>{ props.text }</a>
        <span>{ props.bar }</span>
      </div>
    )
    ```

## Ordering

Ordering for `class extends React.Component`:

  1. optional `static` methods
  1. `constructor`
  1. `getChildContext`
  1. `componentWillMount`
  1. `componentDidMount`
  1. `componentWillReceiveProps`
  1. `shouldComponentUpdate`
  1. `componentWillUpdate`
  1. `componentDidUpdate`
  1. `componentWillUnmount`
  1. *clickHandlers or eventHandlers* like `onClickSubmit()` or `onChangeDescription()`
  1. *getter methods for `render`* like `getSelectReason()` or `getFooterContent()`
  1. *optional private methods* like `doSomething()` or `doSomethingElse()`
  1. *optional render methods* like `renderNavigation()` or `renderProfilePicture()`
  1. `render`

## Redux

For components that use Redux... go here (TODO: create redux README)

## Things to Avoid

- <a id="avoid-react-createclass"></a>`React.createClass`
  - [Do not use](https://facebook.github.io/react/blog/2017/04/07/react-v15.5.0.html).
  > Why? `React.createClass` is deprecated as of v15.5.0. It will still work, but they've extracted it out of core React for optimization.

- Mixins

  - [Do not use mixins](https://facebook.github.io/react/blog/2016/07/13/mixins-considered-harmful.html).

  > Why? Mixins introduce implicit dependencies, cause name clashes, and cause snowballing complexity. Most use cases for mixins can be accomplished in better ways via components, higher-order components, or utility modules.

- `isMounted`

  - Do not use `isMounted`. eslint: [`react/no-is-mounted`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-is-mounted.md)

  > Why? [`isMounted` is an anti-pattern][anti-pattern], is not available when using ES6 classes, and is on its way to being officially deprecated.

  [anti-pattern]: https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html



**[⬆ back to top](#table-of-contents)**
