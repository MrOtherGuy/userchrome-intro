# Introduction to Firefox UI styling

## Preface

This document expects that you already know what userChrome.css file is and how to create one. If you don't however, see [this reddit post for brief start.](https://www.reddit.com/r/FirefoxCSS/comments/73dvty/tutorial_how_to_create_and_livedebug_userchromecss/) Additionally, some basic background knowledge about CSS (Cascading Style Sheet) is required. By some, I mean conceptual understanding of what Document Object Model (DOM) is and how it relates to CSS. There is a whole lot of information about that around the internet, CSS is one of the main web technologies anyway.

Most of the same concepts about CSS as used in web pages to the Firefox UI. However, there are some differences and special cases you should be aware of. These differences are generally not documented anywhere, so the purpose of this document is to provide information about how to apply common CSS knowledge to Firefox UI. Intention is to shed some light about what you can and cannot do with CSS and why. Bindings are generally out of the scope of this introduction, but they have a brief section to explain what they are.

# About XUL

XUL is the language Firefox UI is built in. As far as CSS is concerned it is equivalent to HTML, as in it describes the structure of the document - the DOM tree. HTML is used just about everywhere and you can find information about it everywhere. XUL on the other hand is nowadays pretty much exclusively used in Mozilla software and derivatives of them. Most notably Firefox browser and Thunderbird email client. XUL documentation is thus really scattered at best and even non-existent other than looking at source-code files.

Anyway, since you are presumably interested in *modifying* the UI you'll need information about how the existing layout is built and what styles are applied it by default. And that most certainly is not well documented. It would probably not be worth it anyway because the layout will gradually change as it's updated. Of course, this means that at some point your custom styles will stop working correctly. Changes tend to be quite small though, so usually fixing them won't be too hard. Hopefully you will gain some insight about how to fix them by reading this.

# Basic layout

Here is a super simplified view of the Firefox:

```
window
  |_popups
	|_titlebar
	|_main-UI
    |_menubar
	  |_tabs-toolbar
	  |_navigation-toolbar
	  |_bookmarks-toolbar
  |_content
    |_sidebar
		  |_sidebar-document
	  |_web-content
```

The UI document is basically the whole window that's visible. Most, but not all popups (context menus, dropdowns etc.) are within popups element. The element itself isn't visible but the user's actions trigger the correct popups under it. Titlebar is the one directly on window top-edge. It's exact behavior depends on the user's OS and it's customization is somewhat limited via CSS. Main-UI is what you probably are most interested in, housing all toolbars and buttons and whatnot under it. We'll get to it more later.

Content can be diveded in two portions, sidebar and web_content. Normally all web-pages are loaded as web-content. When you open the sidebar, you are actually creating another area beside the main web page which then *loads some specified document* as sidebar-document. This is somewhat important to know if you happen to stumble on scenario where your css rules won't apply to sidebar content. Hint: it has to do with `@-moz-document url(some-url){ some-rules }` blocks.

# About CSS syntax

See this example for quick remainder of what things represent in CSS files:

```css
/* This is a global declaration block
*  rules inside the block will apply to every element which matches
*  any of the comma separated selectors preceding it.
*  # selects element with the specified ID attribute,
*   - there should only ever be one element with a same ID in one document.
*  . selects a element with that class attribute.
*   - Generally similar elements are grouped into belonging to some class
*  * selects any element
*  If none of tokens [#.*] exist the name selects element with that tag
*   - tag can be thought as the type of the element such as <div> in html.
*  When a name is followed by [], the selector will only match elements which have the specified attribute.
*   - if attribute brackets include = sign then the value of that attribute also needs to match.
*/
#id, .class, tag, *[attribute="value"]{
	padding-right: 42px;
}

/* @-rules limit the scope of the include declaration blocks
*  The included rules will only apply on this specified document
*  browser.xul is the main UI document,
*  but sidebar for example can load other documents
*  Other browser windows, such as library window also are other documents
*  Rules declared in this block won't apply to other documents
*  ::before creates a pseudo-element which will structurally be the first child of the specified element.
*   - The ::after creates the pseudo-element as last child elemet
*   - The styles in this block apply to the pseudo-element
*/
@-moz-document url(chrome://browser/content/browser.xul){
  window::before{
    content: "Hello, World!";
  }
}

/* Combinators are used between selectors to match the latter if it related to the former in some way.
*  " " (whitespace) means the latter is matched if the former is it's ancestor 
*   - ancestor means parent, grand-parent, great-great-great-grand-parent etc.
*   > is the same except the former needs to be direct parent of the latter
*   + select the latter if both share the same parent and the latter becomes directly after the former.
*   ~ same as above but the latter doesn't need to be directly after the former.
*/

.tabbrowser-tab .tab-background,
.tabbrowser-tab > .tab-background,
.tabbrowser-tab + .tab-background,
.tabbrowser-tab ~ .tab-background{
  background: red;
}

/* The list of selectors needs to be separated with a comma
*  If there is no comma then the parser uses the above combinators where newline is treated as whitespace
*   - This probably leads to the selectors not matching anything
*  The list of selectors must NOT end with a comma
*   - This causes parse error and the following block is discarded
*/

#id,
.some > .other
.some + other{
/* latter is parsed as ".some > .other .some + other" which is syntactically correct but probably does not match anything in DOM */
  color:black;  
}

/* properties which begin with "--" are variables
*  - They store some value to the element.
*  - That value can then be used by it or it's child elements
*  When some property is followed by !important it increases that properties priority according to specificity rules.
*/
#id{
  --variable-name: "value";
}
#id > .class::before{
  content: var(--variable-name);
}

/* Typically people begin the userChrome.css file with the namespace line
*   - it is not really needed, but won't cause any harm
*   - It's a global namespace declaration as it doesn't describe any token to refer to the namespace.
*/
/* No token -> global*/
@namespace url("http://www.mozilla.org/keymaster/gatekeeper/there.is.only.xul");

/* This describes the token "html" to refer to html-namespace */
@namespace html url("http://www.w3.org/1999/xhtml");

/* To select elements only in the html namespace you would do this:
html|#id{ color: red }

/*So you could add a token like xul for the xul-namespace
* then you would select each element by prefixing them by xul|
* This won't actually work because Firefox only ever applies the file in xul namespace and declaring other namespaces won't do anything.
* But that's why the line exists, in the distant past it had an effect.
/
```

#
