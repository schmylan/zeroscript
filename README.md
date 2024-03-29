# ZeroScript (RFC)

Build for the web using zero JavaScript.

Basically, ZeroScript is just vanilla HTML with a few extra tags. Its purpose is to reduce the web's dependency on JavaScript by opening the door for other languages to compete.

Making JavaScript interchangeable is unrealistic unless other languages can both _generate HTML_ **AND** _manipulate the DOM_ using one common approach. The idea is to use an HTML-first strategy. Instead of putting HTML inside your logic, ZeroScript puts your logic inside your HTML.

In many ways, ZeroScript is like Markdown. It's not an implementation but rather a small set of rules for outputting predictable results for the web regardless of language choice. Like Markdown, there's also room for various "flavors" to bring their own special embellishments.

ZeroScript's purpose is grounded in the desire for the web to remain **THE** melting pot of human ideas and progress. The best way to prevent stagnation is to open the floodgates for other languages to compete.

> [!IMPORTANT]
> The examples in this README use a fictional language called `Any++` in order to provide concrete examples without any favoritism. Conceptually, any imperative language could be substituted.

### Table of Contents

- [Components](#components)
- [Attributes](#attributes)
- [Children](#children)
- [Control Flow](#control-flow)
- [Styles](#styles)
- [Routing](#routing)
- [Bring Your Own Language (BYOL)](#byol-bring-your-own-language)
- [State Management](#state-management)
- [Ecosystem](#ecosystem)

<br />
<br />

## Components

### File-Based

Every `.html` file is automatically a component. No need to register or import anything. Just use its file name. File names are limited to alpha-numerics, hyphens, underscores, and cannot begin with a number.

<table>
<tr>
<td>
my-button.html
</td>
<td>
index.html
</td>
</tr>
<tr>
<td>

```xml
<button>        
  Click me
</button>
```

</td>
<td>

```xml
<html>        
  <body>
    <my-button />
  </body>
</html>
```

</td>
</tr>
</table>

> [!NOTE]
> Markdown files (`.md`) make great components too.

### Naming Collisions

HTML has a spirit of being forgiving, therefore naming collisions must not result in an error. Since two files can have the same name if located in different directories precedence is determined as follows:

1. Prefer the file that is in the same directory as the referencing component.
2. Prefer any files that are in a descendant directories over any files located in ancestor paths (i.e. great-great-grandchildren before cousins).
3. Next, prefer the file with a shorter directory depth
4. Finally, prefer the file whose path sorts alphanumerically first.

To explicitly reference components with naming collisions:

- Use `/` notation + the names of directories. (Capitalization matters.)
- Use `//` as shorthand notation to skip any number of directories
- There is an implied `//` at the beginning of every component

```xml
📦 my-project
└─ 📂 ui
   ├─ 📜 index.html
   ├─ 📂 music
   │  └─ 📂 components
   │     └─ 📜 score.html  1️⃣
   └─ 📂 sports
      ├─ 📂 components
      │  └─ 📜 score.html  2️⃣
      └─ 📂 music
         └─ 📜 score.html  3️⃣

index.html
<html>
  <body>
1️⃣   <score />
2️⃣   <sports//score />
3️⃣   <music/score />
  </body>
</html>
```

<br />
<br />

## Attributes

### Inputs

HTML attributes are an easy way to pass inputs into a component thus boosting reusability.

You'll notice the use of the `{{ }}` escape sequence in the component file. You'll learn more about this in the [Hole Punch](#hole-punch) section below.

<table>
<tr>
<td>
index.html
</td>
<td>
my-button.html
</td>
</tr>
<tr>
<td>

```xml
<html>                         
  <body>
    <my-button name="Rylan" />
  </body>
</html>
```

</td>
<td>

```xml
<button>                
  Hello {{name}}
</button>
```

</td>
</tr>
</table>

> [!NOTE]
> A component's attributes are optional by default. The ability to denote some attributes as "required" is outside the scope of this document but is a highly recommended "flavor" to be provided by the guest language.

### Strongly Typed

In your attributes, when using data types other than strings, use JSON-like syntax where the double-quotes are excluded. Non-primitives, such as objects and arrays, must be handled inside a hole-punch `{{ }}` so that the specific syntax can vary by language.

```xml
index.html

<html>
  <body>
    <my-button text="Hello" count=1 weight=3.14 cta=true time={{date}} />
  </body>
</html>
```

### Sibling Files

Any file in the same directory with the same filename but different extension must be conceptually treated as a part of the same component. Rules for handling may vary according to file type.

For example, including a `.css` file makes it trivial to include styles with your component. However, should that component ever repeat its HTML, its CSS would only ever be included once.

Sibling files can also make use of hole punches `{{ }}`.

<table>
<tr>
<td>
File Structure
</td>
<td>
index.html
</td>
<td>
Output
</td>
</tr>
<tr>
<td>

- website
  - index.html
  - component.html
  - component.css

</td>
<td>

```xml
<html>               
  <body>
    <component />
    <component />
    <component />
  </body>
</html>
```

</td>
<td>

```xml
<html>                             
  <body>
    <style>
      ...CSS content...
    </style>
    <div>
      ...HTML content...
    </div>
    <div>
      ...HTML content (repeated)...
    </div>
    <div>
      ...HTML content (repeated)...
    </div>
  </body>
</html>
```

</td>
</tr>
</table>

> [!NOTE]
> While sibling files can technically work with `.js` files too, that's not the recommended approach. This might be useful if you are certain your website will only ever be statically generated. But it's important to note that ZeroScript has a different language-agnostic approach for building dynamic features that offers a gradual migration path from a fully static website to a fully dynamic web app. You'll learn more about this in the [BYOL](#🤖-byol-bring-your-own-language) section below.

<br />
<br />

## Children

### Content

To simplify composability, any child-tags to a component can be accessed using `{{content}}`. This is intentionally similar to attribute-usage. This makes `content` a reserved keyword and therefore cannot be used as a component's attribute.

<table>
<tr>
<td>
index.html
</td>
<td>
my-button.html
</td>
</tr>
<tr>
<td>

```xml
<html>                          
  <body>
    <my-button>
      <b>Please</b> click me
    </my-button>
  </body>
</html>
```

</td>
<td>

```xml
<button>            
  {{content}}
</button>
```

</td>
</tr>
</table>

### Slots

It is possible to provide outside HTML as inputs to a component. However this isn't done through its `attribute`s but using `slot`s instead. Similar to Web Components, child-tags can be used as HTML-based inputs using the `slot` keyword.

Any tags without a `slot` attribute must default to the reserved slot `content`. Slot order does not matter and any missing slots are treated as `null`.

<table>
<tr>
<td>
index.html
</td>
<td>
my-button.html
</td>
</tr>
<tr>
<td>

```xml
<html>                                          
  <body>
    <my-button>
      <img slot="icon" src="..." />
      <span slot="text">
        I am <i>more</i> than a <b>string</b>.
      </span>
    </my-button>
  </body>
</html>
```

</td>
<td>

```xml
<button>                    
  {{icon}} • {{text}}
</button>
```

</td>
</tr>
</table>

> [!NOTE]
> Slots are optional and if not supplied, the component must still function. Similar to attributes, though, it possible and recommended for a guest language to support "required inputs" much like a class with a non-nullable property.

<br />
<br />

## Control Flow

ZeroScript uses a few tags for control flow: `<if>`, `<else>`, `<else-if>` and `<foreach>`. These are reserved keywords so custom components by those names are not allowed. While the content inside these tags might be included, excluded or repeated, the enclosing tag itself is never included in the generated HTML. (The browser wouldn't know what to do with an `<if>` tag anyway right?)

Most other frameworks choose to use their natural syntax instead of extending HTML to handle control flow. For them, this is a sensible choice since it allows for greater flexibility. Since the purpose of ZeroScript is to be language-agnostic, it takes a more generic approach by simply extending HTML and to lean into its [hole-punching](#hole-punch) approach for extending functionality.

### Conditions

The `<if>` tag conditionally excludes its content. It uses a valueless attribute as the condition for terseness.

```xml
<if {{user == null}}>
  <sign-in-form />
</if>
```

Use `<else>` and `<else-if>` tags in conjunction with `<if>`. These are sibling tags and do not require a parent container as you see with `<ul>` and `<li>`.

```xml
<if {{loginState == .LOGGED_OUT}}>
  <sign-in-form />
</if>
<else-if {{loginState == .LOGGED_IN}}>
  <sign-out-button />
</else-if>
<else>
  <circular-progress />
</else>
```

### Pattern Matching

The `if-let` syntax lets you handle your dynamic content in a less verbose way. If the expression evaluates to `null`, all child content is excluded from the generated HTML. Otherwise, the compiler can safely assume non-nullability for the provided attribute name.

In this example, if any of `user`, `profile`, `name`, or `first` are null then the entire child contents are skipped, including the "Welcome back " text.

```xml
<if firstName={{user?.profile?.name?.first}}>
  Welcome back {{firstName}}.
</if>
```

Multiple null-checks are allowed as well as chaining with `else` and `else-if`.

```xml
<if first={{user?.firstName}} last={{user?.lastName}}>
  Welcome back {{first}} {{last}}.
</if>
<else>
  <h2>Loading...</h2>
</else>
```

### Loops

Use a `<foreach>` tag to repeat its contents in a declarative way.

```xml
<ul>
  <foreach fruit in={{allFruits}}>
    <li>{{fruit}}</li>
  </foreach>
</ul>
```

<br />
<br />

## Styles

### Platform and File-Based

CSS can be used normally, including embedded `<style>` tags and externally referenced `.css` files.

However, any `.css` file that shares the same filename and parent directory as an `.html` file is considered a [sibling file](#sibling-files-1) and must automatically have its styles included before any HTML to prevent any dreaded [FOUC](https://en.wikipedia.org/wiki/Flash_of_unstyled_content). Inclusion must occur only once per HTML document, since its `.html` counterpart might be repeated multiple times.

### Scoped Styles

Component-level styles must be scoped to its component. It is not required to use a Shadow DOM like Web Components do.

<table>
<tr>
<td>
my-button.css
</td>
<td>
Output
</td>
</tr>
<tr>
<td>

```css
p {                    
  font-size: 18pt;
}
em {
  color: orange;
}
```

</td>
<td>

```html
<html>
  <body>
    <style>
      .my-button p {
        font-size: 18pt;
      }
      .my-button em {
        color: orange;
      }
    </style>
    <button>
      <p>This button has <em>style</em>!</p>
    </button>
  </body>
</html>
                                             
```

</td>
</tr>
</table>

<br />
<br />

## Routing

### File-Based

ZeroScript uses file-based routing as a language-agnostic way to define your URL routing patterns. ZeroScript does encourage any "flavor" to embellish additional features in their own native language syntax.

### Files

The only files publicly accessible by an HTTP route are `index.html` files. This allows you to organize your components in any location without the concern for them being accessed directly.

### Directories

The name of each directory represents a route segment for the URL. For example, if there were an `index.html` file located at `myproject/blog/about/index.html` it could be accessed at `https://example.com/blog/about`. (Trailing slashes are intentionally excluded.)

If a directory does not contain an `index.html` file, its corresponding HTTP route results in a 404.

### Dynamic Routes

A dynamic route segment can be used by wrapping the directory name in square brackets. Accessing the values of the dynamic routes will vary by language.

| File System               | HTTP                          |
| ------------------------- | ----------------------------- |
| `myproject/blog/[slug]`   | `example.com/blog/first-post` |
| `myproject/products/[id]` | `example.com/products/1234`   |

> [!Note]
> Directory-based routing is intentionally designed to only handle very basic use cases. It's encouraged that each guest language support more sophisticated routing in their own unique ways that cater to their each of their strengths.
>
> For example query string parameters could be accessed from from a request object like `{{req.slug}}` or like `{{request.Routes["slug"]}}`, whatever fits most naturally in to the language.
>
> The same non-prescriptive stance applies for supporting HTTP methods beyond simple `GET`s such as `POST`.

### Errors

Displaying errors can be handled with a file by the name of the HTTP status code. For example the output of `404.html` would be used for any not-found routes. Use `*` after the first digit as a wildcard. For example, `5**.html` would cover all server errors, negating the need to create a unique file for each error code. Using both `404.html` and `4**.html` is valid and the most specific pattern should be used.

<br />
<br />

## BYOL (Bring-Your-Own-Language)

### Migration Path from Static to Dynamic

Astute readers might notice that, at this point, we now have all the tools necessary to host or generate a static website composed of reusable components using only `.html` and `.css` files and nothing else.

This is a good thing. It ties the durability of your frontend to the longevity of the web itself. The more you can "embrace the platform" the less susceptible your codebase is to [software rot](https://en.wikipedia.org/wiki/Software_rot). It shouldn't be an unreasonable expectation to return to a 10 year old project and have everything function exactly as before.

The sections that follow cover a common approach for progressively enhancing your website using your language of choice.

### Event Handlers

Use the DOM's regular events for event handling but instead of specifying JavaScript inside a string, reference your own method using a hole-punch escape sequence `{{ }}`. This method can optionally choose to take the event as an argument.

```xml
<button onclick={{handleClick}}>
  Click me
</button>
```

Using closures can be a valid option too, if your language supports it.

```xml
<button onclick={{e => count++}}>
  Clicks: {{count}}
</button>
```

### `<script>` Tags

In the spirit of "embracing the platform" you can repurpose the `<script>` tag for use by any language using the ([to-spec](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#language)) `language` attribute.

When using the `<script>` tag for any language besides JavaScript, be sure to never include it as a part of any generated HTML. (The browser wouldn't know how to execute it anyway right?) Execution can be handled server-side or client-side via WebAssembly. More on that in the [Server-Side or Client-Side](#server-side-or-client-side) section.

```xml
my-button.html

<button onClick={{handleClick}}>
  Clicks: {{count}}
</button>

<script lang="any++">
  count = 0

  void handleClick(event) {
    count++
  }
</script>
```

> [!TIP]
> Whatever value is specified in the `lang` attribute is technically moot since it's never included in the HTML. Therefore, whatever value is specified is more for human consumption than for the machine.

### Sibling Files

The sibling file approach works great for other languages. Any file in the same directory with the same filename but different file extension shall be treated as part of the same component. Rules regarding variable scoping and code-importing are intentionally undefined so that they can vary by language.

<table>
<tr>
<td>
my-button.html
</td>
<td>
my-button.any
</td>
</tr>
<tr>
<td>

```xml
<button onClick={{handleClick}}>         
  Clicks: {{count}}
</button>
```

</td>
<td>

```c
count = 0                      

void handleClick(event) {
  count++
}
```

</td>
</tr>
</table>

### Hole Punch

The "hole punch" pattern `{{ }}` is familiar since it appears like a regular [string interpolation](https://en.wikipedia.org/wiki/String_interpolation). However, in order to excel at both HTML-generation and DOM-manipulation there is one important distinction. Instead of simply returning a final string, it returns a "composition object" which simply just hangs onto the inputs for later use. Once a composition is built, it becomes trivial to either lazily generate the HTML in full or to compare its input values with an older composition's input values for anything that might have changed so that it may generate instructions needed for updating the DOM.

Contents inside the hole punch must be [expressions](<https://en.wikipedia.org/wiki/Expression_(computer_science)>), not [statements](<https://en.wikipedia.org/wiki/Statement_(computer_science)>).

The advantages of this approach are many:

1. **Simplicity** - state changes don't require scope tracking
1. **Derived data** - compositions compare the inline expression values, not the state itself
1. **Portability** - works equally well in WASM as server-side since it does not depend on a DOM tree to re-compose
1. **Event Handling** - DOM events are natural fit for [marshalling](<https://en.wikipedia.org/wiki/Marshalling_(computer_science)>)
1. **Precision** - can modify the values of individual DOM nodes instead of only brute-forcing large HTML fragments at a time
1. **Speed** - no virtual DOM necessary
1. **Memory** - no server-side node tree construction necessary

### Server-Side or Client-Side?

Why not both?

ZeroScript follows a [Unidirectional data flow](https://developer.android.com/jetpack/compose/architecture#udf) design pattern.

> A unidirectional data flow (UDF) is a design pattern where state flows down and events flow up. By following unidirectional data flow, you can decouple composables that display state in the UI from the parts of your app that store and change state.

Thankfully the DOM's event-handling model encapsulates events as objects. This makes them a natural fit for [marshalling](<https://en.wikipedia.org/wiki/Marshalling_(computer_science)>)
across the boundary where your language of choice is running whether that be remotely on the server or locally as WebAssembly.

Additionally, if your transport supports bi-directional communication, like WebSockets or even WASM-marshalling, this opens up your web app for realtime functionality - specifically, DOM updates that aren't user-initiated.

### Transports

Since ZeroScript calls for DOM events to be serialized, they can easily be transported to other runtimes for handling whether that be over WebAssembly interop, or through the network to the server. It can function across a number of different transports, each with their own benefits and tradeoffs.

| Transport          | Browser event fired           | DOM mutation instructions |
| ------------------ | ----------------------------- | ------------------------- |
| HTTP(s)            | Event POSTed to server        | Included in POST response |
| Server-sent events | Event POSTed to server        | Pushed via SSE            |
| WebSockets         | Event sent via WebSocket      | Pushed via WebSocket      |
| WebAssembly        | Event marshalled over interop | Marshalled over interop   |

> [!Important]
> All transports support realtime features through bi-directional communication except for HTTP(s).

### APIs are discouraged

There's nothing negative about using an API. It's just not necessary with the ZeroScript programming model. To compare, classical SPAs might respond to a DOM event in the browser by fetching data from an API and updating the DOM accordingly. With ZeroScript frameworks, since the event is automatically sent to the server for handling, there's no need to call a separate API since the data can be accessed directly.

> [!NOTE]
> This only applies to server-side runtimes. WebAssembly runtimes will likely still prefer using an API.

<br />
<br />

## State Management

Use `<state>` for values that change.  `<state>` can be shared with and changed by other components.

```xml
<state score=0 />
<MyButton count={{score}} />
<h1>The score is {{score}}</h1>
```

When `<state>` is assigned to an attribute, any changes to the attribute's value also changes the `<state>`. Here, every keypress will update the `<h1>` tag.

```xml
<state name="World" />
<input value={{name}} />
<h1>Hello {{name}}!</h1>
```

<br />
<br />

## Ecosystem

### XYZ-Flavored Zero

In the same manner as [GitHub flavored Markdown](https://github.github.com/gfm/), it is encouraged that guest languages bring their own embellishments that take advantage of their syntax's unique features. These deviations can borrow the "flavor" terminology in the same way, for example, "Acme flavored Zero."

### Known Implementations

Below is a running list of known guest languages and the various features they support.

| Features                        |   [xUI Tags](https://github.com/xui/xero)   |
| ------------------------------- | :---------: |
| Dynamic hosting                 |     ✅      |
| SSG (static site generation)    | coming soon |
| AoT (Ahead of time compilation) | coming soon |
| WASM (WebAssembly)              | coming soon |
| WebSockets                      |     ✅      |
| SSE (server-side events)        | coming soon |
| Event-POSTing                   | coming soon |
| Local caching                   | coming soon |
| Optimistic writes               | coming soon |
| Animations                      | coming soon |
| Global & session state          | coming soon |
| Branch preview                  | coming soon |
