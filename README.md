A Simple Mind's Web Components Overview
========================================

An over-simplified, basic definition of what [Web Components](http://webcomponents.org) are:

* [Custom Elements](http://w3c.github.io/webcomponents/spec/custom/) (node names containing a dash)
* [Shadow DOM](http://w3c.github.io/webcomponents/spec/shadow/)
* [HTML Templates](http://www.w3.org/TR/html5/scripting-1.html#the-template-element) (using the `<template>` tag)
* [HTML Imports](http://w3c.github.io/webcomponents/spec/imports/)
* [Best Practices](http://webcomponents.org/articles/web-components-best-practices/)
    
Browser's native support for each of these things varies, but polyfills help. 

Is it a toy? No, its a versatile interlocking brick system for the web. 

    
Custom Elements
----------------

If you don't want to read the spec, they are essentially nodes with names that contain a dash:

* `<super-duper-app>` (valid)
* `<superduperapp>` (invalid)
* `<toggle-button>` (valid)
* `<toggleButton>` (invalid)
    
Before your custom elements are registered using document.registerElement they will be :unresolved and receive that style.
Unknown elements will not be styled with the :unresolved pseudoclass. Invalid elements will be treated as unknown. Try the following style and HTML:

```css
/* Style Unresolved (as opposed to Unknown) elements */
:unresolved { border: 5px dashed red; display: block; margin: 2em; }
```

```html
<toggle-button>Toggle-Button</toggle-button>
<toggleButton>ToggleButton</toggleButton>
```

When you register your element you are able to define the prototype, which includes some event callbacks, and also define 
whether it extends another element. The prototype that the custom element inherits from defaults to HTMLElement, but can be virtualy any element.

* createdCallback = { property: value OR f(x) }           _an instance of the element is created_
* attachedCallback = { property: value OR f(x) }          _an instance was inserted into the document_
* detachedCallback = { property: value OR f(x) }          _an instance was removed from the document_
* attributeChangedCallback(attrName, oldVal, newVal)      _an attribute was added, removed, or updated_

```js
// Custom Element Registration with document.registerElement
document.registerElement('toggle-button', {
    prototype: Object.create(HTMLElement.prototype, {
      createdCallback: {
        // This would normally set the value attribute of the custom element
        value: function() {
            this.innerHTML = '<a class="btn">I am Groot!</a>';  
        }
      }
    })
});
```

Shadow DOM
-----------

Who knows what evil lurks in the hearts of custom elements? The Shadow DOM knows! In this case, that's a good thing though. 
The Shadow DOM can utilize the `<content>` tag to drop the innards of the web component into your element.

```html
<welcome-msg>Star Lord</welcome-msg>
```

```js
// Custom Element Registration with Shadow DOM 
document.registerElement('welcome-msg', {
    prototype: Object.create(HTMLElement.prototype, {
        createdCallback: {
            value: function() {
                // Adds the #shadow-root to the element
                var root = this.createShadowRoot();
                // <content> represents the non-Shadow innards of the element
                root.innerHTML = 'Welcome to MMMBOP, <content></content>!';
            }
        }
    })
});
```

HTML Templates
---------------

Utilizing the `<template>` tag for a custom element enables you to do a little separation of concerns.

```html
<cool-menu>
    <ul>
        <li>First Item</li>
        <li>Second Item</li>
    </ul>
</cool-menu>

<!-- A template to be used by a custom element -->
<template id="menu-template">
    <h1>Menu</h1>
    <content></content>
</template>
```

```js
// Custom Element Registration using a <template>
document.registerElement('cool-menu', {
    prototype: Object.create(HTMLElement.prototype, {
        createdCallback: {
            value: function() {
                // The <template> tag is not visible in the browser, but can still be used
                var t = document.querySelector('#menu-template');
                var clone = document.importNode(t.content, true);
                this.createShadowRoot().appendChild(clone);
            }
        }
    })
});
```

HTML Imports
-------------

Use `<link rel="import" href="">` to pull in the component dependencies and have even more separation.

```html
<link rel="import" href="menu-item.html">
<style> a { text-decoration: none; font-weight: bold; }</style>
<cool-menu>
    <ul>
        <menu-item>First Menu Item</menu-item>
        <menu-item>Second Menu Item</menu-item>
    </ul>
</cool-menu>
```

menu-item.html
```html
<!-- Everything is completely encapsulated in the Shadow DOM -->
<template id="menu-item-template">
   <style>
       a { color: orange; }
    </style>
    <a href="#" class="underline"><content></content></a>
</template>
```
```js
// There may be a better way to reference the included document versus the including document
// I couldn't get it to work inline with the registration though, so this is how I did it
var thisDoc = document.currentScript.ownerDocument;
var thatDoc = document;
    
// Register the element to the including page
thatDoc.registerElement('menu-item', {
    prototype: Object.create(HTMLElement.prototype, {
      createdCallback: {
        value: function() {
            // Find the template within this document (and not the including document)
            var t = thisDoc.querySelector('#menu-item-template');
            var clone = thatDoc.importNode(t.content, true);
            this.createShadowRoot().appendChild(clone);
        }
      }
    })
});
```