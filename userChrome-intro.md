# Introduction to Firefox UI styling

## Preface

This document expects that you already know what userChrome.css file is and how to create one. If you don't however, see [this reddit post for brief start.](https://www.reddit.com/r/FirefoxCSS/comments/73dvty/tutorial_how_to_create_and_livedebug_userchromecss/) [Brower Toolbox](https://developer.mozilla.org/en-US/docs/Tools/Browser_Toolbox) is also a very valuable tool to know. Additionally, some basic background knowledge about CSS (Cascading Style Sheet) is required. By some, I mean conceptual understanding of what Document Object Model (DOM) is and how it relates to CSS. There is a whole lot of information about that around the internet, CSS is one of the main web technologies anyway.

Most of the same concepts about CSS as used in web pages to the Firefox UI. However, there are some differences and special cases you should be aware of. These differences are generally not documented anywhere, so the purpose of this document is to provide information about how to apply common CSS knowledge to Firefox UI. Intention is to shed some light about what you can and cannot do with CSS and why. Bindings are generally out of the scope of this introduction, but they have a brief section to explain what they are.

# About XUL

XUL is the language Firefox UI is built in. As far as CSS is concerned it is equivalent to HTML, as in it describes the structure of the document - the DOM tree. HTML is used just about everywhere and you can find information about it everywhere. XUL on the other hand is nowadays pretty much exclusively used in Mozilla software and derivatives of them. Most notably Firefox browser and Thunderbird email client. XUL documentation is thus really scattered at best and even non-existent other than looking at source-code files.

Anyway, since you are presumably interested in *modifying* the UI you'll need information about how the existing layout is built and what styles are applied it by default. And that most certainly is not well documented. It would probably not be worth it anyway because the layout will gradually change as it's updated. Of course, this means that at some point your custom styles will stop working correctly. Changes tend to be quite small though, so usually fixing them won't be too hard. Hopefully you will gain some insight about how to fix them by reading this.


# About CSS

Few main ideas here about capabilities of CSS:

* You cannot add new content with it
* You cannot add features with it (nor add attributes to elements)
* You cannot change the document structure

CSS can only change how the documet	*looks*. Adding content and to very limited extent features is actually possible with ::before and ::after pseudo-elements but that's is seriously restrictive.

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

# Basic layout

Firefox UI (aka chrome) document is basically the whole window that's visible. Most, but not all popups (context menus, dropdowns etc.) are within popups element. The element itself isn't visible but the user's actions trigger the correct popups under it. Titlebar is the one directly on window top-edge. It's exact behavior depends on the user's OS and it's customization is somewhat limited via CSS. Main-UI is what you probably are most interested in, housing all toolbars and buttons and whatnot under it. We'll get to it more later.

Content can be diveded in two portions, sidebar and web_content. Normally all web-pages are loaded as web-content. When you open the sidebar, you are actually creating another area beside the main web page which then *loads some specified document* as sidebar-document. This is somewhat important to know if you happen to stumble on scenario where your css rules won't apply to sidebar content. Hint: it has to do with `@-moz-document url(some-url){ some-rules }` blocks.

Browser toolbox let's you explore the UI structure. But finding what you are looking for may not always be so easy. Generally, the layout goes from top-to-bottom and left-to-right (or right-to-left depending on your locale). Basically elements closer to the top of the visible window will be earlier in the document. And when the parent element is layed horizontally, then leftmost will be first - assuming no CSS modifications. This is important when you try to style certain parts based on some state.

Suppose the case where you want bookmarks toolbar to be visible only when focus is inside urlbar. Since urlbar is before bookmarks toolbar in the structure you can do this:
```css
#PersonalToolbar{ visibility: collapse } /* default state */
#urlbar[focused] ~ #PersonalToolbar{ visibility: visibile }
```
If bookmarks toolbar would be before urlbar then you could not do this. Remember, information of states in CSS only ever flows forward because you can only use combinator selectors [" ",">","+","~"]. There is no way to tell elements earlier in the structure about things coming later.

Strictly speaking, that example isn't correct because #urlbar and #PersonalToolbar are not children of the same parent but it works as easy to understand example.

Earlier and later here refer to either being higher up in DOM tree and being later child of same parent element. You should ponder about this until you really get it as it's one of the more important things here. Maybe this example will help you a bit.

Here is a super simplified view of the Firefox:
```
window
  |_popups
  |_titlebar
  | |_window-controls
  |_main-UI
  | |_menubar
  | |_tabs-toolbar
  | |_navigation-toolbar
  | |_bookmarks-toolbar
  |_content
    |_sidebar
    | |_sidebar-document
    |_web-content
```
Here menubar can see the state of main-UI, titlebar, popups and window. It cannot gain information about anything else. tabs-toolbar would have that same information but it can also know about the state of menubar. Also note that parent-element won't know about the state of it's children. So main_UI won't know about menubar state.

## Firefox UI properties

When Firefox starts it loads it's main UI document. The document structure (DOM-tree) generally doesn't change at run time, elements just may or may not be visible. There are only some important exceptions:

* New tabs create new elements to tabs toolbar
* Customizing toolbar via customize adds, removes or moves elements
* Sidebar may load other sub-document
* Populating urlbar dropdown list

Firefox' javascript engine manages the internal state of the UI by editing attributes of elements. CSS rules then use those attributes to apply styles to UI. Remember `#id[attribute]` selectors? That's really what's going on. It's not too much of a simplification even.

## Default style

The document nowadays uses css variables (properties that start with "--") quite heavily. The value of CSS variables is also visible to child elements of the node so if some variable is defined in window level it will be visible to every element. Indeed, many variables are defined in such way globally which has a nice effect that you can change the value in one place and the change will be reflected to every element using that one variable.

That's why you will see snippets like this:

```css
:root{
  --toolbarbutton-inner-padding: 6px;
}
```
:root is a special selector to refer to current document root element, in main UI document this is the same as `#main-window`. This enables a really nice and simple way to edit properties for all affected elements. Unfortunately there is a perhaps less than obvious problem with this kind of modification. If the default style is changed so that the elements use that same variable name anymore, then your modified rule is not applied anywhere where you intend to. That is to say, variables don't apply any property themselves. Elements need to use that value. Also check this case:

```css
:root{
  --toolbarbutton-inner-padding: 6px;
}
#navigator-toolbox{
  --toolbarbutton-inner-padding: 4px;
}
#navigator-toolbox > *{
  padding: var(--toolbarbutton-inner-padding); /* will be 4px */
}
```
#navigator-toolbox is a child element of :root. Child elements can override the value of higher level variable for themselves and their children. This means that if default stylsheet moves some variable declaration to a more "specific" element it will override the one declared in :root. The one on :root will still effect other elements, just not any under #navigator-toolbox.

UI uses `display: -moz-box` for almost all elements. -moz-box is rather similar to html flexbox, but there are some differences especially on how it reacts to certain attributes. There is nothing stopping you from using other display values like block, flex and even grid. Exception to this is display:contents which is treated as being equal to display:none.

The UI uses heavily anonymous content. Browser toolbox label such elements beginning with "xul:". From CSS perspective this has following kind of effect:

```
<textbox id="urlbar">
  |_<xul:hbox>
    |_<box id="identity-box">
  	|_<xul:image>
  	|_<hbox id="page-action-buttons">
```
Now, urlbar has an anonymous element hbox which can be selected normally by `#urlbar > hbox`. However, the non-anonymous element don't see the anonymous ones so `hbox > #identity-box` will fail. Same for page actions. But `hbox > image` works just fine. Infact to use sibling combinators you would in this case need to do #urlbar > #identity-box even if there is one element in between. On some structures there will actually be many layers of anonymous content.

# Element descriptions

I'll be describing various elements separately in this chapter.