# About CSS

CSS is basically a collection of styling rules that browser applies to some defined elements. Lot's of information is found all over the web about this subject since it's the styling language that web pages use. Good thorough guide can be found on [MDN](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS), but well go shortly through main concepts.

## Application

CSS itself doesn't do anything. It always needs some document to apply to. Basic HTML document is a *tree* of elements (terms "node" and "element" can be considered as meaning the same thing for our purposes). That tree has some structure like this:

```
html
|_head
|_body
  |_menu
  |_content
  | |_main-content
  | | |_doc
  | |_extra-content
  |_footer
```

Html is the root of the document and it has few *child* elements - head and body. In HTML, head elements is invisible, it just has some meta information like included scripts and language etc.

Body has child elements of it's own of which content again has it's own childs. This is a simple tree - the *structure* of the document.

# Syntax
## Rulesets

Conceptually, CSS files are a list of rulesets. A ruleset is made of two parts:

1. A list of selectors
2. A list of rules

Example:

```css
body, main-content{
  visibility: visible;
}
```
body and main-content make up a comma-separated list of selectors. They are used to select what elements the rules inside the following declaration block are applied to. Declaration block is must begin and end with curly braces.

## Selectors

Elements in the document can have certain properties which CSS can use to select those elements. Those are tag name, id, class and attributes.

```css
/* Comment blocks begin and end with asterisk-slash sequence */
tagname, /* line-breaks are ignored so you can make code more readable */
#id,
.class,
#id[attribute]{
  ...
  <rules>
  ...
}
```

* Tag name (no prefix) is basically a type of that element, such as button.
* ID (prefix: #) There should only ever be one element with some unique ID
* Class (prefix: .) Classes are used to group some elements that should have similar properties with each other.
* Attributes (#id[attribute]) Here, element with ID "id" is only selected when it has an attribute named "attribute".
* Universal selector ( * single asterisk) selects every element.

You should note here that CSS can **not** set or modify these properties. They are set when the document is loaded, but can be modified with the browser engine at runtime for example to set different states for the document. But there is no way for CSS to set those states.

### Combinators

Selectors can be combined to more carefully select which elements the rules are intended for. They care where within the **document-tree** two or more elements are. Their painted position (where you actually see them) is irrelevant.


Descendant combinator (single space in between selectors) matches elements with class "class" that are somewhere inside #id element.

```css
#id .class {...}
```

Child combinator ( > ) matches elements with .class that have a direct parent element with #id

```css
#id > .class {...}
```

General sibling ( ~ ) mathes button elements that come after .class elements and both share the same direct parent. 

```css
.class ~ button {...}
```

Adjacent sibling ( + ) is like general sibling but the second element must be directly after the first one.

```css
.class + button {...}
```

You can also use the following combination to select #id element but only when it has a class "class"

```
#id.class {...}
```

### Declarations

Declarations are made of four parts.

* Property name followed by a single colon character;
* Property value
* Optional `!important` tag
* Single semicolon to end the declaration

There is a whole lot of property names, you should use [MDN reference](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference) to see what they do.

Properties that begin with a dash are vendor prefixed properties. They act like other properties, but are non-standard as they might behave differently across browsers. Browser vendors typically implement new CSS-features first as a browser-prefixed version before there is a an agreed specification for the feature.

```css
#main{
 -moz-border-radius: 2px;
}
```

Properties that begin with a double-dash set a value for some named variable. These variables are visible to all the descendant of the element where they are set and can be referred to with var() function.

Here we set --text-color variable to rgb(0,0,0) (aka black). The second ruleset will always work because the child combinator makes sure that we always select elements that see this variable. However, the third ruleset selects all .content-item elements but some of them might not be descendants of #content. The ones that are will use the --text-color variable but others default to whatever the browser defaults to.

```css
#content{
  --text-color: rgb(0,0,0);
}
#content > .content-item{
  color: var(--text-color) !important;
}
.content-item{
  color: var(--text-color) !important;
}
```

### Pseudo-elements

You can create elements with pseudo-element syntax.

```css
#main::after,
#main::before{
  content: "pseudo";
  color: red;
}
```
This creates something that behaves mostly like an element as a first and last element of #main. Both just show a red text "pseudo". Pseudo-elements are useful when you need to create something. Even if it has limitations, it's the only way you can create new content with CSS.

### Pseudo-classes

Pseudo-classes are kind of attributes that are set and cleared in reaction to user-actions. Here are few useful examples:

Cursor on top of #main element

```css
  #main:hover{...}
```

#main has focus. Not all elements can normally have focus, but things like input boxes can.

```css
  #main:focus{...}
```

Some descendant of #main has focus. Rules are applied to #main, not the element that actually has focus.

```css
  #main:focus-within{...}
```

Select the root document, basically the outermost element for which everything is else is descendant to.

```css
  :root{...}
```

Root is especially useful when you want to for example set variables that are available for all the elements in the document.

### Shorthand properties

Some properties are known as shorthands. They take variable number of values which the browser maps to the "real" properties. Example:

```
#main{
  margin: 2px 0;
  padding: 1px 2% 3em 0;
  background: red url(<image_path_here>);
}
```

This is same as:

```
#main{
  margin-top: 2px;
	margin-right: 0;
	margin-bottom: 2px;
	margin-left: 0;
	padding-top: 1px;
	padding-right: 2%;
	padding-bottom: 3em;
	padding-left: 0;
	background-color: red;
	background-image: url(<image_path_here>)
}
```

Notice also the value type indicators.

* px: absolute pixels
* %: proportional to the size of the element
* em: proportional to the text size of the element
* 0 is a special value that doesn't need indicator because 0 of anything is just 0px;

# Specificity

Single CSS file can have hundreds of rulesets. Often, there will be many rulesets that would apply to any single element creating a scenario where some properties are defined differently in many rulesets. A property may only have one value at the time so there must be some way to select which one should be used on such conflicts.

The basic priority in descending order:

1. Declaration has "!important" tag
2. Uses inline style (elements may have "style" attribute)
3. Selector uses ID (#)
4. Selector uses class, attribute or pseudo-class
5. Selector uses tag name
6. Selector uses universal selector *
7. Declaration becomes later in the css file

Example consider a following element:

`<div id="main" class="thing" attr1 attr2 attr3 />`

```css
#main{
  color:green;
  font-size: 24px;
}
.thing[attr1][attr2][attr3]{
  color: red;
  opacity: 0.9;
  font-size: 19px !important;
}
#main{
  color:blue;
}
```
Computed style will be { color: blue ; opacity: 0.9; font-size: 19px }
Even if there are four tokens belonging to category 4 the single category 3 token (ID selector) will override it. color:blue takes priority over color:green since it comes after it. There is no conflicts on opacity so that will apply normally. For font-size there exists !important tag in the second ruleset though, so that will take priority.

[See MDN for more thorough information](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Cascade_and_inheritance)

# Box model

Eventually the browser will create a "box" for each element which will be it's bounding box when rendered to screen. The type of that box is controlled by CSS "display"-property. In HTML the elements' default value is dependent on the tag-name of the element. The type affects how the element behaves related to available space and size of it's contents (think child elements).

## Box type

Some common values for display are "block, inline, flex, none". "Block" and "inline" create basic boxes. You can read the about the actual differences [here on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Visual_formatting_model). "Flex" is more modern addition, which automatically scales and distributes available space and can be controlled by additional control properties. [More about flexbox model.](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout). "None", well no box will be created for this element, ie. it becomes invisible.

## Dimensions

Each block will have few properties which affects it's eventual size and position.

### Margins

How much space the element reserves *outside* of it's box. Negative margins reserve negative space, and that generally results in the element being moved up or left for top and left margins. For right and bottom margins the adjacent content may be moved on top the element with negative margins. 

### Paddings

Similar to margins, but describes how much space the element reserves *inside* it's box. Conceptually you can think this as equal to the elements children having margin. Negative padding doesn't exist though.

### Borders

Between padding and margin.

### Width / height

Elements may scale depending on how much space it's contents need. There are three properties for width and height which control this behavior.

`width` If the elements' width is not constrained by any external or internal elements then this is the width that the browser will use. Like when the browser window is really wide and there is "empty" space on both sides of content. In such case you can set width to some value and the element will use that even if it does not need that much space.

`min-width` This is the minimum width of the element. If the element would be naturally scaled below this value by external pressure then this will restrict the distribution of space and any further scaling will either affect other siblings proportionally more or create empty space inside the element or make the parent element overflow.

`max-width` If there is empty space around the element that could be used then this element only uses the maximum of this width. Or this element will overflow at this width if its children would need more space than this.

Concepts are same for height. Width/height values are hard to explain clearly. But the main point is that max-* and min-*  values create additional restrictions for natural scaling. Width/height just set a base point which will be used when no restrictions from min-* or max-* or other elements apply. [See more about box model on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model)

That's it. Well, obviously there is a lot more to it than that, but that is CSS syntax and matching rules in short form. [Next up we'll be looking at how CSS differs between HTML and XUL](userChrome_intro_3_xulcss.md)