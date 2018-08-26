# XUL CSS extensions

## What is XUL?

XUL (XML User interface Language) is a markup language like HTML and XML. Conceptually it works the same as HTML in web pages, as in it describes the document structure and loads CSS to style the elements. However, XUL doesn't have any formal specification and since it's not widely used (like HTML is) the documentation about its exact behavior is sparse and scattered - at best.

Firefox is probably the single most used software that uses XUL, so the information in this guide is written for that in mind. Other software could have a slightly different implementation or at least different default styling rules, but this series is about userChrome modifications for Firefox so we'll only consider the implementation in Firefox.

Additionally, we'll limit the scope of this XUL guide to bits that are relevant to CSS.

# XUL layout

Most of the normal HTML+CSS concepts apply just as well to XUL+CSS. There are, however certain additional properties and property values that you can use. But also some properties which don't work or will work differently.

## Box model

The default box type (CSS "display") for all xul elements is "-moz-box". This can be thought as very similar to flexbox in HTML as in it will automatically scale to use available space. The relative flexibility of these elements can be controlled by -moz-box-flex: property (similar to flex-grow in HTML). There is an exception to this though; -moz-box-flex is totally ignored if the element in question has "flex" attribute with some value. The value of flex is just some number like "100" or "50". Example:

```css
```