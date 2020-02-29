# 综述

> 参考[Tiptap](https://github.com/scrumpy/tiptap)

## 介绍

* 一个基于Prosemirror和Vue实现的富文本编辑器. 

* 编辑器的内容可与**html**或文档的**json描述**之间转化

  > Prosemirror很容易支持html与markdown之间的转化, 这个Tiptap...不支持.

* 支持Vue组件

  > 构建时需要添加Vue编译器, 否则会报错.

## Examples
To check out some live examples, visit [tiptap.scrumpy.io](https://tiptap.scrumpy.io/).

## Installation

```
npm install tiptap
```
or
```
yarn add tiptap
```

# Basic Setup
```vue
<template>
  <editor-content :editor="editor" />
</template>

<script>
// Import the editor
import { Editor, EditorContent } from 'tiptap'

export default {
  components: {
    EditorContent,
  },
  data() {
    return {
      editor: null,
    }
  },
  mounted() {
    this.editor = new Editor({
      content: '<p>This is just a boring paragraph</p>',
    })
  },
  beforeDestroy() {
    this.editor.destroy()
  },
}
</script>
```

# Components

| **Name** | **Description** |
| --- | --- |
| `<editor-content />` | Here the content will be rendered. |
| `<editor-menu-bar />` | Here a menu bar will be rendered. |
| `<editor-menu-bubble />` | Here a menu bubble will be rendered. |
| `<editor-floating-menu />` | Here a floating menu will be rendered. |

## Editor

### Editor Properties

| **Property** | **Type** | **Default** | **Description** |
| --- | :---: | :---: | --- |
| `content` | `Object\|String` | `null` | The editor state object used by Prosemirror. You can also pass HTML to the `content` slot. When used both, the `content` slot will be ignored. |
| `editorProps` | `Object` | `{}` | A list of [Prosemirror editorProps](https://prosemirror.net/docs/ref/#view.EditorProps). |
| `editable` | `Boolean` | `true` | When set to `false` the editor is read-only. |
| `autoFocus` | `Boolean` | `false` | Focus the editor on init. |
| `extensions` | `Array` | `[]` | A list of extensions used, by the editor. This can be `Nodes`, `Marks` or `Plugins`. |
| `useBuiltInExtensions` | `Boolean` | `true` | By default tiptap adds a `Doc`, `Paragraph` and `Text` node to the Prosemirror schema. |
| `dropCursor` | `Object` | `{}` | Config for `prosemirror-dropcursor`. |
| `parseOptions` | `Object` | `{}` | A list of [Prosemirror parseOptions](https://prosemirror.net/docs/ref/#model.ParseOptions). |
| `onInit` | `Function` | `undefined` | This will return an Object with the current `state` and `view` of Prosemirror on init. |
| `onFocus` | `Function` | `undefined` | This will return an Object with the `event` and current `state` and `view` of Prosemirror on focus. |
| `onBlur` | `Function` | `undefined` | This will return an Object with the `event` and current `state` and `view` of Prosemirror on blur. |
| `onUpdate` | `Function` | `undefined` | This will return an Object with the current `state` of Prosemirror, a `getJSON()` and `getHTML()` function and the `transaction` on every change. |

### Editor Methods

| **Method** | **Arguments**| **Description** |
| --- | :---: | --- |
| `setContent` | `content, emitUpdate, parseOptions` | Replace the current content. You can pass an HTML string or a JSON document. `emitUpdate` defaults to `false`. `parseOptions` defaults to those provided in constructor. |
| `clearContent` | `emitUpdate` | Clears the current content. `emitUpdate` defaults to `false`. |
| `setOptions` | `options` | Overwrites the current editor properties. |
| `registerPlugin` | `plugin` | Register a Prosemirror plugin. |
| `getJSON` | – | Get the current content as JSON. |
| `getHTML` | – | Get the current content as HTML. |
| `focus` | – | Focus the editor. |
| `blur` | – | Blur the editor. |
| `destroy` | – | Destroy the editor. |

## EditorMenuBar

The `<editor-menu-bar />` component is renderless and will receive some properties through a scoped slot.

| **Property** | **Type** | **Description** |
| --- | :---: | --- |
| `commands` | `Array` | A list of all commands. |
| `isActive` | `Object` | An object of functions to check if your selected text is a node or mark. `isActive.{node|mark}(attrs)` |
| `getMarkAttrs` | `Function` | A function to get all mark attributes of your selection. |
| `focused` | `Boolean` | Whether the editor is focused. |
| `focus` | `Function` | A function to focus the editor. |

**Example**

```vue
<template>
  <editor-menu-bar :editor="editor" v-slot="{ commands, isActive }">
    <div>
      <button :class="{ 'is-active': isActive.bold() }" @click="commands.bold">
        Bold
      </button>
      <button :class="{ 'is-active': isActive.heading({ level: 2 }) }" @click="commands.heading({ level: 2 })">
        H2
      </button>
    </div>
  </editor-menu-bar>
</template>
```

## EditorMenuBubble

The `<editor-menu-bubble />` component is renderless and will receive some properties through a scoped slot.

| **Property** | **Type** | **Description** |
| --- | :---: | --- |
| `commands` | `Array` | A list of all commands. |
| `isActive` | `Object` | An object of functions to check if your selected text is a node or mark. `isActive.{node|mark}(attrs)` |
| `getMarkAttrs` | `Function` | A function to get all mark attributes of your selection. |
| `focused` | `Boolean` | Whether the editor is focused. |
| `focus` | `Function` | A function to focus the editor. |
| `menu` | `Object` | An object for positioning your menu. |

**Example**

```vue
<template>
  <editor-menu-bubble :editor="editor" v-slot="{ commands, isActive, menu }">
    <div
      :class="{ 'is-active': menu.isActive }"
      :style="`left: ${menu.left}px; bottom: ${menu.bottom}px;`"
    >
      <button :class="{ 'is-active': isActive.bold() }" @click="commands.bold">
        Bold
      </button>
      <button :class="{ 'is-active': isActive.heading({ level: 2 }) }" @click="commands.heading({ level: 2 })">
        H2
      </button>
    </div>
  </editor-menu-bubble>
</template>
```

## EditorFloatingMenu

The `<editor-floating-menu />` component is renderless and will receive some properties through a scoped slot.

| **Property** | **Type** | **Description** |
| --- | :---: | --- |
| `commands` | `Array` | A list of all commands. |
| `isActive` | `Object` | An object of functions to check if your selected text is a node or mark. `isActive.{node|mark}(attrs)` |
| `getMarkAttrs` | `Function` | A function to get all mark attributes of your selection. |
| `focused` | `Boolean` | Whether the editor is focused. |
| `focus` | `Function` | A function to focus the editor. |
| `menu` | `Object` | An object for positioning your menu. |

**Example**

```vue
<template>
  <editor-floating-menu :editor="editor" v-slot="{ commands, isActive, menu }">
    <div
      :class="{ 'is-active': menu.isActive }"
      :style="`top: ${menu.top}px`"
    >
      <button :class="{ 'is-active': isActive.bold() }" @click="commands.bold">
        Bold
      </button>
      <button :class="{ 'is-active': isActive.heading({ level: 2 }) }" @click="commands.heading({ level: 2 })">
        H2
      </button>
    </div>
  </editor-floating-menu>
</template>
```

# Extensions

By default, the editor will only support paragraphs. Other nodes and marks are available as **extensions**. There is a package called `tiptap-extensions` with the most basic nodes, marks, and plugins.

## Available Extensions

```vue
<template>
  <div>
    <editor-menu-bar :editor="editor" v-slot="{ commands, isActive }">
        <button :class="{ 'is-active': isActive.bold() }" @click="commands.bold">
          Bold
        </button>
    </editor-menu-bar>
    <editor-content :editor="editor" />
  </div>
</template>

<script>
import { Editor, EditorContent, EditorMenuBar } from 'tiptap'
import {
  Blockquote,
  CodeBlock,
  HardBreak,
  Heading,
  OrderedList,
  BulletList,
  ListItem,
  TodoItem,
  TodoList,
  Bold,
  Code,
  Italic,
  Link,
  Strike,
  Underline,
  History,
} from 'tiptap-extensions'

export default {
  components: {
    EditorMenuBar,
    EditorContent,
  },
  data() {
    return {
      editor: new Editor({
        extensions: [
          new Blockquote(),
          new CodeBlock(),
          new HardBreak(),
          new Heading({ levels: [1, 2, 3] }),
          new BulletList(),
          new OrderedList(),
          new ListItem(),
          new TodoItem(),
          new TodoList(),
          new Bold(),
          new Code(),
          new Italic(),
          new Link(),
          new Strike(),
          new Underline(),
          new History(),
        ],
        content: `
          <h1>Yay Headlines!</h1>
          <p>All these <strong>cool tags</strong> are working now.</p>
        `,
      }),
    }
  },
  beforeDestroy() {
    this.editor.destroy()
  },
}
</script>
```

## Create Custom Extensions

The most powerful feature of tiptap is that you can create your own extensions. There are 3 types of extensions.

| **Type** | **Description** |
| --- | --- |
| `Extension` | The most basic type. It's useful to register some [Prosemirror plugins](https://prosemirror.net/docs/guide/) or some input rules. |
| `Node` | Add a custom node. Nodes are block elements like a headline or a paragraph. |
| `Mark` | Add a custom mark. Marks are used to add extra styling or other information to inline content like a strong tag or links. |

## Extension Class

| **Method** | **Type** | **Default** | **Description** |
| --- | :---: | :---: | --- |
| `get name()` | `String` | `null` | Define a name for your extension. |
| `get defaultOptions()` | `Object` | `{}` | Define some default options. The options are available as `this.$options`. |
| `get plugins()` | `Array` | `[]` | Define a list of [Prosemirror plugins](https://prosemirror.net/docs/guide/). API见[Plugin System](https://prosemirror.net/docs/ref/#state.Plugin_System) |
| `keys({ schema })` | `Object` | `null` | Define some keybindings. |
| `commands({ schema, attrs })` | `Object` | `null` | Define a command. |
| `inputRules({ schema })` | `Array` | `[]` | Define a list of input rules. |
| `pasteRules({ schema })` | `Array` | `[]` | Define a list of paste rules. |
| `get update()` | `Function` | `undefined` | Called when options of extension are changed via `editor.extensions.options` |

## Node|Mark Class

| **Method** | **Type** | **Default** | **Description** |
| --- | :---: | :---: | --- |
| `get name()` | `String` | `null` | Define a name for your node or mark. |
| `get defaultOptions()` | `Object` | `{}` | Define some default options. The options are available as `this.$options`. |
| `get schema()` | `Object` | `null` | Define a [schema](https://prosemirror.net/docs/guide/#schema). |
| `get view()` | `Object` | `null` | Define a node view as a vue component. |
| `keys({ type, schema })` | `Object` | `null` | Define some keybindings. |
| `commands({ type, schema, attrs })` | `Object` | `null` | Define a command. For example this is used for menus to convert to this node or mark. |
| `inputRules({ type, schema })` | `Array` | `[]` | Define a list of input rules. |
| `pasteRules({ type, schema })` | `Array` | `[]` | Define a list of paste rules. |
| `get plugins()` | `Array` | `[]` | Define a list of [Prosemirror plugins](https://prosemirror.net/docs/guide/). API见[Plugin System](https://prosemirror.net/docs/ref/#state.Plugin_System) |

## Create a Node

Let's take a look at a real example. This is basically how the default `blockquote` node from [`tiptap-extensions`](https://www.npmjs.com/package/tiptap-extensions) looks like.

```js
import { Node } from 'tiptap'
import { wrappingInputRule, setBlockType, toggleWrap } from 'tiptap-commands'

export default class BlockquoteNode extends Node {

  // choose a unique name
  get name() {
    return 'blockquote'
  }

  // the prosemirror schema object
  // take a look at https://prosemirror.net/docs/guide/#schema for a detailed explanation
  get schema() {
    return {
      content: 'block+',
      group: 'block',
      defining: true,
      draggable: false,
      // define how the editor will detect your node from pasted HTML
      // every blockquote tag will be converted to this blockquote node
      parseDOM: [
        { tag: 'blockquote' },
      ],
      // this is how this node will be rendered
      // in this case a blockquote tag with a class called `awesome-blockquote` will be rendered
      // the '0' stands for its text content inside
      toDOM: () => ['blockquote', { class: 'awesome-blockquote' }, 0],
    }
  }

  // this command will be called from menus to add a blockquote
  // `type` is the prosemirror schema object for this blockquote
  // `schema` is a collection of all registered nodes and marks
  commands({ type, schema }) {
    return () => toggleWrap(type)
  }

  // here you can register some shortcuts
  // in this case you can create a blockquote with `ctrl` + `>`
  keys({ type }) {
    return {
      'Ctrl->': toggleWrap(type),
    }
  }

  // a blockquote will be created when you are on a new line and type `>` followed by a space
  inputRules({ type }) {
    return [
      wrappingInputRule(/^\s*>\s$/, type),
    ]
  }

}
```

## Create a Node as a Vue Component

The real power of the nodes comes in combination with Vue components. Let us build an iframe node, where you can change its URL (this can also be found in our [examples](https://github.com/scrumpy/tiptap/tree/master/examples/Components/Routes/Embeds)).

```js
import { Node } from 'tiptap'

export default class IframeNode extends Node {

  get name() {
    return 'iframe'
  }

  get schema() {
    return {
      // here you have to specify all values that can be stored in this node
      attrs: {
        src: {
          default: null,
        },
      },
      group: 'block',
      selectable: false,
      // parseDOM and toDOM is still required to make copy and paste work
      parseDOM: [{
        tag: 'iframe',
        getAttrs: dom => ({
          src: dom.getAttribute('src'),
        }),
      }],
      toDOM: node => ['iframe', {
        src: node.attrs.src,
        frameborder: 0,
        allowfullscreen: 'true',
      }],
    }
  }

  // return a vue component
  // this can be an object or an imported component
  get view() {
    return {
      // there are some props available
      // `node` is a Prosemirror Node Object
      // `updateAttrs` is a function to update attributes defined in `schema`
      // `view` is the ProseMirror view instance
      // `options` is an array of your extension options
      // `selected`
      props: ['node', 'updateAttrs', 'view'],
      computed: {
        src: {
          get() {
            return this.node.attrs.src
          },
          set(src) {
            // we cannot update `src` itself because `this.node.attrs` is immutable
            this.updateAttrs({
              src,
            })
          },
        },
      },
      template: `
        <div class="iframe">
          <iframe class="iframe__embed" :src="src"></iframe>
          <input class="iframe__input" type="text" v-model="src" v-if="view.editable" />
        </div>
      `,
    }
  }

}
```

--------------

**NodeView Prop Types**

| **Prop** | **Type** | **Description** |
| --- | :---: | --- |
| `node` | `Object` | The Prosemirror node object. Common use case is to get `node.attrs` using a getter on a computed property. |
| `updateAttrs` | `Function` | A function to update `node.attrs` defined in `schema`. Common use case is as setter on a computed property. |
| `view` | `Object` | The Prosemirror editor view instance. |
| `options` | `Array` | An array of your extension options. |
| `getPos` | `Function` | A function that returns the anchored position of the node. |
| `selected` | `Boolean` | A boolean that is set when the node is or is not selected. Common use case is using `watch` to see when the view is selected/unselected to do something, such focus an `<input>` or refocus the editor. |

