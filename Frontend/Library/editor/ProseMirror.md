* 介绍: 构建富文本编辑器的工具 ( toolkit )

* 核心模块
  * [`prosemirror-model`](http://prosemirror.net/docs/ref/#model) defines the editor's [document model](http://prosemirror.net/docs/guide/#doc), the data structure used to describe the content of the editor.
  * [`prosemirror-state`](http://prosemirror.net/docs/ref/#state) provides the data structure that describes the editor's whole state, including the selection, and a transaction system for moving from one state to the next.
  * [`prosemirror-view`](http://prosemirror.net/docs/ref/#view) implements a user interface component that shows a given editor state as an editable element in the browser, and handles user interaction with that element.
  * [`prosemirror-transform`](http://prosemirror.net/docs/ref/#transform) contains functionality for modifying documents in a way that can be recorded and replayed, which is the basis for the transactions in the `state` module, and which makes the undo history and collaborative editing possible.

