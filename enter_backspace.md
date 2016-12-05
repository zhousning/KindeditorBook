# 回车退格

## 简介

​	键盘代码(keycode)：backspace 8，delete 46，enter 13

## 富文本编辑器特性

### 回车特性

​	1. 回车总是会复制回车位置外层的父节点的样式，使得新产生的段落会有光标原先所在位置相关属性，如 ：

```html
<p class = "title"><img></p>
回车后新段落
<p class = "title"><br/></p>
```

​	2. 富文本编辑器默认是用div表示新的段落

```html
<div>
	<br />
</div>
产生的结果是：
	如果段落里有内容<h1>xxxxxxxxxxxx</h1>，回车产生的新段落外层是<h1>标签<h1><br/></h1>
	如果段落里无内容<h1><br/></h1>，回车产生的新段落外层是<div>标签<div><br/></div>
```

​	kindeditor当中通过以下函数，将div替换成了p

```javascript
	if (!pSkipTagMap[tagName]) {
		  _nativeCommand(doc, 'formatblock', '<p>');
	}
变成了我们需要的<p><br/></p>
```

​	3. 段落之前回车，光标在当前行，段落之后回车，光标在新行，这和普通的word操作是一样的

​	4.新段落是keydown之后，keyup之前产生的

### 退格/delete特性

```
退格相当于将最外层父节点下面的所有节点移动到上一个节点内部

退格之前段落有内容
	<p>
		123| <span>456</span>
	</p>

退格之前段落无内容
	<p><br/></p>
退格后
	<p>
 		<span>|456</span>
	</p>

不同标签退格
	<h1>
   	123
	<p>
  		<img>

img退格后
	<h1>
    	123<img>
```

### 	

## Kindeditor处理回车

产生流程：键盘按下——产生新段落——键盘抬起，新段落是在它俩之间的过程中产生的，（这里也可能不是这样，但不影		响理解）

处理流程：keydown处理原段落，keyup处理新段落

处理函数

```javascript
K(doc).keydown(function(e) {
    //backspace(8)、delete(46)处理
    if (e.which == 8 || e.which == 46) {
     	......
    }
    //以下处理回车事件
	if (e.which != 13 || e.shiftKey || e.ctrlKey || e.altKey) {
		return;
	}
	self.cmd.selection();
    //获取光标所在位置外层祖先节点标签名
	var tagName = getAncestorTagName(self.cmd.range);
	if (tagName == 'marquee' || tagName == 'select') {
		return;
	}
    //newlineTag:新的段落是<p><br/></p>还是<br/>,我们设置是前者
    //以下if不执行，不用看
	if (newlineTag === 'br' && !brSkipTagMap[tagName]) {
		e.preventDefault();
		self.insertHtml('<br />' + (_IE && _V < 9 ? '' : '\u200B'));
		return;
	}
    //外层标签如果不是pSkipTagMap = _toMap('p,h1,h2,h3,h4,h5,h6,pre,li,blockquote')这其中的就将其替换成P标签
    //pSkipTagMap = {"p": true, "h1": true, ...}
    //比如：<div>123|456</div>,在|回车，div被替换成P，变为<P>123|456</P>
    //如果是在多媒体工具条或者标题中回车，因为外层我们设置了contenteditable=false，if里面会执行，但不会产生编辑效果
	if (!pSkipTagMap[tagName]) {
		 _nativeCommand(doc, 'formatblock', '<p>');
	}
});
  //回车新段落产生是在keydown结束后，keyup之前
  //回车段落：<P>123</P><P>456</P>
K(doc).keyup(function(e) {
    //backspace(8)、delete(46)处理
    //退格光标会跑到前一个节点内，退格相当于将光标原位置最外层父节点下面的所有节点移动到上一个节点内部
    if (e.which == 8 || e.which == 46) {
    	......
    }
    //以下处理回车事件
	if (e.which != 13 || e.shiftKey || e.ctrlKey || e.altKey) {
		return;
	}
	if (newlineTag == 'br') {
		return;
	}
    //内核是GECKO的(火狐)的处理
	if (_GECKO) {
		var root = self.cmd.commonAncestor('p');
		var a = self.cmd.commonAncestor('a');
		if (a && a.text() == '') {
			a.remove(true);
			self.cmd.range.selectNodeContents(root[0]).collapse(true);
			self.cmd.select();
		}
		return;
	}
	self.cmd.selection();
	var tagName = getAncestorTagName(self.cmd.range);
	if (tagName == 'marquee' || tagName == 'select') {
		return;
	}
    //开始处理新产生的段落    
    /*
     * 新产生的段落存在有div的情况：
     * <h1>xxxxxxxxxxxx</h1>回车产生的新段落外层是<h1>标签
     * 然而<h1><br/></h1>无内容回车产生的新段落外层是<div>标签
     */
	if (!pSkipTagMap[tagName]) {
      //执行formatblock并不会改变原range当中的相关属性内容，比如产生了div，将div替换成了p，但是range的endcontainer还是原来的div，以下的div获取还是一样能获取到
      //针对这种已经被变为了<p><br/></p>的情况，这个地方也可以设置不让他继续往下执行，谁想改改可以试试
	   _nativeCommand(doc, 'formatblock', '<p>');
	}
	var div = self.cmd.commonAncestor('div');
	if (div) {
      //工具条内回车,就终止继续执行
      if (div.hasClass(self.plugin.kemdClassName.mediaOptCtn)) {
        return;
      }
      //除工具条以外的多媒体内容里回车
      if (div.hasAttr("data-type")) {
        //多媒体内回车，在多媒体节点div下面添加空行段落
			  var p = K('<p><br/></p>');
        div.after(p[0]);
        self.cmd.range.selectNodeContents(p[0]);
        self.cmd.select();
      } else {
        //普通段落内回车产生div，替换div为p
        var p = K('<p></p>'),
            child = div[0].firstChild;
        //将新段落里的所有内容放到p标签内        
        while (child) {
          var next = child.nextSibling;
          p.append(child);
          child = next;
        }
        //将p插入到div之后        
        div.before(p);
        //删除div，连同子节点一块        
        div.remove();
        //高亮选择新产生的段落        
        self.cmd.range.selectNodeContents(p[0]);
        self.cmd.select();
      }
	}
});
```

## Kindeditor处理退格

```javascript
K(doc).keydown(function(e) {
    //backspace(8)、delete(46)处理
    if (e.which == 8 || e.which == 46) {
      var outNode = self.cmd.commonNode({'h1,h2,h3,h4,p' : '*'});
      var range = self.cmd.range;
      var mediaNode = null, position;
      var ec = range.endContainer, eo = range.endOffset;

      //处理文本节点内退格
      if (outNode) {
        if (e.which == 8) {
          mediaNode = outNode.prev();
          position = 0;
        } else if (e.which == 46) {
          mediaNode = outNode.next();
          position = ec.length;
        }
        //光标在文本节点之内正常删除，光标在段落最前面，就要判断一下它之前的是不是多媒体节点，如果是就禁止继续删除
        //根据光标是在最开头的位置来判断是不是到段落的开始位置了
        if (mediaNode && mediaNode.hasClass(self.plugin.kemdClassName.mediaCtn) && eo == position) {
          e.preventDefault();
        }
      }
    }
});

K(doc).keyup(function(e) {
    //backspace(8)、delete(46)处理
    //退格光标会跑到前一个节点内，退格相当于将光标原位置最外层父节点下面的所有节点移动到上一个节点内部
    if (e.which == 8 || e.which == 46) {
      var range = self.cmd.range;
      var ec = range.endContainer;
      var parents = ec.parentNode;
      var childs = parents.childNodes;
      /*
       * 退格之前的段落有无内容，光标所在的位置是不一样的，endcontainer也是不一样的，如下|代表光标
       * 针对这两种情况产生的span标签需要分别处理
       */
      //退格之前的段落无内容，如：<p><br/></p>,退格后：<p><span>|456</span></p>，它的endcontainer就是文本节点
      if (parents.nodeName === "SPAN") {
        K(parents).remove(true);
        range.setStartBefore(ec).collapse(true);
        self.cmd.select();
      }
      //退格之前的段落有内容，如：<p>123</p>,退格后：<p>123|<span>456</span></p>,它的endcontainer就是p节点
      for(var i = 0;i<childs.length;i++){
        if(childs[i].nodeName === "SPAN"){
          K(childs[i]).remove(true);
        }
      }
      //退格会产生多个分段的文本节点，使用序列化将分割的文本节点拼接在一起
      self.cmd.doc.normalize();
      return;
    }
});
```