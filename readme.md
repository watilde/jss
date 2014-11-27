## Dynamic stylesheets for web components.

Why do we need transpilers like [sass](http://sass-lang.com/) or [stylus](http://learnboost.github.io/stylus/) when we can use javascript to do the same and much more?

By leveraging [namespaces](http://kof.github.io/jss/examples/namespace/index.html) we can solve the [cascading](http://www.phase2technology.com/blog/used-and-abused-css-inheritance-and-our-misuse-of-the-cascade/) problem better than [bem](http://bem.info/) and make our components truly reusable and composable.

[Access css](http://kof.github.io/jss/examples/commonjs/index.html) declarations and values from js without DOM round trip.

Smaller footprint because of code reuse and no vendor specific declarations

Take a look at [examples](http://kof.github.io/jss/examples/index.html) directory.

## Syntactic differences compared to CSS

Jss styles are just plain javascript objects. They map 1:1 to css rules, except of those modified by preprocessors.

### Nested Rules

Put an `&` before a selector within a rule and it will be replaced by the parent selector and extracted to a [separate rule](http://kof.github.io/jss/examples/nested/index.html).


```javascript
{
    '.container': {
        padding: '20px',
        // Will result in .container.clear
        '&.clear': {
            clear: 'both'
        },
        // Will result in .container .button
        '& .button': {
            background: 'red'
        },
        '&.selected, &.active': {
            border: '1px solid red'
        }
    }
}
```
```css
.container {
    padding: 20px;
}
.container.clear {
    clear: both;
}
.container .button {
    background: red;
}
.container.selected, .container.active {
    border: 1px solid red;
}
```

### Inheritance

Inherit a rule(s) by using `extend` keyword. This makes it easy to reuse code. [See example.](http://kof.github.io/jss/examples/extend/index.html)


```javascript
var rules = {}

var button1 = {
    padding: '20px',
    background: 'blue'
}

rules['.button-1'] = button1

rules['.button-2'] = {
    extend: button1, // can be an array of styles
    padding: '30px'
}
```
```css
.button-1 {
    padding: 20px;
    background: blue;
}

.button-2 {
    padding: 30px;
    background: blue;
}
```

### Vendor prefixes

Vendor prefixes are handled automatically using a smart check which results are cached. [See example.](http://kof.github.io/jss/examples/vendor-prefixer/index.html)


```javascript
{
    '.container': {
        transform: 'translateX(100px)'
    }
}
```
```css
.container {
    transform: -webkit-translateX(100px);
}
```

### Multiple declarations with identical property names

I recommend to not to use this if you use jss on the client. Instead you should write a function, which makes a test for this feature support and generates just one final declaration.

In case you are using jss as a server side precompiler, you might want to have more than one property with identical name. This is not possible in js, so you can use an array.

```js
{
    '.container': {
        background: [
            'red',
            '-moz-linear-gradient(left, red 0%, green 100%)',
            '-webkit-linear-gradient(left, red 0%, green 100%)',
            '-o-linear-gradient(left, red 0%, green 100%)',
            '-ms-linear-gradient(left, red 0%, green 100%)',
            'linear-gradient(to right, red 0%, green 100%)'
        ]
    }
}
```

```css
.container {
    background: red;
    background: -moz-linear-gradient(left, red 0%, green 100%);
    background: -webkit-linear-gradient(left, red 0%, green 100%);
    background: -o-linear-gradient(left, red 0%, green 100%);
    background: -ms-linear-gradient(left, red 0%, green 100%);
    background: linear-gradient(to right, red 0%, green 100%);
}
```

## API

### Access the jss namespace

```javascript
// Pure js
var jss = window.jss

// Commonjs
var jss = require('jss')
```

### Create stylesheet

`jss.createStylesheet([rules], [named], [attributes])`

- `rules` is an object, where keys are selectors if `named` is not true
- `named` rules keys are not used as selectors, but as names, will cause auto generated class names and selectors. It will also make class names accessible via `stylesheet.classes`.
- `attributes` allows to set any attributes on style element.


```javascript
var stylesheet = jss.createStylesheet({
    '.selector': {
        width: '100px'
    }
}, {media: 'print'}).attach()
```

```css
<style media="print">
    .selector {
        width: 100px;
    }
</style>
```

### Create namespaced stylesheet.

Create a stylesheet with [namespaced](http://kof.github.io/jss/examples/namespace/index.html) rules. For this set second parameter to `true`.

```javascript
var stylesheet = jss.createStylesheet({
    myButton: {
        width: '100px',
        height: '100px'
    }
}, true).attach()

console.log(stylesheet.classes.myButton) // .jss-0
```

```css
<style>
    .jss-0 {
        width: 100px;
        height: 100px;
    }
</style>
```

### Attach stylesheet

`stylesheet.attach()`

Insert stylesheet into render tree.

```javascript
stylesheet.attach()
```

### Detach stylesheet

`stylesheet.detach()`

Remove stylesheet from render tree to increase runtime performance.

```javascript
stylesheet.detach()
```

### Add a rule

`stylesheet.addRule([selector], rule)`

Returns an array of rules, because you might have a nested rule in your style.


#### You might want to add rules dynamically.

```javascript
var rules = stylesheet.addRule('.my-button', {
    padding: '20px',
    background: 'blue'
})
```
#### Add rule with generated class name.

```javascript
var rules = stylesheet.addRule({
    padding: '20px',
    background: 'blue'
})
document.body.innerHTML = '<button class="' + rules[0].className + '">Button</button>'
```

### Get a rule

`stylesheet.getRule(selector)`

```javascript
// Using selector
var rule = stylesheet.getRule('.my-button')

// Using name, if named rule was added.
var rule = stylesheet.getRule('myButton')

```

### Add a rules

`stylesheet.addRules(rules)`

Add a list of rules.

```javascript
stylesheet.addRules({
    '.my-button': {
        float: 'left',
    },
    '.something': {
        display: 'none'
    }
})
```

### Create a rule without a stylesheet.

`jss.createRule([selector], rule)`

```javascript
var rule = jss.createRule({
    padding: '20px',
    background: 'blue'
})

// Apply styles directly using jquery.
$('.container').css(rule.style)
```

## Install

```bash
npm i jss
```

## Convert CSS to JSS


```bash
# print help
jss
# convert css
jss source.css -p > source.jss
```

## Benchmarks

To make some realistic assumptions about performance overhead, I have converted bootstraps css to jss. In `bench/bootstrap` folder you will find [jss](http://kof.github.io/jss/bench/bootstrap/jss.html) and [css](http://kof.github.io/jss/bench/bootstrap/css.html) files. You need to try more than once to have some average value.

In my tests overhead is 10-15ms.

## Run tests

### Locally
```bash
npm i
open test/local.html
```
### From github

[Tests](https://kof.github.com/jss/test)

## License

MIT
