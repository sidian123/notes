# 引言

* 介绍

  * Firefox 有多种附加组件 ( Add-ons ), 这里仅介绍扩展 ( Extensions ) 

    > 其他附加组件有
    >
    > * 主题 ( Themes )
    > * [Search engine plugins](https://developer.mozilla.org/en-US/docs/Web/OpenSearch) add new search engines to the browser's search bar.
    > * [User dictionaries](https://developer.mozilla.org/en-US/docs/Mozilla/Creating_a_spell_check_dictionary_add-on) let you spell-check in different languages.
    > * [Language packs](https://support.mozilla.org/kb/use-firefox-interface-other-languages-language-pack) let you have more languages available for the user interface of Firefox. 

  * 扩展是JavaScript, HTML和CSS编写, 并提供了浏览器相关的[WebExtensions APIs](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions).

    > 据说可跨平台? 不能API还是要去查看下兼容性的

* 发布

  * [Addons.mozilla.org](https://addons.mozilla.org) 上可发布和获取插件
  * 插件需要被签名才能被其他用户安装, 见[Signing and distributing your add-on](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/Distribution).

# 扩展

## manifest.json











* 参考
  * [Add-ons](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons)
  * [WebExtensions APIs](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions)
  * [Browser support for JavaScript APIs](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Browser_support_for_JavaScript_APIs) 扩展API在不同浏览器中的兼容性