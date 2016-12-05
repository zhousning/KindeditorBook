# Range

## 简介

​	range作为选区有开始位置和结束位置。

​	选区在本章当中用两个|表示。如：12|345|6，竖线是光标开始与结尾，两者之间代表选区。

测试用代码

```html
<p>
	<strong>123abc</strong>defg<b>hijk</b>lmn
</p>
<h1>
	123<br />
  	<br />
</h1>
```

```javascript
          var range = editor.cmd.range;
          var doc = editor.cmd.doc;
          var strong = doc.getElementsByTagName("strong")[0];
          var pnode = doc.getElementsByTagName("p")[0];      
          console.log(range);
		  //range.selectNode(strong);
          //range.selectNodeContents(pnode);
          //range.setStart(strong, 1);
          
          var text = range.toString();
          var stctn = range.startContainer;
          var stoft = range.startOffset;
          var edctn = range.endContainer;
          var edoft = range.endOffset;
          console.log("startCtn:");
          console.log(stctn);
          console.log("startOfst:");
          console.log(stoft);
          console.log("endCtn:");
          console.log(edctn);
          console.log("endOfst:");
          console.log(edoft);

          var cmacestor = range.commonAncestor();
          console.log("commonAncestor:");
          console.log(cmacestor);

          var text = range.toString();
          console.log("rangeToString");
          console.log(text);

          var fragment = range.cloneContents();
          console.log("cloneContent:");
          console.log(fragment);

          //up、down、enlarge、shrink这四个函数在这里调试就可以，我还有弄清楚这四个函数具体怎么回事！
		  range.up();
		  //range.down();
		  //range.enlarge();
		  //range.shrink();
          var stctn1 = range.startContainer;
          var stoft1 = range.startOffset;
          var edctn1 = range.endContainer;
          var edoft1 = range.endOffset;
          console.log(">>>>>>>>>>change range border");
          console.log("startCtn:");
          console.log(stctn1);
          console.log("startOfst:");
          console.log(stoft1);
          console.log("endCtn:");
          console.log(edctn1);
          console.log("endOfst:");
          console.log(edoft1);
```

## container

​	container就是光标所在的节点，分为startContainer(开始位置所在容器)和endContainer(结束位置所在容器)

​	光标存在的位置有两种类型：文本节点(光标在文本节点内)和元素节点(光标在元素节点内)，如：

```html
<p>
  <strong>12|3abc</strong>def
</p>
<h1>
  123<br/>|<br/>
</h1>
```

​	startContainer："123abc"文本节点，endContainer：h1元素节点，Kindeditor返回的这些值都是普通的dom节点

## offset

​	offset：在容器所有子节点中，相对于同辈兄弟节点的偏移位置，分为startOffset(开始位置偏移量)和endOffset(结束位置偏移量)。

​	光标在文本节点内，偏移量是相对于文本节点内的节点；光标在元素内，偏移量是相对于元素内的节点

​	比如上例：光标开始位置在文本节点内，前头有2个节点(1和2)，所以startOffset 是2；光标结束位置在元素节点内，前头有2个节点(“123”和br)，所以endOffset是2。

​	注：这里有一点需要注意的，以下格式化后的代码产生的offset是不一样的，因为浏览器在格式化代码显示的时候，通过加入回车来产生换行，所以格式化后的文本节点其实是“\n12|3abc”，第二种情况产生的startOffset是3

```html
<strong>12|3abc</strong>

<strong>
  12|3abc
</strong>
```

注：编辑器初始化时

```javascript
startContainer、endContainer是body，startOffset、endOffset是0
```

range是对象，是引用类型，但container、offset不是引用类型，如果range改变了，相应的内容需要重新获取，比如

```javascript
var range = editor.cmd.range;
var stctn = range.startContainer;
var stoft = range.startOffset;

range.setStart(strong, 1);//range改变，但stctn、stoft还是原先的值，需要重新获取新的range内容
var stctn1 = range.startContainer;
var stoft1 = range.startOffset;
```

**以下讲解Kindeditor中api常用

## commonAncestor()

取得同时包含着startContainer和endContainer节点的外层节点，如:

示例一：cmancestor是外层的p

```html
<p>
	<strong>123|abc</strong>defg<b>hi|jk</b>lmn
</p>
<h1>
	123<br />
  	<br />
</h1>
```

示例二：endContainer是body，cmancestor也是body

```html
<body>
  <p>
	<strong>123|abc</strong>defg<b>hijk</b>lmn
  </p>
  <h1>
	 123<br />
  <br />
  </h1>
  |<br />
</body>
```

## selectNode(node)

选择一个节点作为选区，比如选择strong节点，则光标就在节点前后放置

```javascript
range.selectNode(doc.getElementsByTagName("strong")[0]);
```
```html
<p>
	|<strong>123abc</strong>|defg<b>hijk</b>lmn
</p>
```

## selectNodeContents(node)

选择一个节点的内容作为选区，比如选择P节点的内容

```javascript
range.selectNodeContents(doc.getElementsByTagName("p")[0])
```

```html
<p>
	|<strong>123abc</strong>defg<b>hijk</b>lmn|
</p>
```

## toString()

返回选区内的文本内容

```html
<p>
	|<strong>123abc</strong>defg<b>hijk</b>lmn|
</p>
```

返回的结果是

```html
123|abcdefghi|jklmn
```

## cloneContents()

复制并返回range的内容。将range当中的内容放到一个document-fragment结果

var fragment = range.cloneContents();

```html
<p>
	|<strong>123abc</strong>defg<b>hijk</b>lmn|
</p>
```

返回结果

```html
#document-fragment
	<strong>123abc</strong>
	defg
	<b>hijk</b>
	lmn
```

## extractContents()

删除并返回range的内容。返回一个一个document-fragment结果，示例如上：

var fragment = range.extractContents();

注：以下4个函数还没有整明白具体怎么回事！！！

## up()

## down()

只处理container是元素节点的情况，且container当中有至少一个子节点

```

```

## enlarge()

以下内容是随便写的，仅做参考

光标在边界上，就查找最远的元素节点

```html
<strong><b>1|23|</b>abc</strong>def//左侧不动，右侧查找与包裹它的元素节点为兄弟节点的最近元素节点
```
如果光标(开始、结束)在边界上，

有相邻兄弟节点（非文本），就用下一个兄弟节点的最外层作为边界

没有兄弟节点，就用父节点作为边界

在父节点当中查找

## shrink()