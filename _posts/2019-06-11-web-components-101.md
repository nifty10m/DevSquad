---
layout: post
title: Web Components 101
---

> Web Components is a suite of different technologies allowing you to create reusable custom elements – with their functionality encapsulated away from the rest of your code – and utilize them in your web apps.
>
> [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/Web_Components)

The three technologies behind web components are

* Custom elements, which lets you define the behavior of your component
* Shadow DOM, which encapsulates your component in a separate DOM tree preventing unwanted collisions
* HTML templates, which lets you define the markup of your component

## How can I use a web component

So with web components you can create custom reusable html elements much like the ones in React or Angular that can be used like this:
```html
<!-- This is just one option to fetch the script -->
<script src="path/to/my-custom-elemnt.js"></script>

<div>
  <my-custom-element></my-custom-element>
</div>
```

## How can I create a web component

```javascript
// template for your element
const template = document.createElement('template');
template.innerHTML = `
  <style>
    button {
      border: 1px solid limegreen;
    }
  </style>
  <template>
    <button>
      <slot></slot>
    </button>
  </template>
`


class MyButton extends HTMLButtonElement {
  constructor() {
    // always call super() first to inherit from the parent class
    super();
  }
  
  connectedCallback() {
    if (!this.shadowRoot) {
      // create a shadow DOM root in open mode (element can be accessed from outside) and attach your template to it
      this.attachShadow({ mode: 'open' });
      this.shadowRoot.appendChild(template.content.cloneNode(true));
    }
  }
}

// this registers a new html tag with its respective class
// Note: the tag name must contain a dash to avoid conflicts with existing elements
// you can think of the first part being a namespace
customElements.define('my-button', MyButton);
```

TODO: elaborate

## Why should I create/use web components?

Web components provide all the benefits of any component based architectures:
* Composability
* Reusability
* Maintainability
* Extensibility
* Productivity

And some additional benefits because they are used in the web:
* Scoping
* Interoperability (framework agnostic)
* Accessibility (by extending default HTML elements)

Interoperability is especially important, since this allows you easy brand consistency. Create your corporate components library or design system as web components
and all your teams can work off the same base no matter which framework they use. This will also directly reflect in higher development speed because
the components don't have to be implemented for each framework over and over again.
Lastly web components is a browser standard added by the W3C which means it uses core features of each browser.

However, there are certain downsides to web components that justify the coexistence of the known web frameworks.

## What can't I do with web components?

Web components have inputs and outputs (events) much like any other framework but primarily the inputs are pretty limited for they only
accept strings. Which means you can pass in types that can be cast to string or can be deserialized from JSON at most.
Also reacting to changes of input values is rather tedious to setup since you have to create a separate listener for each attribute you want to react on.

All in all I would recommend using web components for simple leaf nodes (e.g. buttons, inputs, etc.) or certain layout components (e.g. a bootstrap grid system).

{% include twitter.html %}
