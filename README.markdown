## Contents

* [Stateless function](#stateless-function)
* [JSX spread attributes](#jsx-spread-attributes)
* [Destructuring arguments](#destructuring-arguments)
* [Conditional rendering](#conditional-rendering)
* [Children types](#children-types)
* [Array as children](#array-as-children)
* [Function as children](#function-as-children)
* [Render callback](#render-callback)
* [Children pass-through](#children-pass-through)
* [Proxy component](#proxy-component)
* [Style component](#style-component)
* [Event switch](#event-switch)
* [Layout component](#layout-component)
* [Container component](#container-component)
* [Higher-order component](#higher-order-component)
* [State hoisting](#state-hoisting)
* [Controlled input](#controlled-input)

## Stateless function

[Stateless functions](https://facebook.github.io/react/docs/components-and-props.html)은 매우 재사용 가능한 컴포넌트를 정의하는 훌륭한 방법입니다.  
이 컴포넌트들은 `state`를 가지지 않으며 단지 함수일뿐입니다.

```js
const Greeting = () => <div>Hi there!</div>
```

이 컴포넌트들은 `props`와 `context`를 받습니다.

```js
const Greeting = (props, context) =>
  <div style={{color: context.color}}>Hi {props.name}!</div>
```

이 함수 블럭이 사용되는 곳에서 이 컴포넌트들은 로컬 변수로 정의할 수 있습니다.

```js
const Greeting = (props, context) => {
  const style = {
    fontWeight: "bold",
    color: context.color,
  }

  return <div style={style}>{props.name}</div>
}
```

But you could get the same result by using other functions.

```js
const getStyle = context => ({
  fontWeight: "bold",
  color: context.color,
})

const Greeting = (props, context) =>
  <div style={getStyle(context)}>{props.name}</div>
```

이 컴포넌트들은 `defaultProps`, `propTypes`, `contextTypes` 또한 정의할 수 있습니다.

```js
Greeting.propTypes = {
  name: PropTypes.string.isRequired
}
Greeting.defaultProps = {
  name: "Guest"
}
Greeting.contextTypes = {
  color: PropTypes.string
}
```


## JSX spread attributes

Spread Attributes는 JSX의 기능입니다. 이것은 JSX 속성으로 다른 객체의 모든 프로퍼티(property)를 넘기기 위한 문법적 설탕입니다.

예를들어, 이 두 예제는 동일합니다.
```js
// 속성으로써 쓰여진 props
<main className="main" role="main">{children}</main>

// 오브젝트로부터 "뿌려진(spread)" props
<main {...{className: "main", role: "main", children}} />
```

Use this to forward `props` to underlying components.
이것은 `props`를 근본적인 컴포넌트로 넘기는데 사용됩니다.

```js
const FancyDiv = props =>
  <div className="fancy" {...props} />
```

이제, 우리는 `FancyDiv`에 쓰여진 속성뿐만 아니라 쓰여져있지 않은 속성들이 더해졌다는 것을 예상할 수 있습니다.

```js
<FancyDiv data-id="my-fancy-div">So Fancy</FancyDiv>

// output: <div className="fancy" data-id="my-fancy-div">So Fancy</div>
```

순서가 중요하다는 것을 명심해야합니다.  
만약 `props.className`가 정의된다면, `FancyDiv`에 의해 정의된 `className`은 덮어씌워질 것입니다.

```js
<FancyDiv className="my-fancy-div" />

// output: <div className="my-fancy-div"></div>
```

우리는 className을 spread props인 `({...props})`보다 뒤에 위치 시키는 방법으로  
`FancyDiv`의 className을 항상 "win"으로 만들 수 있습니다.

```js
// 기존의 `className`이  props로 넘어온 `className`을 이깁니다!
const FancyDiv = props =>
  <div {...props} className="fancy" />
```

여러분은 이런 타입의 props를 우아하게 처리해야합니다.  
아래의 예제를 보면 컴포넌트 사용자의 `props.className`과 컴포넌트를 스타일링하기 위해 필요한 `className`가 합쳐지도록 처리했다.

```js
const FancyDiv = ({ className, ...props }) =>
  <div
    className={["fancy", className].join(' ')}
    {...props}
  />
```


## destructuring arguments

[Destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)은 ES2015의 기능입니다.  
이것은 stateless function의 `props`와 잘 어울립니다.

다음 두 예제들은 동일합니다.
```js
const Greeting = props => <div>Hi {props.name}!</div>

const Greeting = ({ name }) => <div>Hi {name}!</div>
```

[rest parameter syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters) (`...`)은 선언되고 남은 오브젝트의 모든 프로퍼티들을 모아서 넘길 수 있도록 해줍니다.

```js
const Greeting = ({ name, ...props }) =>
  <div>Hi {name}!</div>
```

결국, 이 오브젝트는 조합된 컴포넌트에 `props`를 넘기기 위해서 [JSX Spread Attributes](#jsx-spread-attributes)를 사용할 수 있습니다.

```js
const Greeting = ({ name, ...props }) =>
  <div {...props}>Hi {name}!</div>
```

Avoid forwarding non-DOM `props` to composed components. Destructuring makes this very easy because you can create a new `props` object **without** component-specific `props`.
조합된 컴포넌트에 non-DOM `props`를 전달하는 것을 피하세요. 컴포넌트 특화된 `props` 없이 새로운 `props` 오브젝트를 생성할 수 있기 때문에 Destructuring은 이것을 매우 쉽게 만듭니다.


## conditional rendering

컴포넌트 정의 안에서 여러분은 일반적인 if/else 조건문을 사용할 수 없습니다.  
대신 [The conditional (ternary) operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)가 여러분의 친구가 되어줄 것입니다.

`if`

```js
{condition && <span>Rendered when `truthy`</span> }
```

`unless`

```js
{condition || <span>Rendered when `falsey`</span> }
```

`if-else` (tidy one-liners)

```js
{condition
  ? <span>Rendered when `truthy`</span>
  : <span>Rendered when `falsey`</span>
}
```

`if-else` (big blocks)

```js
{condition ? (
  <span>
    Rendered when `truthy`
  </span>
) : (
  <span>
    Rendered when `falsey`
  </span>
)}
```


## Children types

리엑트는 많은 타입의 `children`을 렌더링 할 수 있으며 대부분의 경우 `array` 또는 `string`입니다.

`string`

```js
<div>
  Hello World!
</div>
```

`array`

```js
<div>
  {["Hello ", <span>World</span>, "!"]}
</div>
```

함수들도 `children`으로써 사용될 수 있습니다.  
[coordination with the parent component](#render-callback)가 유용하게 사용됩니다.

`function`

```js
<div>
  {(() => { return "hello world!"})()}
</div>
```


## Array as children

Providing an array as `children` is a very common. It's how lists are drawn in React.

We use `map()` to create an array of React Elements for every value in the array.

```js
<ul>
  {["first", "second"].map((item) => (
    <li>{item}</li>
  ))}
</ul>
```

That's equivalent to providing a literal `array`.

```js
<ul>
  {[
    <li>first</li>,
    <li>second</li>,
  ]}
</ul>
```

This pattern can be combined with destructuring, JSX Spread Attributes, and other components, for some serious terseness.

```js
<ul>
  {arrayOfMessageObjects.map(({ id, ...message }) =>
    <Message key={id} {...message} />
  )}
</ul>
```


## Function as children

Using a function as `children` isn't inherently useful.

```js
<div>{() => { return "hello world!"}()}</div>
```

However, it can be used in component authoring for some serious power. This technique is commonly referred to as `render callbacks`.

This is a powerful technique used by libraries like [ReactMotion](https://github.com/chenglou/react-motion). When applied, rendering logic can be kept in the owner component, instead of being delegated.

See [Render callbacks](#render-callback), for more details.

## Render callback

Here's a component that uses a Render callback. It's not useful, but it's an easy illustration to start with.

```js
const Width = ({ children }) => children(500)
```

The component calls `children` as a function, with some number of arguments. Here, it's the number `500`.

To use this component, we give it a [function as `children`](#function-as-children).

```js
<Width>
  {width => <div>window is {width}</div>}
</Width>
```

We get this output.

```js
<div>window is 500</div>
```

With this setup, we can use this `width` to make rendering decisions.

```js
<Width>
  {width =>
    width > 600
      ? <div>min-width requirement met!</div>
      : null
  }
</Width>
```

If we plan to use this condition a lot, we can define another components to encapsulate the reused logic.

```js
const MinWidth = ({ width: minWidth, children }) =>
  <Width>
    {width =>
      width > minWidth
        ? children
        : null
    }
  </Width>
```


Obviously a static `Width` component isn't useful but one that watches the browser window is. Here's a sample implementation.

```js
class WindowWidth extends React.Component {
  constructor() {
    super()
    this.state = { width: 0 }
  }

  componentDidMount() {
    this.setState(
      {width: window.innerWidth},
      window.addEventListener(
        "resize",
        ({ target }) =>
          this.setState({width: target.innerWidth})
      )
    )
  }

  render() {
    return this.props.children(this.state.width)
  }
}
```

Many developers favor [Higher Order Components](#higher-order-component) for this type of functionality. It's a matter of preference.


## Children pass-through

You might create a component designed to apply `context` and render its `children`.

```js
class SomeContextProvider extends React.Component {
  getChildContext() {
    return {some: "context"}
  }

  render() {
    // how best do we return `children`?
  }
}
```

You're faced with a decision. Wrap `children` in an extraneous `<div />` or return `children` directly. The first options gives you extra markup (which can break some stylesheets). The second will result in unhelpful errors.

```js
// option 1: extra div
return <div>{children}</div>

// option 2: unhelpful errors
return children
```

It's best to treat `children` as an opaque data type. React provides `React.Children` for dealing with `children` appropriately.

```js
return React.Children.only(this.props.children)
```

## Proxy component

*(I'm not sure if this name makes sense)*

Buttons are everywhere in web apps. And every one of them must have the `type` attribute set to "button".

```js
<button type="button">
```

Writing this attribute hundreds of times is error prone. We can write a higher level component to proxy `props` to a lower-level `button` component.

```js
const Button = props =>
  <button type="button" {...props}>
```

We can use `Button` in place of `button` and ensure that the `type` attribute is consistently applied everywhere.

```js
<Button />
// <button type="button"><button>

<Button className="CTA">Send Money</Button>
// <button type="button" class="CTA">Send Money</button>
```

## Style component

This is a [Proxy component](#proxy-component) applied to the practices of style.

Say we have a button. It uses classes to be styled as a "primary" button.

```js
<button type="button" className="btn btn-primary">
```

We can generate this output using a couple single-purpose components.

```js
import classnames from 'classnames'

const PrimaryBtn = props =>
  <Btn {...props} primary />

const Btn = ({ className, primary, ...props }) =>
  <button
    type="button"
    className={classnames(
      "btn",
      primary && "btn-primary",
      className
    )}
    {...props}
  />
```

It can help to visualize this.

```js
PrimaryBtn()
  ↳ Btn({primary: true})
    ↳ Button({className: "btn btn-primary"}, type: "button"})
      ↳ '<button type="button" class="btn btn-primary"></button>'
```

Using these components, all of these result in the same output.
```js
<PrimaryBtn />
<Btn primary />
<button type="button" className="btn btn-primary" />
```

This can be a huge boon to style maintenance. It isolates all concerns of style to a single component.

## Event switch


When writing event handlers it's common to adopt the `handle{eventName}` naming convention.

```js
handleClick(e) { /* do something */ }
```

For components that handle several event types, these function names can be repetitive. The names themselves might not provide much value, as they simply proxy to other actions/functions.

```js
handleClick() { require("./actions/doStuff")(/* action stuff */) }
handleMouseEnter() { this.setState({ hovered: true }) }
handleMouseLeave() { this.setState({ hovered: false }) }
```

Consider writing a single event handler for your component and switching on `event.type`.

```js
handleEvent({type}) {
  switch(type) {
    case "click":
      return require("./actions/doStuff")(/* action dates */)
    case "mouseenter":
      return this.setState({ hovered: true })
    case "mouseleave":
      return this.setState({ hovered: false })
    default:
      return console.warn(`No case for event type "${type}"`)
  }
}
```

Alternatively, for simple components, you can call imported actions/functions directly from components, using arrow functions.

```js
<div onClick={() => someImportedAction({ action: "DO_STUFF" })}
```

Don't fret about performance optimizations until you have problems. Seriously don't.


## Layout component


Layout components result in some form of static DOM element. It might not need to update frequently, if ever.

Consider a component that renders two `children` side-by-side.

```js
<HorizontalSplit
  leftSide={<SomeSmartComponent />}
  rightSide={<AnotherSmartComponent />}
/>
```

We can aggressively optimize this component.

While `HorizontalSplit` will be `parent` to both components, it will never be their `owner`. We can tell it to update never, without interrupting the lifecycle of the components inside.

```js
class HorizontalSplit extends React.Component {
  shouldComponentUpdate() {
    return false
  }

  render() {
    <FlexContainer>
      <div>{this.props.leftSide}</div>
      <div>{this.props.rightSide}</div>
    </FlexContainer>
  }
}
```


## Container component

"A container does data fetching and then renders its corresponding sub-component. That’s it."&mdash;[Jason Bonta](https://twitter.com/jasonbonta)

Given this reusable `CommentList` component.

```js
const CommentList = ({ comments }) =>
  <ul>
    {comments.map(comment =>
      <li>{comment.body}-{comment.author}</li>
    )}
  </ul>
```

We can create a new component responsible for fetching data and rendering the stateless `CommentList` component.

```js
class CommentListContainer extends React.Component {
  constructor() {
    super()
    this.state = { comments: [] }
  }

  componentDidMount() {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: comments =>
        this.setState({comments: comments});
    })
  }

  render() {
    return <CommentList comments={this.state.comments} />
  }
}
```

We can write different containers for different application contexts.


## Higher-order component

A [higher-order function](https://en.wikipedia.org/wiki/Higher-order_function) is a function that takes and/or returns a function. It's not more complicated than that. So, what's a higher-order component?

If you're already using [container components](#container-component), these are just generic containers, wrapped up in a function.

Let's start with our stateless `Greeting` component.

```js
const Greeting = ({ name }) => {
  if (!name) { return <div>Connecting...</div> }

  return <div>Hi {name}!</div>
}
```

If it gets `props.name`, it's gonna render that data. Otherwise it'll say that it's "Connecting...". Now for the the higher-order bit.

```js
const Connect = ComposedComponent =>
  class extends React.Component {
    constructor() {
      super()
      this.state = { name: "" }
    }

    componentDidMount() {
      // this would fetch or connect to a store
      this.setState({ name: "Michael" })
    }

    render() {
      return (
        <ComposedComponent
          {...this.props}
          name={this.state.name}
        />
      )
    }
  }
```

This is just a function that returns component that renders the component we passed as an argument.

Last step, we need to wrap our our `Greeting` component in `Connect`.

```js
const ConnectedMyComponent = Connect(Greeting)
```

This is a powerful pattern for providing fetching and providing data to any number of [stateless function components](#stateless-function).

## State hoisting
[Stateless functions](#stateless-function) don't hold state (as the name implies).

Events are changes in state.
Their data needs to be passed to stateful [container components](#container-component) parents.

This is called "state hoisting".
It's accomplished by passing a callback from a container component to a child component.

```js
class NameContainer extends React.Component {
  render() {
    return <Name onChange={newName => alert(newName)} />
  }
}

const Name = ({ onChange }) =>
  <input onChange={e => onChange(e.target.value)} />
```

`Name` receives an `onChange` callback from `NameContainer` and calls on events.

The `alert` above makes for a terse demo but it's not changing state.
Let's change the internal state of `NameContainer`.

```js
class NameContainer extends React.Component {
  constructor() {
    super()
    this.state = {name: ""}
  }

  render() {
    return <Name onChange={newName => this.setState({name: newName})} />
  }
}
```

The state is _hoisted_ to the container, by the provided callback, where it's used to update local state.
This sets a nice clear boundary and maximizes the re-usability of stateless function.

This pattern isn't limited to stateless functions.
Because stateless function don't have lifecycle events,
you'll use this pattern with component classes as well.

*[Controlled input](#controlled-input) is an important pattern to know for use with state hoisting*

*(It's best to process the event object on the stateful component)*


## Controlled input
It's hard to talk about controlled inputs in the abstract.
Let's start with an uncontrolled (normal) input and go from there.

```js
<input type="text" />
```

When you fiddle with this input in the browser, you see your changes.
This is normal.

A controlled input disallows the DOM mutations that make this possible.
You set the `value` of the input in component-land and it doesn't change in DOM-land.

```js
<input type="text" value="This won't change. Try it." />
```

Obviously static inputs aren't very useful to your users.
So, we derive a `value` from state.

```js
class ControlledNameInput extends React.Component {
  constructor() {
    super()
    this.state = {name: ""}
  }

  render() {
    return <input type="text" value={this.state.name} />
  }
}
```

Then, changing the input is a matter of changing component state.

```js
    return (
      <input
        value={this.state.name}
        onChange={e => this.setState({ name: e.target.value })}
      />
    )
```

This is a controlled input.
It only updates the DOM when state has changed in our component.
This is invaluable when creating consistent UIs.

*If you're using [stateless functions](#stateless-function) for form elements,
read about using [state hoisting](#state-hoisting) to move new state up the component tree.*
