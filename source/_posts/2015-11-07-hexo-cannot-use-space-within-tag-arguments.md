title: hexo cannot use space within tag plugin arguments
sticky: 0
toc_number: false
date: 2015-11-07 10:44:06
tags:
  - hexo
  - tag plugin
  - Nunjucks
categories:
  - hexo
---

When I use the hexo tag plugin, i found that I cannot use space within the parameter of a tag plugin, even I wrap the parameter with double quote. This confuses me a lot.

Originally, I think this may be the bug of a 3rd party tag plugin for `hexo`.
But after some investigation, this is the bug of hexo itself.


<!--more-->

# Nunjucks

Before I start to explain about the detail of this bug.
I indeed speed several hours to dive into the code of `Nunjucks`.
{% blockquote Nunjucks https://mozilla.github.io/nunjucks/ %}
Nunjucks
A rich and powerful templating language for JavaScript.
{% endblockquote %}
`Nunjucks` is an interesting template framework for JavaScript. It has many great features and one of them used by `hexo` is the [custom tag](https://mozilla.github.io/nunjucks/api.html#custom-tags).

# Nunjucks Custom Tag
In `Nunjucks`, you can define the `custom tag` to integrate it into whole lexical analytics system. So that it will be processed as a `Nunjucks` native supported tag.

In order to implement a `custom tag`, you need to implement two function:
## parse
`parse` is used to get the travel through lexically-marked token. So that you will analyze each part of the arguments you want for your specific `custom tag`.
For example, if you define a tag like below:

```javascript
{% mytag "param1" "param2" "param3" %}
```
You need to define the `parse` function like:

```javascript
function(parser, nodes, lexer)
```

You use the `parser.nextToken()` to get each part of your arguments.

For example (here is the pseudo code to explain, more info refer to [lexer.js of Nunjucks](https://github.com/mozilla/nunjucks/blob/master/src/lexer.js):
1. Node -> { type: TOKEN_WHITESPACE, value: ' ' }
2. Node -> { type: TOKEN_STRING, value: 'param1' }
3. Node -> { type: TOKEN_WHITESPACE, value: ' ' }
4. Node -> { type: TOKEN_STRING, value: 'param2' }
5. Node -> { type: TOKEN_WHITESPACE, value: ' ' }
6. Node -> { type: TOKEN_STRING, value: 'param3' }
7. Node -> { type: TOKEN_BLOCK_END, value: '%}' }

Usually, we travel through each of the lexical node, save them as specific node in `nodes` parameter back to `Nunjucks` framework, and found the `TOKEN_BLOCK_END` to stop our process for any `custom tag`

## Run
`Run` is used to process the information you just analyze from the `parse` function. Simply speaking, it is used to  render the parameters to its expected result for the case of `hexo`


# Hexo implementation issue
Here is the [pull request](https://github.com/hexojs/hexo/pull/1581) for fix the issue, i would like to talk based on it.

![pull_request_hexo_space_issue.jpg](http://7xljtv.com1.z0.glb.clouddn.com/images/2015-11-07-hexo-cannot-use-space-within-tag-arguments/pull_request_hexo_space_issue.jpg)


We can see that, current implementation of `hexo`:
1. simply connect all the parameter together as a string in `parse` function
In fact, i do not understand here, it use a array `args`, but use the string `+=` to connection all things together, which result in a string. Cannot make sure it is a expected code or the writer think he could use `+=` for array elements.
2. simply split the string with space and ignore all the lexical context in `run`
`Hexo` simply split all the parameter with space, this is the root cause for non space supported issue. And it pass the arguments into registered function. so we cannot use the space anymore even wrapped in a string literal.

One more detail about my change:
If you want to use the `Nunjucks` to pass the result of `parse` correctly to `run`, you need to follow the internal structure of `Nunjucks`. Here my fix need to pass a array to `run`, instead of give the value of a `nodes.Literal` a simple javascript array object, we need to create a `nodes.Array` object, and add each of part of the tag arguments as `nodes.Literal` into it. so that `Nunjucks` can understand what we really want to pass.

---
So here i give a fix and hope hexo will merge it ASAP.
In case anyone see the same issue, so I write it down here.


---

**References**:

- [Nunjucks](https://mozilla.github.io/nunjucks/)



---

{% raw %}
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
Please refer to the author's name `winterTTr` when you remix, transform, and build upon this material. 
{% endraw %}
本作品由`winterTTr`创作，采用[知识共享署名-非商业性使用 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。修改，参照或者转载请注明作者`winterTTr`


