# @klarna/higher-order-components

[![Build Status](https://travis-ci.org/klarna/higher-order-components.svg)](https://travis-ci.org/klarna/higher-order-components)
[![npm version](https://img.shields.io/npm/v/@klarna/higher-order-components.svg?maxAge=10000)](https://www.npmjs.com/package/@klarna/higher-order-components)

This library is a collection of useful React higher-order Components.

**Note**: Documentation is a work in progress. It will probably be expanded with examples later on.

## withDisplayName (string) ... (Component)

**withDisplayName** let's you easily set the `displayName` of components.

For example:

```javascript
import {withDisplayName} from '@klarna/higher-order-components'

function UnnamedComponent () {
  return <hr />
}

const Hr = withDisplayName('Hr')(UnnamedComponent)

Hr.displayName // 'Hr'
```

This function though takes an arbitrary number of parameters. You can use this to generate functions to set namespaced `displayName`s:

```javascript
import {withDisplayName} from '@klarna/higher-order-components'

function UnnamedComponent () {
  return <hr />
}

const Hr = withDisplayName('Basic')('HTML')('Hr')(UnnamedComponent)

Hr.displayName // 'Basic.HTML.Hr'
```

## withFocusProps (props) (Component)

Adds the props to the component if the element is focused.

```javascript
// InputBlock.js
import {withFocusProps} from '@klarna/higher-order-components'

function InputBlock ({focused, onFocus, onBlur}) {
  return <div>
    <input onFocus={onFocus} onBlur={onBlur} />
    {focused ? 'It’s focused!' : 'It’s not focused'}
  </div>
}

export withFocusProps({
  focused: true
})(InputBlock)
```

## withHoverProps (props) (Component)

Adds the props to the component if the element is hovered.

```javascript
// Hovereable.js
import {withHoverProps} from '@klarna/higher-order-components'

function Hovereable ({hovered, onMouseEnter, onMouseLeave}) {
  return <div onMouseEnter={onMouseEnter} onMouseLeave={onMouseLeave}>
    {hovered ? 'I’m hovered!' : 'I’m not hovered'}
  </div>
}

export withHoverProps({
  hovered: true
})(Hovereable)
```

## withMouseDownProps (props) (Component)

Adds the props to the component if the element is being pressed with the mouse.

```javascript
// Pressable.js
import {withMouseDownProps} from '@klarna/higher-order-components'

function Pressable ({pressed, onMouseDown, onMouseUp}) {
  return <div onMouseDown={onMouseDown} onMouseUp={onMouseUp}>
    {pressed ? 'I’m pressed!' : 'I’m not pressed'}
  </div>
}

export withMouseDownProps({
  pressed: true
})(Pressable)
```

## withTouchProps (props) (Component)

Adds the props to the component if the element is being touched.

```javascript
// Pressable.js
import {withTouchProps} from '@klarna/higher-order-components'

function Touchable ({touched, onTouchStart, onTouchEnd}) {
  return <div onTouchStart={onTouchStart} onTouchEnd={onTouchEnd}>
    {touched ? 'I’m touched!' : 'I’m not touched'}
  </div>
}

export withTouchProps({
  touched: true
})(Touchable)
```

## withFPSGauge ({threshold: number}) (Component)

**withFPSGauge** allows you to track the frames per second that the browser window is achieving when your component is rendered. This is particularly useful for components that are animated.

In order to do this, **withFPSGauge** uses the `collect-fps` library to collect the rate in which `requestAnimationFrame` is being called. If the frames per second drop below a threshold (30 FPS by default) then a property is set in the decorated component to notify that the animation speed is slow (the property is `lowFPS` by default).

**withFPSGauge** passes two props down to the inner component:

- `onStartFPSCollection`: to be called when the inner component starts a heavy animation of some sort
- `onEndFPSCollection`: to be called when the animation is complete

**withFPSGauge** updates the value of the `lowFPS` prop when the collection is completed.

```javascript
class AnimatedComponent extends Component {
  componentDidMount () {
    // Say that the animation starts when the component is mounted and that
    // it takes a fixed time to complete
    this.props.onStartFPSCollection()

    setTimeout(() => {
      this.props.onEndFPSCollection()
    }, 300)
  }

  render () {
    return <div
      className={this.props.lowFPS ? 'no-animation' : 'expensive-animation'}
    />
  }
}

const DecoratedAnimatedComponent = withFPSGauge({
  threshold: 30, // default threshold of frames per second. Below this number it will be considered to be low frame rate
})(AnimatedComponent)
```

The decorated component exposes the `onLowFPS` handler. This handler will be called if the FPS counts ever drops below the threshold.

```javascript
import {render} from 'react-dom'

render(
  <DecoratedAnimatedComponent
    onLowFPS={() => console.log('fps count dropped below threshold')}
  />,
  domElement
)
```

## withDeprecationWarning (config) (Component)

A component wrapped with `withDeprecationWarning` will print an error to the console when used so that consumers know they need to update their codebase to the latest component. It can be configured with the name of a component to use instead, and a URL where to read more.

```javascript
import React from 'react'
import {withDeprecationWarning} from '@klarna/higher-order-components'

function ObsoleteUnderlinedComponent ({ children }) {
  return <u>{children}</u>
}

export default withDeprecationWarning({
  readMore: 'http://example.com/why-old-component-is-deprecated',
  useInstead: 'Underlined'
})(ObsoleteUnderlinedComponent)
```

If the component doesn’t have a defined `name` or `displayName`, you can specify its name:

```javascript
withDeprecationWarning({
  …,
  name: 'ObsoleteUnderlinedComponent'
})
```

## withUniqueFormIdentifier

**withUniqueFormIdentifier** is a helper for components that need a `name` prop, so that it defaults to a namespaced UUID if not specified. This is useful for components that wrap `checkbox` or `radio` input types, which will not behave properly without an unique name. When using those Component types as fully controlled, names are unimportant, so it’s easy to forget to add them. This is a common source of problem for this family of components, which **withUniqueFormIdentifier** helps you to avoid.

Say that you have the component:

```javascript
// Radio.jsx
import React from 'react'

function Radio ({name, value, onChange}) {
  return <div>
    <p>
      <input
        type='radio'
        name={name}
        id={`${name}-acceptable`}
        value='acceptable'
      />
      <label
        htmlFor={`${name}-acceptable`}>
        Acceptable
      </label>
    </p>

    <p>
      <input
        type='radio'
        name={name}
        id={`${name}-adequate`}
        value='adequate'
      />
      <label
        htmlFor={`${name}-adequate`}>
        Adequate
      </label>
    </p>

    <p>
      <input
        type='radio'
        name={name}
        id={`${name}-close-enough`}
        value='close-enough'
      />
      <label
        htmlFor={`${name}-close-enough`}>
        Close enough
      </label>
    </p>
  </div>
}

export default Radio
```

…you can add the `withUniqueFormIdentifier` higher-order component around it:

```diff
+import {withUniqueFormIdentifier} from '@klarna/higher-order-components'

-export default Radio
+export default withUniqueFormIdentifier(Radio)
```

…and it no longer matters if you forget to set a `name` when using it, unless you actually care about that name of course.

The UUID for this would look something like: `Radio-c821f424-053a-4175-8112-1e0a6370b4cc`

## Overridable

**Overridable** provides a way of replacing the styles or the full implementation of a component.

Say that you have a single component for a list of articles:

```javascript
function Blog ({name, articles}) {
  return <main>
    <h1>{name}</h1>

    {articles.map(({title, content}) => <Article key={title}>
      <Title>{title}</Title>
      <section>
        {content}
      </section>
    </Article>)}
  </main>
}
```

…and you want to reuse the **Blog** component but would like the **Title** to look different in your context. There are many ways of solving this issue, but in the particular use case of having exactly the same app rendering with many variations for some components, a solution is to have components that can be overridden from the outside — with a property in the React.context — so that the app is not concerned at all with the fact that the internal components might change.

**TODO**: Complete explanation with the `<Design>` component as well.

```javascript
// This is the object that you get when you
// import styles from './styles.css'
type CSSModule = {
  [key: string]: string
}

type Overridable = (
  cssModule: CSSModule,
  ?designName: string // The designName is by default the `name` or `displayName` of the Component
) => (target: ReactComponent) => Component
```

## withTheme (themeToProps) (Component)

**withTheme** allows you to configure your components so that they can take information from the `React.context` to customize some props, whenever in the tree they might be. This higher-order component is useful for theming your components without having to use `React.context` explicitly in your component implementation.

Say you have a set of textual components that support a small version of themselves via the `small: boolean` prop.

```javascript
// Title.jsx
function Title ({children, small}) {
  return <h2 style={{ fontSize: small ? '12px' : '18px' }}>{children}</h2>
}

export default Title
```

```javascript
// Paragraph.jsx
function Paragraph ({children, small}) {
  return <p style={{ fontSize: small ? '12px' : '18px' }}>{children}</p>
}

export default Paragraph
```

…and you compose them in a more complex view layer:

```javascript
// MoreComplexView.jsx
import React from 'react'
import {render} from 'react-dom'

function MoreComplexView () {
  return <div>
    <Title>Hello world!</Title>
    <div>
      <Paragraph>
        Lorem ipsum dolor sit amet et conseqtetur
      </Paragraph>
    </div>
  </div>
}

render(
  <MoreComplexView />,
  document.getElementById('root')
)
```

You could of course pass a `small` prop to the MoreComplexView and have that one send the value down to each Title and Paragraph, but it can easily get cumbersome. If you happen, for example, to use a component inside MoreComplexView that in turn uses Title or Paragraph inside, you would have to pass `small` to that new component as well, and so on and so forth. What you really want to do is to set a global option for whether the text is regular or small, which is what React.context is for. Adding support for contextProps in your Title and Paragraph components makes their implementation complex though: there is a more elegant way to do it, with the **withTheme** higher-order component:

```diff
// Title.jsx
+import {withTheme} from '@klarna/higher-order-components'

function Title ({children, small}) {
  return <h2 style={{ fontSize: small ? '12px' : '18px' }}>{children}</h2>
}

-export default Title
+export default withTheme((customizations, props) => ({
+  small: customizations.textSize === 'small'
+}))(Title)
```

```diff
// Paragraph.jsx
+import {withTheme} from '@klarna/higher-order-components'

function Paragraph ({children, small}) {
  return <p style={{ fontSize: small ? '12px' : '18px' }}>{children}</p>
}

-export default Paragraph
+export default withTheme((customizations, props) => ({
+  small: customizations.textSize === 'small'
+}))(Paragraph)
```

The predicate function that you pass to `withTheme` will only be called if there is a `customizations` from in the context, which means that wrapping your components with `withTheme` is safe since nothing will change unless that prop is set.

Now you only need to set the prop in the React.context. You can easily do that with a little help from [`react-context-props`](https://github.com/xaviervia/react-context-props):

```javascript
// Theme.jsx
import React from 'react'
import PropTypes from 'prop-types'
import {getContextualizer} from 'react-context-props'

const Theme = getContextualizer({
  customizations: PropTypes.shape({
    textSize: PropTypes.oneOf(['small', 'regular'])
  })
})

export default Theme
```

```diff
// MoreComplexView.jsx
import React from 'react'
import {render} from 'react-dom'
+import Theme from './Theme'

function MoreComplexView () {
  return <div>
    <Title>Hello world!</Title>
    <div>
      <Paragraph>
        Lorem ipsum dolor sit amet et conseqtetur
      </Paragraph>
    </div>
  </div>
}

render(
-  <MoreComplexView />,
+  <Theme customizations={{textSize: 'small'}}>
+    <MoreComplexView />
+  </Theme>,
  document.getElementById('root')
)
```

**TODO** explain:
- what the result of the predicate function will be used for (give an example)
- why the props are necessary in the predicate function (again, an example)
- how this could be used to make arbitrary components themeable, including third party ones

## withUncontrolledProp (config) (Component)

**withUncontrolledProp** is a generic method of making a controlled property of a Component behave as an uncontrolled prop when not set. This is the default behavior that React exposes for form components such as `<input>`:

- `<input value='Controlled' />` and `<input value='' />` will have a controlled value
- `<input />` and `<input defaultValue='Initial value, before user interaction' />` will have an uncontrolled value

By using the **withUncontrolledProp**, the prop `prop` will be treated as uncontrolled if not defined by the user and the functions specified on `handlers` will be called with the current props and the arguments that the original handlers got called with, and the return value will be used as the new value for the prop. `defaultProp` allows you to configure a new prop that, when used, will set an initial value to the prop but make it stay uncontrolled.

```javascript
import {withUncontrolledProp} from '@klarna/higher-order-components'

function Counter ({value, onClick}) {
  return <div>
    <button onClick={onClick}>
      Add one
    </button>
    {value}
  </div>
}

export default withUncontrolledProp({
  prop: 'value',
  defaultProp: 'defaultValue',
  handlers: {
    onClick: props => e => props.value + 1
  }
})(Counter)
```

> The behavior of this higher-order component is very close to combining `withState` and `withHandlers` from [`recompose`](https://github.com/acdlite/recompose/blob/master/docs/API.md#withstate). The reason why it was created anyway is that it also provides the `defaultProp`.

## License

See [LICENSE](LICENSE)

MIT License
