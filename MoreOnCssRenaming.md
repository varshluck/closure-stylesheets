# Introduction #

Before reading this article, you should have already read the section on [renaming](http://code.google.com/p/closure-stylesheets/#Renaming) from the Closure Stylesheets overview. This article explains more of the options related to CSS renaming.


# Renaming Options #

There are two command-line flags of interest with respect to CSS renaming:

**[--rename](#--rename_Option.md)** tells Closure Stylesheets how to rename CSS classes
<br>
<b><a href='#--output-renaming-map-format_Option.md'>--output-renaming-map-format</a></b> tells Closure Stylesheets how to format the renaming map<br>
<br>
Each of these command line flags supports several different values. This is done to accommodate different styles of development.<br>
<br>
<br>
<h2>--rename Option</h2>

The <code>--rename</code> flag determines the algorithm that Closure Stylesheets uses to rename your CSS classes. The options are as follows:<br>
<br>
<h3>NONE</h3>

<code>NONE</code> should be used when no renaming should be done. This is the default option.<br>
<br>
<br>
<h3>DEBUG</h3>

<b><code>DEBUG</code></b> should be used when debugging CSS renaming. Each CSS class is trivially renamed by taking each part of the class name (where parts are delimited by hyphens) and appending a trailing underscore.<br>
<br>
This mode is helpful as a "debug" mode of sorts to catch cases where a class name is not wrapped with <code>goog.getCssName()</code>. For example, if you use <code>DEBUG</code> renaming in development and you notice that the style of a DOM element does not look right, then you should take a look to see what its CSS class name is. If you notice that it does not contain any underscores but you expected it to be renamed, then you should be able to track down the reference to the class name in your code and add the appropriate <code>goog.getCssName()</code> wrapper. If <code>NONE</code> renaming were used, then this bug would not reveal itself in development, and if <code>CLOSURE</code> renaming were used, it would take more work to reverse-map the obfuscated class name in the DOM to the original class name in your code.<br>
<br>
<br>
<h3>CLOSURE</h3>

<b><code>CLOSURE</code></b> should be used when minifying class names for production. Each CSS class name is split into parts (as delimited by hyphens), and then parts are renamed using the shortest available name. Parts are renamed consistently, so if <code>.foo-bar</code> is renamed to <code>.a-b</code>, then <code>.bar-foo</code> will be renamed to <code>.b-a</code>.<br>
<br>
This focus on splitting on hyphens stems from the fact that the <code>goog.getCssName()</code> function in the Closure Library has an optional second argument that is appended using a hyphen. For example, both of the following could be used to construct the CSS class <code>.foo-bar</code>:<br>
<br>
<pre><code>// Using the single-argument form:<br>
goog.getCssName('foo-bar');<br>
<br>
// Using the two-argument form:<br>
goog.getCssName(goog.getCssName('foo'), 'bar');<br>
</code></pre>

Both of the above expressions will yield the string <code>'foo-bar'</code> in the absence of any renaming map. Because the classes in the stylesheet are renamed without any knowledge of how the class names will be constructed in the JavaScript code, it must use a renaming scheme that supports both forms of construction.<br>
<br>
For historical reasons, the second argument to <code>goog.getCssName()</code> is appended using a hyphen. This is somewhat unfortunate because hyphens are also frequently used as visual separators in long CSS names, making it impossible to distinguish which parts come from the "base" name and which parts come from the "suffix" by looking at the name alone.<br>
<br>
This is unfortunate because it often results in slightly suboptimal CSS. For example, if a stylesheet has only one CSS class named <code>.foo-bar</code> that is exclusively referenced via <code>goog.getCssName('foo-bar')</code> in JavaScript code, it will be renamed to <code>.a-b</code> rather than <code>.a</code> in both the CSS and JS. One workaround is to adopt the convention of using an underscore as a visual separator rather than a hyphen. That is, name your class <code>.foo_bar</code> if it is referenced exclusively via <code>goog.getCssName('foo_bar')</code> in your JavaScript code. This will result in a single-letter renaming, as desired.<br>
<br>
Note that although non-ASCII characters are allowed in CSS class names, which could be used to produce class names with fewer characters, this generally results in less efficient gzip compression of the minified stylesheet. For this reason, the class names produced by <code>CLOSURE</code> renaming are limited to alphanumeric ASCII characters.<br>
<br>
<h2>--output-renaming-map-format Option</h2>

The renaming map produced by Closure Stylesheets often needs to be consumed by another tool, such as the <a href='http://code.google.com/p/closure-compiler/'>Closure Compiler</a>. Closure Stylesheets provides several output formats to provide straightforward integration with the Closure Tools, while also providing the opportunity for you to build your own tools on top of Closure Stylesheets.<br>
<br>
<br>
<h3>CLOSURE_COMPILED</h3>

<b><code>CLOSURE_COMPILED</code></b> should be used when compiling JavaScript with the <a href='http://code.google.com/p/closure-compiler/'>Closure Compiler</a> in either <code>SIMPLE</code> or <code>ADVANCED</code> mode. When <code>CLOSURE_COMPILED</code> is specified, the output is a JSON map of renaming information wrapped in a call to <code>goog.setCssNameMapping()</code>, such as:<br>
<br>
<pre><code>goog.setCssNameMapping({<br>
  "foo": "a",<br>
  "bar": "b"<br>
});<br>
</code></pre>

This file should be passed as an input to your compilation. The Compiler will remove this call to <code>goog.setCssNameMapping()</code> and use the object literal that is passed to it as the basis for replacing all calls to <code>goog.getCssName()</code> in the compiled code.<br>
<br>
<br>
<h3>CLOSURE_UNCOMPILED</h3>

<b><code>CLOSURE_UNCOMPILED</code></b> should be used with uncompiled Closure Library code. When <code>CLOSURE_UNCOMPILED</code> is specified, the output is a JSON map of renaming information assigned to the global variable <code>CLOSURE_CSS_NAME_MAPPING</code>, such as:<br>
<br>
<pre><code>CLOSURE_CSS_NAME_MAPPING = {<br>
  "foo": "a",<br>
  "bar": "b"<br>
};<br>
</code></pre>

This file should be loaded via a <code>&lt;script&gt;</code> tag before <code>base.js</code> is loaded for the Closure Library. This is because <code>base.js</code> checks to see whether the global <code>CLOSURE_CSS_NAME_MAPPING</code> variable is declared, and if so, uses its value as the renaming data for <code>goog.getCssName()</code>. This ensures that the mapping data is set before any calls to <code>goog.getCssName()</code> are made.<br>
<br>
<br>
<h3>JSON</h3>

<b><code>JSON</code></b> should be used when building a tool that wants to consume the renaming map data. When <code>JSON</code> is specified, the output is a pure JSON map, such as:<br>
<br>
<pre><code>{<br>
  "foo": "a",<br>
  "bar": "b"<br>
}<br>
</code></pre>

Currently, <code>JSON</code> is the default value for the <code>--output-renaming-map-format</code> option.<br>
<br>
<br>
<h3>PROPERTIES</h3>

<b><code>PROPERTIES</code></b> should be used as an alternative to <code>JSON</code> if, for some reason, your toolchain does not have a JSON parser available. When <code>PROPERTIES</code> is specified, the output is a <a href='http://en.wikipedia.org/wiki/.properties'>.properties</a> file formatted as <code>key=value</code> pairs without any comments, such as:<br>
<br>
<br>
<pre><code>foo=a<br>
bar=b<br>
</code></pre>

This format is extremely strict and simple so that it should be easy to write a custom parser for it in whatever language you are using to process the renaming map data.