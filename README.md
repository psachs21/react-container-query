# React Container Query

[![npm version](https://badge.fury.io/js/react-container-query.svg)](https://badge.fury.io/js/react-container-query)
[![Build Status](https://travis-ci.org/d6u/react-container-query.svg?branch=master)](https://travis-ci.org/d6u/react-container-query)

[![Sauce Test Status](https://saucelabs.com/browser-matrix/react-cq.svg)](https://saucelabs.com/u/react-cq)

## Intro

**True modularity in styling responsive component.**

Modularity is the heart of component based UI. With most JavaScript modularized, CSS failed to catch up. When developing responsive webpage, we use media queries to toggle styles based on the size of the viewport. This creates problems when creating component level styles. The same component will behave differently when it is placed in different locations on a page. It seriously breaks the modularity of a component. We need components to be responsive and independent of viewport sizes.

## What is Container Query

**For real usage see [usage](#usage) section.**

Container query is a work in process CSS feature. "Container queries allow an author to control styling based on the size of a containing element rather than the size of the user’s viewport." (from [Container Query](http://responsiveimagescg.github.io/container-queries/)). [Container Queries: Once More Unto the Breach](http://alistapart.com/article/container-queries-once-more-unto-the-breach) is the inspiration of this repo.

With below CSS, `.box` will be blue when `.container` is wider than 600px, green when width between 400px and 599px, and red for the rest of time.

```css
.box {
  background-color: red;
}

.container:media(min-width: 400px) {
  .box {
    background-color: green;
  }
}

.container:media(min-width: 600px) {
  .box {
    background-color: blue;
  }
}
```

## Usage

```sh
npm install --save-dev react-container-query
```

### `applyContainerQuery(Component, query, [opts])`

Create [higher order component](https://gist.github.com/sebmarkbage/ef0bf1f338a7182b6775) for container.

```js
import React, { Component } from 'react';
import { render } from 'react-dom';
import classnames from 'classnames';
import { applyContainerQuery } from 'react-container-query';

class MyComponent extends Component {
  render() {
    // `this.props.containerQuery` is a property provided by the higher order
    // component. It will look like:
    //
    //   {
    //     width_between_400_and_599: true,
    //     width_larger_than_600: false
    //   }
    return (
      <div className={classnames('container', this.props.containerQuery)}>
        <div className='box'>the box</div>
      </div>
    );
  }
}

const query = {
  width_between_400_and_599: {
    minWidth: 400,
    maxWidth: 599
  },
  width_larger_than_600: {
    minWidth: 600,
  }
};

const HigherOrderComponent = applyContainerQuery(MyComponent, query);

render(<HigherOrderComponent/>, document.getElementById('app'));
```

#### options

- `opts.propName = "containerQuery"`: use this arguments, you can customized the name of property to pass container query state to.

- `opts.setAttribute = false`: if true, when container query changes, it will set custom attribute to the container DOM element. In above example, DOM could look like:

    ```html
    <div class='container' width_between_400_and_599>
      <div class="box">the box</div>
    </div>
    ```

    This way you can use CSS [attribute selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors) to target container elements.

### `createContainerQueryMixin(query, [opts])`

If you want greater control or prefer mixin style API, you can use this function to create mixin for your container component. Under the hood, higher order component API is implemented using function.

#### ES5

```js
var React = require('react');
var ReactDOM = require('react-dom');
var ReactContainerQuery = require('react-container-query');

var query = {
  width_between_400_and_599: {
    minWidth: 400,
    maxWidth: 599
  },
  width_larger_than_600: {
    minWidth: 600,
  }
};

var MyComponent = React.createClass({

  mixins: [ ReactContainerQuery.createContainerQueryMixin(query) ],

  render: function () {
    return (
      // IMPORTANT: assign `ref` property here
      // `defineContainer` is a method provided by `createContainerQueryMixin`
      <div ref={this.defineContainer} className='container'>
        <div className='box'>the box</div>
      </div>
    );
  }
});

ReactDOM.render(<MyComponent/>, document.getElementById('app'));
```

#### ES2015 Class

React lacks mixin support for ES2015 class syntax. You will need packages like [react-mixin](https://github.com/brigand/react-mixin) to apply container query mixin to your component.

```js
import React, { Component } from 'react';
import { render } from 'react-dom';
import reactMixin from 'react-mixin';
import { createContainerQueryMixin } from 'react-container-query';

const query = {
  width_between_400_and_599: {
    minWidth: 400,
    maxWidth: 599
  },
  width_larger_than_600: {
    minWidth: 600,
  }
};

class MyComponent extends Component {
  render() {
    return (
      // IMPORTANT: assign `ref` property here
      // `defineContainer` is a method provided by `createContainerQueryMixin`
      <div ref={this.defineContainer.bind(this)} className='container'>
        <div className='box'>the box</div>
      </div>
    );
  }
}

reactMixin(MyComponent.prototype, createContainerQueryMixin(query));

render(<MyComponent/>, document.getElementById('app'));
```

#### options

- `opts.setAttribute = false`: same as `applyContainerQuery` option

#### Lifecycle

The mixin API also provides lifecycle hooks in case you need to create additional logics depend on state of a component.

##### `containerQueryWillUpdate(stateMap: Object?)`

Will be called with current state map right before container query changes. The state map will look like:

```js
{
  width_between_400_and_599: true,
  width_larger_than_600: false
}
```

When first mounted, will be called with `null`. When component will unmount, will be called with current state map.

##### `containerQueryDidUpdate(stateMap: Object?)`

Will be called right after container query is changed. `stateMap` has the same structure as in `containerQueryWillUpdate`.

When component will unmount, will be called with `null`.

### CSS

If you set `opts.setAttribute = true`, you write CSS like below. The CSS doesn't look exactly like the container query syntax, but the idea is to switch `width_between_400_and_599` and `width_larger_than_600` attribute on and off based on the width of `.container`, so we can use [attribute selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors) to target child elements.

```css
.box {
  background-color: red;
}

.container[width_between_400_and_599] .box {
  background-color: green;
}

.container[width_larger_than_600] .box {
  background-color: blue;
}
```

Of course with `applyContainerQuery` function, you can create your own logic for applying styles.

## Demo

Checkout CodePen 

- Adjustable Sidebar http://codepen.io/daiweilu/pen/wMrrZM
- Responsive Component Layout http://codepen.io/daiweilu/pen/XXexrj

You can also checkout [react-container-query-demo](https://github.com/d6u/react-container-query-demo).

## Performance

Internally, it's using [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) API to check container element size periodically. You can checkout `demo/perf.html` to test with your own eyes. But generally this is not a concern.
