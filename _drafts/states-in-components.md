---
layout: default
title: Props, state and data treatment in components
resume: React not only changed the way we build our interfaces but also put data in the spotlight. This is a short take around components internals and good and bad cases of data misplacing and states.
---

I will explore different ways React component are able to contain information and how to detect cases where it might be better to move it to a different place.

## Props

By definition _props_ are initial properties that affect how a component is rendered.

**Props** must be first class citizen in your application code base, and you should try to achieve all possible _flavors_ or variants using them.

By giving _props_ this role, you will favor a unique down data flow direction.


## State

The _state_ of a component is also a set of properties that affects the result of its render function, but this time they live internally in the component.

> “Simplicity is prerequisite for reliability.” - Edsger W. Dijkstra

States are necessary, specially to respond to user input and data fetching, but they also add complexity.

That's why storing a value in a component's state should be a well thought decision.


### Should a property belong to the state?

These are three axioms I think a property should pass to be placed inside a state:

- The property must alter the render output.
- The property might change over the component's life cycle.
- The property is changed by an action which takes place inside the component.

If a piece of data doesn't pass these requirements then it should be treated as a _prop_ or even a constant value outside the component.


#### Quick example

Let's imagine a component which output changes depending on the type of device.

```js
import { Component } from 'react';
import MobileDetect from 'mobile-detect';

const ua = window.navigator.userAgent;

class Header extends Component {
  constructor(props) {
    super(props);

    this.state = {
      isMobile: new MobileDetect(ua).mobile()
    };
  }

  render() {
    return (
      <header className={this.state.isMobile ? 'mobile' : 'desktop'}>
        <h1>Your app</h1>
      </header>
    );
  }
}
```

`isMobile` property checks the first axiom as it alters the output, but not the second one because after the component has been rendered the device won't change.

It's better to treat it as a _prop_ in case a parent component holds this value or move it outside the component as a constant value.

```js
import { Component } from 'react';
import MobileDetect from 'mobile-detect';

const ua = window.navigator.userAgent;
const isMobile = new MobileDetect(ua).mobile();

class Header extends Component {
  constructor(props) {
    super(props);
  }

  render() {
    return (
      <header className={ isMobile ? 'mobile' : 'desktop' }>
        <h1>Your app</h1>
      </header>
    );
  }
}
```

The component from above can even be transformed into a functional component and it's now easier to test by just mocking the `userAgent` value.


## Stateful components and dumb components

Applications usually become more complex as they grow, but if we embrace simplicity concepts from its birth it will scale more easily.

It is preferable to have a lot of **stateless** and _dumb_ components that don't know what's happening inside the app, they just receive props and render the output.

The **stateful** ones will hold and pass down the data, but no logic. Logic should be decoupled and come from an external module.

```
business_logic_module <=> stateful_component => stateless_component(s)
```

State mutations are the ones that will trigger updates down the render tree. The less you spread the state of your app, the easier it gets to control them.


### Hints

Using `shouldComponentUpdate` across your project also acts as a great indicator of bad design around components.

If you find yourself writing too much `shouldComponentUpdate` functions or the logic inside of them is too complex, try uprising states to a higher component and pass _props_ to less complex ones.


## Wrap-up

Defering to _stateless_ components over _stateful_ ones will make your project more suitable for testing, less fragmented and more reliable.

Even when they won't make it to the code base, write `shouldComponentUpdate` functions to unveil situations where a separation of concerns is needed.


#### Recommended readings

- [9 things every React.js beginner should know](https://camjackson.net/post/9-things-every-reactjs-beginner-should-know) by Cam Jackson
- [React State](https://medium.com/react-tutorials/react-state-14a6d4f736f5) by Christopher Pitt
- [State and Lifecycle](https://facebook.github.io/react/docs/state-and-lifecycle.html) by React official docs
