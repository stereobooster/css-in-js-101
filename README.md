# CSS-in-JS 101 [![Twitter Follow](https://img.shields.io/twitter/url/http/shields.io.svg?style=social&label=Follow)](https://twitter.com/stereobooster)

![CSS-in-JS 101](images/css-in-js.png)

<!-- toc -->

- [What is CSS-in-JS?](#what-is-css-in-js)
- [Inline styles](#inline-styles)
  * [Pros](#pros)
  * [Cons](#cons)
  * [Inline styles vs CSS-in-JS](#inline-styles-vs-css-in-js)
- [Style tag](#style-tag)
  * [Pros](#pros-1)
  * [Cons](#cons-1)
- [CSS Modules](#css-modules)
  * [Pros](#pros-2)
  * [Cons](#cons-2)
  * [Dead Code Elimination, Critical CSS](#dead-code-elimination-critical-css)
- [Global Name Space, Globally Unique Identifier](#global-name-space-globally-unique-identifier)
- [Dependencies](#dependencies)
- [Minification](#minification)
  * [Traditional CSS minification](#traditional-css-minification)
  * [Minification of class name](#minification-of-class-name)
  * [Dead Code Elimination](#dead-code-elimination)
  * [Critical CSS](#critical-css)
  * [Automatic Atomic CSS](#automatic-atomic-css)
- [Sharing Constants, variables in CSS](#sharing-constants-variables-in-css)
  * [Sharing variables inside CSS](#sharing-variables-inside-css)
  * [Sharing variables in CSS and exposing it to JS](#sharing-variables-in-css-and-exposing-it-to-js)
  * [Passing variable from JS to CSS](#passing-variable-from-js-to-css)
- [Non-deterministic Resolution](#non-deterministic-resolution)
- [Isolation](#isolation)
- [Theming](#theming)
  * [Overriding styles](#overriding-styles)
  * [Overriding theme variables](#overriding-theme-variables)
- [SSR, Server-Side Rendering](#ssr-server-side-rendering)
  * [HTML SSR](#html-ssr)
  * [CSS SSR](#css-ssr)
- [Zero runtime dependency](#zero-runtime-dependency)
- [CSS-in-JS implementation specific features](#css-in-js-implementation-specific-features)
  * [Non-DOM targets](#non-dom-targets)
  * [CSS as object (object literal)](#css-as-object-object-literal)
  * [CSS as template literal](#css-as-template-literal)
  * [Framework agnostic](#framework-agnostic)
  * [Build step](#build-step)
  * [Dynamic](#dynamic)
  * [Generate components based on CSS](#generate-components-based-on-css)
  * [Developer tools integration](#developer-tools-integration)
- [Progressive enhancement, graceful degradation](#progressive-enhancement-graceful-degradation)
- [Uncovered subjects](#uncovered-subjects)
  * [Security](#security)
  * [Async components](#async-components)
  * [CSS-in-JS and Async components](#css-in-js-and-async-components)
  * [Atomic CSS](#atomic-css)
  * [Animations](#animations)

<!-- tocstop -->

## What is CSS-in-JS?

CSS-in-JS is an umbrella term for technologies which help you define styles in JS in more component-approach-way. The idea was introduced by `@vjeux` in 2014. Initially, it was described as inline styles, but since then battlefield changed a lot. There are about 50 different solutions in this area.

#### References

- [@vjeux, 2014][@vjeux, 2014]
- [Big list of CSS-in-JS solutions](http://michelebertoli.github.io/css-in-js/)
- [Comparison of CSS-in-JS by styled-components](https://github.com/styled-components/comparison/pull/44)
- [Yet another comparison](http://seifi.org/css/sxsw-slides-css-in-react-the-good-the-bad-and-the-ugly.html)

## Inline styles

This is built in feature of React. You can pass styles as an object to the component and it will be converted to the string and attached as style attribute to the element.

### Pros

- No `gloabl namespace`
- `Full isolation`
- No `non-deterministic resolution`
- Clear `dependencies`
- `Dead code elimination`
- `Variables, Passing variable from JS to CSS`

### Cons

- Code duplication in case of SSR.
- Additional costs in JS payload. Remember that styles which are embedded in JS are not for free. It is not only about download time, it is also about parsing and compiling. See [this detailed explanation by Addy Osmani, why JS is expensive](https://medium.com/dev-channel/the-cost-of-javascript-84009f51e99e)
- No media queries (`@media`)
- No CSS animations (`@keyframes`)
- No pseudo classes (`:hover`)
- No web fonts (`@font`)
- No autoprefixer (well there is [inline-style-prefixer](https://github.com/rofrischmann/inline-style-prefixer))

#### Example

**TODO**: verify

JSX:

```jsx
hundred_length_array
  .map(x => <div key={x} style={{color: "#000"}}></div>)
```

Generated HTML:

```html
<div style="color:#000"></div>
...(98 times)
<div style="color:#000"></div>
```

### Inline styles vs CSS-in-JS

`@mxstbr` differentiate `Inline styles` and `CSS-in-JS`. By `Inline styles` he means React built-in support for style attribute and by `CSS-in-JS` he means a solution which generates CSS and injects it via style tag.

On the other hand, `CSS-in-JS` is the term coined by `@vjeux` in 2014 and he meant `Inline styles`. `Inline styles` is not React-only feature. There is, for example, Radium which also uses `inline styles`.

So I would suggest to use `CSS-in-JS` as an umbrella term and specify implementation:
- inline styles
- style tag. Also can be referred as "style element" or "real CSS"
- mixed (like Radium)

#### References

- [@mxstbr, 2016][@mxstbr, 2016]

## Style tag

This approach is alternative to `Inline styles`. Instead of attaching styles as property to the element you are inserting real CSS in style tag and append style tag to the document.

Pros and cons can vary from implementation to implementation. But basically, it looks like this:

### Pros

- (Almost) No `global namespace`
- (Almost) `Full isolation`
- (Almost) No `non-deterministic resolution`
- Clear `dependencies`
- `Dead code elimination`
- `Variables` (depends on implementation)
- No code duplication in case of SSR
- Additional costs in JS payload (depends on implementation)
- Media queries (`@media`)
- CSS animations (`@keyframes`)
- Pseudo-classes (`:hover`)
- Web fonts (`@font`)

### Cons

Cons depend on implementation.

#### Example

[from this blog post](https://medium.learnreact.com/the-style-tag-and-react-24d6dd3ca974):

```js
const MyStyledComponent = props =>
  <div className="styled">
    Hover for red
    <style dangerouslySetInnerHTML={{__html: `
      .styled { color: blue }
    `}} />
  </div>
```

Generated HTML:

```html
<div class="styled">
  Hover for red
  <style>      .styled { color: blue }    </style>
</div>
```

## CSS Modules

A CSS Module is a CSS file in which all class names and animation names are scoped locally by default. All URLs (url(...)) and @imports are in module request format (./xxx and ../xxx means relative, xxx and xxx/yyy means in modules folder, i. e. in node_modules).

### Pros

- (Almost) No `global namespace`
- (Almost) `Full isolation`
- (Almost) No `non-deterministic resolution`
- Clear `dependencies`
- (Almost) `Dead code elimination`
- `Variables, Sharing variables in CSS and exposing it to JS`
- No Code duplication in case of SSR
- No Additional costs in JS payload.
- Media queries (`@media`)
- CSS animations (`@keyframes`)
- Pseudo-classes (`:hover`)
- Web fonts (`@font`)
- Autoprefixer

### Cons

See all points with "(Almost)"

### Dead Code Elimination, Critical CSS

Strictly speaking, there are no official solutions to those problems in `CSS Modules`, but there is some work in this direction. Correct me if I'm wrong if there is one, why isn't it promoted?

- [Comment by @sokra on critical CSS](https://github.com/webpack-contrib/style-loader/pull/159#issuecomment-286729044)
- [isomorphic-style-loader](https://github.com/kriasoft/isomorphic-style-loader)
- [Support Tree shaking of CSS ](https://github.com/webpack-contrib/css-loader/issues/506)
- [Atomic CSS a la styletron](https://github.com/deamme/styletron-loader)
- [CSSO scopes](https://github.com/css/csso#scopes)

#### References

- [css-modules](https://github.com/css-modules/css-modules)
- [@markdalgleish, 2015][@markdalgleish, 2015]

## Global Name Space, Globally Unique Identifier

All declarations in CSS are global, which is bad because you never know what part of application change in global scope will affect.

#### Possible solutions

- Attach styles to each element (`Inline styles`)
- Use Globally Unique Identifiers for classes (`CSS modules`)
- Use naming conventions (BEM and others)

#### References

- [@vjeux, 2014][@vjeux, 2014]

## Dependencies

Ability to programmatically resolve dependency between component (JS and HTML) and styles, to decrease error of forgetting to provide appropriate styles, to decrease fear of renaming CSS classes or moving them between files.

#### Possible solutions

- bundle styles with-in component (CSS-in-JS)
- `import styles from "styles.css"` (CSS modules)

#### Related
- `Dead Code Elimination`

## Minification

There is more than one aspect of minification. Let's explore:

### Traditional CSS minification

This is the simplest approach - remove whitespace, minify color name, remove unnecessary quotes, collapse CSS rules etc. See big list of minifiers [here](https://goalsmashers.github.io/css-minification-benchmark/)

### Minification of class name

In CSS modules and CSS-in-JS you do not use class names directly, instead, you use JS variables, so class names can be easily mangled.

Note: This type of minification is not possible for traditional CSS.

#### Example

```js
import styles from "styles.css";

<div className={styles.example} />
```

`styles` compiles to `{ example: "hASh"}`

### Dead Code Elimination

Because there is no connection between JS/HTML and CSS, you cannot be sure if it is safe to remove some parts of CSS or not. If it is stale or not? If it is used somewhere or not?

`CSS-in-JS` solves this problem because of a link between JS/HTML and CSS is known, so it is easy to track if this CSS rule required or not.

#### Related
- `Dependencies`
- `Critical CSS`

##### References

- [@vjeux, 2014][@vjeux, 2014]

### Critical CSS

The ability of a system to extract and inline styles in head required for current page viewed by the user not more nor less.

Note: this is slightly different from the definition by `@addyosmani`, [which defines critical as above-the-fold](https://github.com/addyosmani/critical).

##### Example

[aphrodite](https://github.com/Khan/aphrodite):

```js
import { StyleSheet, css } from 'aphrodite'

const styles = StyleSheet.create({
  heading: { color: 'blue' }
})

const Heading = ({ children }) => (
  <h1 className={css(styles.heading)}>
    { children }
  </h1>
)
```

```js
import { StyleSheetServer } from 'aphrodite'

const { html, css } = StyleSheetServer.renderStatic(
  () => ReactDOMServer.renderToString(<App />)
)
```

#### Related

- `Dependencies`
- `Dead Code Elimination`
- `SSR`

##### References

- [@markdalgleish, 2017][@markdalgleish, 2017]

### Automatic Atomic CSS

In CSS modules and CSS-in-JS you do not use class names directly, instead, you use JS variables, so class names can be easily mangled. The same as in "Minification of class name". But we can go further - generate smaller classes and reuse them to achieve smaller CSS

Note: This type of minification is not possible for traditional CSS.

#### Example

[styletron](https://github.com/rtsao/styletron)

```js
import {injectStyle} from 'styletron-utils';
injectStyle(styletron, {
  color: 'red',
  display: 'inline-block'
});
// → 'a d'
injectStyle(styletron, {
  color: 'red',
  fontSize: '1.6em'
});
// → 'a e'
```

#### Related

- [Atomic CSS](#atomic-css)

##### References

- [@markdalgleish, 2017][@markdalgleish, 2017]

## Sharing Constants, variables in CSS

There are different approaches.

### Sharing variables inside CSS

- [CSS variables](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables)
- [Can I use css variables](https://caniuse.com/#feat=css-variables)
- CSS variables in preprocessors, like PostCSS, SASS etc.

### Sharing variables in CSS and exposing it to JS

This is mainly a feature of `CSS modules` with variables.

#### Example

[postcss-icss-values](https://github.com/css-modules/postcss-icss-values)

```css
/* colors.css */
@value primary: #BF4040;
```

Sharing variables in CSS:
```css
@value primary from './colors.css';

.panel {
  background: primary;
}
```

Exposing it to JS:

```js
import { primary } from './colors.css';
// will have similar effect
console.log(primary); // -> #BF4040
```

### Passing variable from JS to CSS

This is only possible with `CSS-in-JS`. This approach gives maximum flexibility and dynamics.

#### Example

[styling](https://github.com/andreypopp/styling)

```js
import styling from 'styling'
import {baseColor} from './theme'

export let button = styling({
  backgroundColor: baseColor
})
```

#### Related

- `Overriding theme variables`

## Non-deterministic Resolution

Resolution depends on the order of declarations in stylesheets (if declarations have the same specificity).

##### References

- [@vjeux, 2014][@vjeux, 2014]

## Isolation

Because of CSS cascading nature and Global Name Space, there is no way to isolate things. Any other part code can use more specificity or use `!important` to override your "local" styles and it is hard to prevent this situation

Strictly speaking, only inline styles gives full isolation. Every other solution gives just a bit more isolation over pure CSS, because of solving Global Name Space problem.

##### References

- [@vjeux, 2014][@vjeux, 2014]

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Two CSS properties walk into a bar.<br><br>A barstool in a completely different bar falls over.</p>&mdash; Thomas &quot;Kick Nazis out, @jack&quot; Fuchs (@thomasfuchs) <a href="https://twitter.com/thomasfuchs/status/493790680397803521?ref_src=twsrc%5Etfw">July 28, 2014</a></blockquote>

## Theming

The idea is to be able to change the look of existing components without the need to change actual code.

### Overriding styles

This way you can override styles based on "class names" (keys of objects in case of inline styles).

#### Example

[react-themeable](https://github.com/markdalgleish/react-themeable)

With `CSS modules`:

```js
import theme from './MyComponentTheme.css';
<MyComponent theme={theme} />
```

Same with inline styles:

```js
const theme = {
  foo: {
    'color': 'red'
  },
  bar: {
    'color': 'blue'
  }
};
<MyComponent theme={theme} />
```

### Overriding theme variables

This way you can override styles based on variables passed to the theme. The theme basically works like a function - accepts variables as input and produce styles as a result.

#### Related
- `Variables, Passing variable from JS to CSS`

## SSR, Server-Side Rendering

### HTML SSR
Make sure that `CSS-in-JS` solution doesn't brake default React isomorphism e.g. you are able to generate HTML on the server, but not necessary CSS.

### CSS SSR
Be able to prerender CSS on the server the same way as HTML can be prerendered for React.

#### Example

[aphrodite](https://github.com/Khan/aphrodite):

```js
import { StyleSheet, css } from 'aphrodite'

const styles = StyleSheet.create({
  heading: { color: 'blue' }
})

const Heading = ({ children }) => (
  <h1 className={css(styles.heading)}>
    { children }
  </h1>
)
```

```js
import { StyleSheetServer } from 'aphrodite'

const { html, css } = StyleSheetServer.renderStatic(
  () => ReactDOMServer.renderToString(<App />)
)
```

**TODO**: add example with `Inline Styles`

#### Related
- `Critical CSS`

## Zero runtime dependency

Almost all `CSS-in-JS` solutions have runtime dependency e.g. library required to generate styles at runtime and CSS encoded as JS.

Some solutions do not have this issue, they basically vanished after build step. Examples: `CSS modules`, linaria.

#### Example

[linaria](https://github.com/callstack/linaria)

```js
import React from 'react';
import { css, styles } from 'linaria';

const title = css`
  text-transform: uppercase;
`;

export function App() {
  return <Header {...styles(title)} />;
}
```

Transpiled to:

```css
.title__jt5ry4 {
  text-transform: uppercase;
}
```

```js
import React from 'react';
import { styles } from 'linaria/build/index.runtime';

const title = 'title__jt5ry4';

export function App() {
  return <Header {...styles(title)} />;
}
```

## CSS-in-JS implementation specific features

### Non-DOM targets

React can target different platforms, not just DOM. It would be nice to have `CSS-in-JS` solution which supports different platforms too. For example: [React Native](https://facebook.github.io/react-native/), [Sketch](https://github.com/airbnb/react-sketchapp).

### CSS as object (object literal)

```js
const color = "red"
const style = {
  color: 'red',
}
```

### CSS as template literal

```js
const color = "red"
const style = `
  color: ${color};
`
```

### Framework agnostic

Does it depend on React or not?

### Build step

If build step required or not?

#### Related

- `SSR`
- `Progressive enhancement`
- `Dynamic`

### Dynamic

If you can pass values to CSS at runtime.

Note: it is not the same as `Variables, Passing variable from JS to CSS`, for example in [linaria](https://github.com/callstack/linaria) you can pass variables from JS to CSS, but only at build time.

Note 2: cannot stop myself from drawing analogy between static and dynamic type systems.

#### Related

- `Build step`
- `Variables, Passing variable from JS to CSS`

### Generate components based on CSS

If your component has pretty simple structure and you care more about how it looks instead of markup (which most likely will be `div` anyway). You can go straight to write CSS and library will generate components for you.

#### Examples

[decss](https://github.com/kossnocorp/decss)

```js
import React from 'react'
import { Button } from './style.css'

<Button>
  Panic
</Button>
```

[styled-components](https://github.com/styled-components/styled-components)

```js
import React from 'react'
import styled from 'styled'

const Button = styled.button`
  background-color: red;
`;

<Button>
  Panic
</Button>
```

### Developer tools integration

If there are special perks for developer tools?

#### Example

[emotion](https://emotion.sh/) supports source maps for styles authored in javascript

[GIF of source maps in action](images/emotion-source-maps.gif)

## Progressive enhancement, graceful degradation

If you do not know what is it read this [article](https://www.shopify.com/partners/blog/what-is-progressive-enhancement-and-why-should-you-care).

In the context of `CSS-in-JS` it boils down to one question - will your website be styled with disabled JS.

The first requirement would be to have some HTML rendered on the server (SSR or [snapshoting](https://github.com/stereobooster/react-snap)). After this you have two options:

- prebuild CSS e.g. `Build step` required
- rendered CSS e.g. `CSS SSR` required

#### Related

- `SSR`
- `Build step`

## Uncovered subjects

### Security

See [this post](https://reactarmory.com/answers/how-can-i-use-css-in-js-securely)

### Async components

Also known as `code splitting`, `dynamic import`

Async component is a technique (typically implemented as a higher order component) for loading components with `dynamic import`. There are a lot of solutions in this field here are some examples:

- [`loadable-components`](https://github.com/smooth-code/loadable-components)
- [`react-loadable`](https://github.com/thejameskyle/react-loadable)
- [`react-async-component`](https://github.com/ctrlplusb/react-async-component)
- [`react-code-splitting`](https://github.com/didierfranc/react-code-splitting)

#### References

- [Dynamic import](https://github.com/tc39/proposal-dynamic-import) is the TC39 proposal.

> Webpack has a feature to split your codebase into “chunks” which are loaded on demand. Some other bundlers call them “layers”, “rollups”, or “fragments”. This feature is called “code splitting”.
>
> — [Code splitting](https://webpack.github.io/docs/code-splitting.html)

### CSS-in-JS and Async components

This works for most `CSS-in-JS` solutions because CSS is bundled inside JS. This is a more complicated task for CSS modules.

### Atomic CSS

Also known as `immutable`, `functional`, `utility-class`.

Idea boils down to use one property per class, so you create required look by composing more than one class. Because each class contains only one property, you do not override those properties and this can be interpreted as immutability.

Do not confuse with [Atomic CSS](https://acss.io/) framework.

#### References

- [Functional CSS - The Good, The Bad, and Some Protips for React.js Users](https://github.com/chibicode/react-functional-css-protips#sunglasses-act-i-what-is-functional-css-and-why-would-i-want-to-use-it-sunglasses)
- [immutable-css](https://github.com/johnotander/immutable-css)

### Animations

#### Sequential

Basically CSS3 animations. Pros: can be GPU accelerated.

#### Interruptible

Also known as `interactive`.

Basically JS animations. Pros: can be interrupted.

#### References

- [Building Interruptible and Responsive Interactions](https://developer.apple.com/videos/play/wwdc2014/236/)

---

[@vjeux, 2014]: http://blog.vjeux.com/2014/javascript/react-css-in-js-nationjs.html
[@markdalgleish, 2015]: https://www.youtube.com/watch?v=zR1lOuyQEt8
[@mxstbr, 2016]: https://www.youtube.com/watch?v=19gqsBc_Cx0
[@markdalgleish, 2017]: https://www.youtube.com/watch?v=X_uTCnaRe94
