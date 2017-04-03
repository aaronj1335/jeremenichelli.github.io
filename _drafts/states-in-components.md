---
layout: default
title: Data, state and components
resume: The versatility provided by class based components usually leads to data misplacing inside a modern web app. This is a short and basic take around components internals regarding data.
---

I will explore different ways React component are able to contain information and how to detect cases where it might be better to move it to a different place.

## Props

By definition _props_ are basically data which will affect the component output, though it is not held by it but by its parent.

**Props** must be first class citizen in your application code base, try to accomplish all possible _flavors_ or variants using them.

By giving _props_ this role, you will favor a unique down data flow direction which will translate in better control over render cycles and performance.


## State

The _state_ of a component is a set of internal properties that also affects the resulting output of its render function.

> “Simplicity is prerequisite for reliability.” - Edsger W. Dijkstra

Here is probably where most of the misconceptions take place. I've seen developers overpopulate a component's state unnecessarily which increases the complexity of the application.

### Should a property belong to the state?

These are three axioms I think a property should pass to be placed inside a state:

- The property must alter the render output.
- The property might change over the component's life cycle.
- The property change is triggered by an interaction inside the component.

If a piece of data doesn't pass these requirements then it should be treated as a _prop_ or even a constant value outside the component.

#### Quick example

Let's imagine a component which returns a different element depending on the detected device.

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

The `isMobile` property checks the first axiom and it alters the output, but it doesn't check the second one because once the component has been rendered there's no way the device will change once the site has loaded.

So it's better to move it outside the component as a constant value.

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

The component from above can be transformed into a function component and it is easier to test now by just mocking the `userAgent` value.


## Stateful components and dumb components

Applications can become complex as they grow, but if we embrace _simplicity_ concepts from its birth it will scale more easily.

To achieve this is preferable to have a lot of **stateless** components that are _dumb_ and don't know what's happening inside the app, they just receive props and render the output.

The **stateful** ones should contain and pass down the date determining new render cycles but no logic. Logic should be decoupled and come from an external module.

```
business_logic_module => stateful_component => stateless_component(s)
```

Changes in states are the ones that end up triggering render cycles, so generally more stateless components means more control.


### Hints

If you find yourself writing a lot of `shouldComponentUpdate` functions across your project to prevent wasted performance, the reason could be that **states** should be removed from _leaf components_ and be treated as **props**.

Another symptom is when the logic inside `shouldComponentUpdate` becomes too big or harder to read. This is usually a sign that the component should _pour down_ data to less complex ones.


## Wrap-up

Defering to _stateless_ components over _stateful_ ones will make your project more suitable for testing, less fragmented and more reliable.

> “The cheapest, fastest, and most reliable components are those that aren’t there.” - Gordon Bell

Even when they won't make it to the code base, write `shouldComponentUpdate` functions to unveil situations where a separation of concerns is needed.

It is good to mention though this is a personal take around states and data, most of the concepts respond to general software design rules and shared views on component based development in web community.
