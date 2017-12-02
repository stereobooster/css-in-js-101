# CSS in JS: talks

Disclaimer: information on this page can be a bit outdated. See [Readme](README.md) for latest vision.

## CSS in JS by Christopher Chedeau (@vjeux), 2014

[presentation](https://speakerdeck.com/vjeux/react-css-in-js), [video](http://blog.vjeux.com/2014/javascript/react-css-in-js-nationjs.html)

### 1. Global Name Space

All declarations in CSS are global.

**Example**: you declared `.button` in `a.css` other developer declared `.button` in `b.css`. You both have no idea you used the same name until you will get a bug.

**CSS solution**: naming convention, like BEM.

### 2. Dependencies

No connection between HTML/JS and CSS.

**Example**: you used `<div class="button" />`, but forgot to load css file where it is declared.

**CSS solution**: webpack's `require("a.css")` (kind of).

### 3. Dead Code Elimination

Other consequences of "no connection between HTML/JS and CSS".

**Example**: in your code, you use `<div class="button" />`, but in styles, you loaded much more classes and/or keyframes

**CSS solution**: you can use something like critical, but it is hard.

### 4. Minification

@vjeux talks about minification of class names. Obviously, there are a lot of general [CSS minifiers](https://goalsmashers.github.io/css-minification-benchmark/), but none of those can minify class names.

**Note**: in the end, @vjeux suggests to use inline styles, so there are no class names anymore, and this point doesn't make sense  `¯\_(ツ)_/¯`.

**CSS solution**: none

### 5. Sharing Constants

The point is that you can share variables between CSS in JS.

**Example**: breakpoints or duration of animations.

**CSS solution**: closest options are SASS, CSS variables, CSS preprocessor.

### 6. Non-deterministic Resolution

Resolution depends on the order of declarations in stylesheets (if declarations have the same specificity).

**Example**: you declared `.button` in `a.css` other developer declared `.button` in `b.css`. Depending on order of inclusion of files `.button` can be resolved to `a.css` or to `b.css`.

**CSS solution**: increase specificity (which will increase fragility and decrease portability).

### 7. Isolation

Because of CSS cascading nature and Global Name Space, there is no way to isolate things.

**Example**: you declared class `.button`, but other developer declared CSS for `div`.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Two CSS properties walk into a bar.<br><br>A barstool in a completely different bar falls over.</p>&mdash; Thomas &quot;Kick Nazis out, @jack&quot; Fuchs (@thomasfuchs) <a href="https://twitter.com/thomasfuchs/status/493790680397803521?ref_src=twsrc%5Etfw">July 28, 2014</a></blockquote>

### Prposed solution: inline styles

React:

```js
<div style={{color: "#000"}}></div>
```

Generated HTML:

```html
<div style="color:#000"></div>
```

**Downsides**:
1. Code duplication in case of SSR - imagine you have inline style for a repetitive element.
2. Additional costs in JS payload. Remember that styles which are embedded in JS are not for free. It is not only about download time, it is also about parsing and compiling. See [this detailed explanation by Addy Osmani, why JS is expensive](https://medium.com/dev-channel/the-cost-of-javascript-84009f51e99e)
3. No media queries
4. No CSS animations
5. No pseudo classes
6. No autoprefixer
7. No standard CSS minifiers

## The case for CSS modules by Mark Dalgleish (@markdalgleish), 2015

[video](https://www.youtube.com/watch?v=zR1lOuyQEt8), [presentation](https://github.com/markdalgleish/presentation-the-case-for-css-modules/blob/master/src/index.jade)

### proposed solution: [CSS modules](https://github.com/css-modules/css-modules)

**Editor note**: if you treat CSS-in-JS as an umbrella term than CSS modules fails into this category too.

Please read the specification, but basic idea is to have local CSS declarations by default and expose them as JS module, which is basically a hash which maps local CSS values to globally unique values.

Solves:

1. Global Name Space
2. Dependencies. With `import "a.css"` you will not forget to load your CSS file. Even more you can check if actual classes are declared in CSS file with [TypeScript](https://github.com/Quramy/typed-css-modules) or [Flow](https://github.com/skovhus/css-modules-flow-types).
3. Dead Code Elimination. **Not quite** - see section below.
4. Minification
5. Sharing Constants. Yes with variables
6. Non-deterministic Resolution. **No** - possible solution is to use linter
7. Isolation. **Yes and no**. You still can get yourself into troubles with `:global`, but this is antipattern

And also:

1. No Code duplication in case of SSR
2. No Additional costs in JS payload, except objects with class names, like `import style from "a.css"`, where `style == {'a': '<hash>'}`
3. Media queries
4. CSS animations
5. Pseudo-classes
6. Autoprefixer
7. Standard CSS minifiers

### Dead Code Elimination

Lets define some terminology:

**Critical CSS** - critical-path (above-the-fold) CSS (according to **@addyosmani**)

**Critical CSS** - CSS required to generate current page e.g. all components on the page (according to **@markdalgleish**).

**Dead Code Elimination** - remove stale code not used anywhere in the project, still serving more than required per current page (according to **@vjuex**, based on screenshots). **Note**: he showed this in screenshots, but proposed solution (e.g. inline styles) is actually in the realm of Critical CSS `¯\_(ツ)_/¯`.

Ordered by size (`>` stands for "is more than"):

```
CSS after Dead Code Elimination > Critical CSS according to @markdalgleish > Critical CSS according to @addyosmani
```

**Note** in further discussion I will stick to the definition of Critical CSS according to @markdalgleish.

It is (almost) possible to do Dead Code Elimination in CSS modules. See [webpack-contrib/css-loader#506](https://github.com/webpack-contrib/css-loader/issues/506), [dead-css-loader](https://github.com/simlrh/dead-css-loader).

To do Critical CSS, we need some solution to track what actually is required to render the current page at runtime. It could look something like this:

```js
import { criticalTracker } from 'criticalModules'
import { a, b } from "a.css";

// it should track styles on server,
// but can be no-op at the client
criticalTracker({ a, b }, "a.css");

const Heading = ({ children }) => (
  <h1 className={a}>
    { children }
  </h1>
)
```

And SSR will look like:

```js
import { StyleSheetServer } from 'criticalModules'

const { html, css } = StyleSheetServer.renderStatic(
  () => ReactDOMServer.renderToString(<App />)
)
```

**Related**: [CSS modulues in c-r-a](https://github.com/facebookincubator/create-react-app/pull/2285), [comment by @sokra](https://github.com/webpack-contrib/style-loader/pull/159#issuecomment-286729044)

### Theming

#### Overriding styles

Proposed solution: [react-themeable](https://github.com/markdalgleish/react-themeable)

With CSS modules:

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

#### Overriding theme variables

For this you need CSS-in-JS.

##  Styling React.JS applications by Max Stoiber (@mxstbr), 2016

[video](https://www.youtube.com/watch?v=19gqsBc_Cx0), [code](https://github.com/styled-components/comparison)

### Inline styles vs CSS-in-JS
@mxstbr differentiate `Inline styles` and `CSS-in-JS`. By `inline styles` he means React built-in support for style property and by `CSS-in-JS` he means a solution which generates CSS and injects via style tag. On the other hand, `CSS-in-JS` is the term coined by @vjeux in 2014 and he meant `Inline styles`.  But there is Radium which also uses inline styles. Also sometimes people tend to use `CSS-in-JS` as an umbrella term and CSS modules fell into this category too `¯\_(ツ)_/¯`

So I would suggest to use CSS-in-JS as an umbrella term and specify technique behind the implementation, like inline styles, style element, mixed (like Radium).

### No build requirements
The title is pretty self-explanatory. You still need build step for JSX though.

### Small and lightweight
This is a pretty hand-wavy category. @mxstbr never defines how much is a small size.

Note: it doesn't make sense to measure gzipped KB of download - because this doesn't account the cost of JS parsing and compiling. See [this post by @addyosmani for details](https://medium.com/dev-channel/the-cost-of-javascript-84009f51e99e)

Note: in the end, he will remove this column from comparison anyway `¯\_(ツ)_/¯`

### Supports global CSS
@mxstbr mentions `@font-face`. I suppose all HTML elements and `@keyframes` fall into this category.

Also, `@media` queries fall into this category. Important is to distinguish between real CSS and JS simulated `@media` queries. Second can't be prerendered as CSS at a server.

### Supports entirety of CSS
@mxstbr mentions `:hover`, `:focus` (pseudo-classes). Important is to distinguish between real CSS and JS simulated pseudo-classes. Second can't be prerendered as CSS at a server. And not all of the pseudo-classes is easy to simulate with JS.

### Colocoted
@mxstbr didn't explicitly explained it, but I suppose this is the same approach as BEM - CSS lives near the corresponding JS file.

### Isolated
See "Isolation" according to @vjeux

### Easy to override
Not sure what that suppose to mean.

### Theming
I suppose @mxstbr keep in mind "Overriding theme variables" as explained by `@markdalgleish`

### SSR
This requirement can mean two things:

First. Be able to run React in server environment e.g. do not use `window` or any other browser specific APIs for injecting CSS.

Second. Be able to prerender CSS on the server the same way as HTML can be prerendered for React (example Aphrodite).
If you will prerender HTML, but not prerender CSS, you basically lose performance boost which you can have from prerendering HTML. But there is still a benefit for crawlers who cares mainly about HTML and not CSS.

### No wrapper components
@mxstbr complains that it is kind of clumsy - you need to place wrappers everywhere and you need to remember the exact syntax.

## A Unified Styling Language by Mark Dalgleish (@markdalgleish), 2017

[presentation](https://markdalgleish.github.io/presentation-a-unified-styling-language/), [video](https://www.youtube.com/watch?v=X_uTCnaRe94), [blog](https://medium.com/seek-blog/a-unified-styling-language-d0c208de2660)

### Real CSS

When the library doesn't do inline styles - they generate real CSS. It means you can use pseudo-classes and media queries.

Example (JSS):

```js
const styles = {
  button: {
    padding: '10px',
    '&:hover': {
      background: 'blue'
    }
  },
  '@media (min-width: 1024px)': {
    button: {
      padding: '20px'
    }
  }
}
```

### Critical CSS

**Critical CSS** - CSS required to generate current page e.g. all components on the page

Example (Aphrodite):

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

### Atomic CSS

A technique to minify CSS even more by extracting repeating parts into small (atomic) classes.

Example: Styletron

see [this slide](https://markdalgleish.github.io/presentation-a-unified-styling-language/#79)

### Functions in CSS

Example: Polished

### Non DOM targets

Exapmle: ReactNative

