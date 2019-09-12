# Babel Plugin Style Props <!-- omit in toc -->

Use theme aware style props on any JSX element.

```jsx
<h1 mt={0} mb={4} color="primary" textDecoration="underline">
  Hello
</h1>
```

- [Features](#features)
- [Getting Started](#getting-started)
  - [Installation](#installation)
  - [Configure Babel](#configure-babel)
    - [Styled Components](#styled-components)
    - [Emotion](#emotion)
  - [Setup your `<ThemeProvider>`](#setup-your-themeprovider)
    - [Minimal theme](#minimal-theme)
    - [Tailwind](#tailwind)
- [What this plugin does](#what-this-plugin-does)
  - [Use values from your theme](#use-values-from-your-theme)
  - [Use arrays for responsive styles](#use-arrays-for-responsive-styles)
  - [Use negative values](#use-negative-values)
- [Use function calls, variables, and expressions in style props](#use-function-calls-variables-and-expressions-in-style-props)
- [Use custom variants](#use-custom-variants)
- [Opinionated gotchas](#opinionated-gotchas)
  - [Breakpoints](#breakpoints)
  - [Nested theme properties](#nested-theme-properties)
  - [Incompatible with components built with `styled-system`](#incompatible-with-components-built-with-styled-system)
- [Other limitations compared to `styled-system`](#other-limitations-compared-to-styled-system)
- [License](#license)

## Features

- Support for **all** CSS properties.
- Use values from your `<ThemeProvider>` and `theme`, or just use raw CSS units
  and properties.
- Use arrays for responsive styles.
- Performant. Equivalent to using `styled-components` or `emotion` directly.
- Customizable variants.
- Removes style props from rendered HTML.

## Getting Started

### Installation

```bash
# yarn
yarn add -D babel-plugin-style-props

# npm
npm i -D babel-plugin-style-props
```

### Configure Babel

Add the plugin to your Babel config file and specify the `stylingLibrary`
option. Be sure that the appropriate `css` prop babel plugin is included
**after** this plugin.

See below for examples with popular CSS-in-JS libraries.

#### Styled Components

```js
// babel.config.js
module.exports = {
  presets: ['@babel/preset-env', '@babel/preset-react'],
  plugins: [
    [
      'babel-plugin-style-props',
      {
        stylingLibrary: 'styled-components',
      },
    ],
    'babel-plugin-styled-components',
  ],
}
```

#### Emotion

```js
// babel.config.js
module.exports = {
  presets: [
    '@babel/preset-env',
    '@babel/preset-react',
    '@emotion/babel-preset-css-prop',
  ],
  plugins: [
    [
      'babel-plugin-style-props',
      {
        stylingLibrary: 'emotion',
      },
    ],
    'babel-plugin-styled-components',
  ],
}
```

### Setup your `<ThemeProvider>`

Place your `<ThemeProvider>` component around your React app as you normally
would, and pass your `theme` object.

```jsx
import { ThemeProvider } from 'styled-components'
import { theme } from './pathToYourTheme'

const YourApp = () => (
  <ThemeProvider theme={theme}>
    <App />
  </ThemeProvider>
)
```

In order for this plugin to work, you **must** specify a `theme` and
`<ThemeProvider>`.

#### Minimal theme

For a barebones theme to start working with, see this
[example](docs/examples/minimalTheme.js).

#### Tailwind

For a TailwindCSS copycat theme to get started with, see this
[example](docs/examples/tailwindTheme.js).

Your `theme` should follow the `styled-system` specification that you can find
detailed [here](https://styled-system.com/theme-specification).

## What this plugin does

`babel-plugin-style-props` converts style props to an object or function in a
`css` prop. This allows libraries like `styled-components` or `emotion` to parse
the styles into CSS.

```jsx
// Your JSX
<div color='red' px={5} />

// Output JSX (simplified): `styled-components`
<div
  css={theme => ({
    color: p.theme.colors.red,
    paddingLeft: p.theme.space[5],
    paddingRight: p.theme.space[5],
  })}
/>

// Output JSX (simplified): `emotion`
<div
  css={theme => ({
    color: theme.colors.red,
    paddingLeft: theme.space[5],
    paddingRight: theme.space[5],
  })}
/>
```

### Use values from your theme

When colors, fonts, font sizes, a spacing scale, or other values are definied in
a `<ThemeProvider>`, the values can be referenced by key in the props.

```jsx
// example theme
const theme = {
  // ...
  colors: {
    primary: '#07c',
    muted: '#f6f6f9',
  },
}

<div color="primary" bg="muted" />
```

### Use arrays for responsive styles

You can use arrays to specify responsive styles.

```jsx
<div width={['100%', '50%', '25%']} />
```

Responsive arrays will generate styles according to the breakpoints defined in
your babel config. See [breakpoints](#breakpoints) for more info.

### Use negative values

When a style prop has keys that are defined in a `<ThemeProvider>`, you can
negate them by prefixed them with a '-' (hyphen).

```jsx
const theme = {
  // ...
  space: [
    0,
    '5rem'
  ]
}
// theme alias
theme.space.large = theme.space[1]

<div mt="-large" mr={-1} />

// transpiles to:
<div
  css={theme => ({
    marginTop: '-' + theme.space.large,
    marginRight: '-' + theme.space[1]
  })}
/>

// resulting in:
<div css={theme => ({ marginTop: '-5rem', marginRight: '-5rem' })} />
```

## Use function calls, variables, and expressions in style props

Function calls, expressions, and variables are dropped into the `css` prop as
computed properties. Consider the following example:

```jsx
const Box = () => {
  const myColor = 'primary'
  const myFunction = () => 'muted'
  const boolean = true
  const size = 'small'

  return <div color={myColor} bg={myFunction()} mt={boolean ? 'large' : size} />
}

// Transpiles to:
const Box = () => {
  const myColor = 'primary'
  const myFunction = () => 'muted'
  const boolean = true
  const size = 'small'

  return (
    <div
      css={theme => ({
        color: theme.colors[myColor], // theme.colors.primary
        backgroundColor: theme.colors[myFunction()], // theme.colors.muted
        marginTop: theme.space[boolean ? 'large' : size], // theme.space.large || theme.space.small
      })}
    />
  )
}
```

If you are using `styled-components`, this plugin will automatically handle
passing your functions, variables, and expressions as props to the `styled.div`
that is generated by `babel-plugin-styled-components`.

## Use custom variants

Custom variants and style props can be defined in the babel plugin options under
`variants`. See below for an example config:

```js
// babel.config.js
module.exports = {
  presets: ['@babel/preset-env', '@babel/preset-react'],
  plugins: [
    [
      'babel-plugin-style-props',
      {
        stylingLibrary: 'styled-components',
        variants: {
          boxStyle: 'boxStyles',
        },
      },
    ],
    'babel-plugin-styled-components',
  ],
}
```

The above config will tell `babel-plugin-style-props` to transpile the
`boxStyle` prop on any JSX element to properties in the `css` prop.

```jsx
const theme = {
  // ...
  boxStyles: {
    primary: {
      color: 'white',
      backgroundColor: '#f0f'
    }
  }
}

// `boxStyle` on an element:
<div boxStyle="primary" />

// will transpile to:
<div css={theme => ({ ...theme.boxStyles.primary })} />

// which results in:
<div css={theme => ({ color: 'white', backgroundColor: '#f0f' })} />
```

Currently, variants can only specify raw CSS rules (no theme values). In the
future, they will be able to support `theme` values.

## Opinionated gotchas

To achieve a similar API to `styled-system`/`theme-ui` without the performance
cost, this plugin makes some opinionated decisions as to how you can structure
your theme.

### Breakpoints

Unlike `styled-system`, breakpoints can **only** be configured in the Babel
plugin options. This is an intentional limitation for performance.

```js
// babel.config.js
module.exports = {
  presets: ['@babel/preset-env', '@babel/preset-react'],
  plugins: [
    [
      'babel-plugin-style-props',
      {
        stylingLibrary: 'styled-components',
        breakpoints: ['32rem', '60rem', '100rem'],
      },
    ],
    'babel-plugin-styled-components',
  ],
}
```

### Nested theme properties

Unlike `styled-system`, this plugin **only** supports two levels of nesting in a
`theme` object. Consider the following example.

```js
// theme.js
const theme = {
  colors: {
    primary: '#fff',
    red: {
      light: '#f0f',
      dark: '#0f0',
    },
  },
  lineHeights: {
    copy: 1.5,
  },
}

const Box = () => <div color="red.light" bg="primary" />
```

The above example will not work because we are accessing a third level of
nesting for our `color` style prop. This is an intentional limitation for
performance, and is largely how this plugin eliminates the `styled-system`
runtime cost.

If you want to have namespaced-like behavior, consider flatly namespacing your
keys as a workaround.

```js
const theme = {
  colors: {
    primary: '#fff',

    'red.light': '#f0f',
    'red.dark': '#0f0',
  },
  lineHeights: {
    copy: 1.5,
  },
}
```

### Incompatible with components built with `styled-system`

Due to this plugin transpiling away style props, this plugin is incompatibile
with any component that is built with `styled-system` **or** any component that
uses any of the expected style prop names.

> In general, a style prop is the `camelCase` equivalent of any CSS property
> name.

## Other limitations compared to `styled-system`

- Cannot specify `theme` keys that begin with `-`. This plugin relies on the `-`
  preceeding a theme key to determine the negation of a scale.
- Does not transform fractional width values.
- Does not include a default theme.
- Does not parse props on SVG elements.

## License

MIT.
