# 标题设置

## 简介

要求：将最外层父节点替换成相应的标题节点，所有标题节点都直接存在body下

​	    多媒体节点内不允许设置标题样式

注：请参考数据结构规则阅读本章

## 创建工具

```javascript
	_each('title_one,title_two,title_three,title_four,title_content,order_list'.split(','), function(i, name) {
		self.clickToolbar(name, function() {
      var range = self.cmd.range;
      //获取当前光标所在的节点
      var dataTypeEc = range.endContainer;
      //由光标所在的节点向body遍历（由内到外），判断是不是在多媒体节点内
      while(dataTypeEc.nodeName !== "BODY"){
        if (dataTypeEc.nodeName !== "#text") {
          var dataType = dataTypeEc.getAttribute("data-type");
          if (dataType == "image" || dataType == "image_caption" || dataType == "table" || dataType == "table_caption" || dataType == "math") {
            return;
          }
        }
        dataTypeEc = dataTypeEc.parentNode;
      }
      //排除多媒体节点后，获取光标所在的最外层的节点
      //比如要设置<p>XXXXX</p>为一级标题
      var cnode = self.cmd.commonNode({'h1,h2,h3,h4,p' : '*'});
      if (cnode) {
        //选择除最外层节点内的所有内容做为选区
        //即XXXXXX
        range.selectNodeContents(cnode[0]);
        //将新选区传入标题设置函数中
        //title = <h1>XXXXXX</h1>
        var title = _setTitle(range,name);
        //执行完以上步骤之后节点就变成了<p><h1>XXXXXX</h1></p>
        //删除最外层节点，保留其中的所有子节点,<h1>XXXXXX</h1>
        cnode.remove(true);
        //选择新插入的内容，将<h1>XXXXXX</h1>做为选区
        range.selectNodeContents(title);
        //高亮显示选区
        self.cmd.select();
        self.cmd.selection();
        //更新工具条状态
				self.updateState();
      }
    });
  });
```

## 设置标题

```javascript
 //surroundContents使用指定节点包裹内容
  function _setTitle(range,name) {
    var title = null;
    switch(name){
      case 'title_one':
        title = K('<h1 data-type="text" class="title-one"></h1>')[0];
        range.surroundContents(title);
        break;
      case 'title_two':
        title = K('<h2 data-type="text" class="title-two"></h2>')[0];
        range.surroundContents(title);
        break;
      case 'title_three':
        title = K('<h3 data-type="text" class="title-three"></h3>')[0];
        range.surroundContents(title);
        break;
      case 'title_four':
        title = K('<h4 data-type="text" class="title-four"></h4>')[0];
        range.surroundContents(title);
        break;
      case 'title_content':
        title = K('<p></p>')[0];
        range.surroundContents(title);
        break;
      case 'order_list':
        title = K('<h4 data-type="text" data-type="order_list"></h4>')[0];
        range.surroundContents(title);
        break;
    }
    return title;
  }

```

## 更新工具条显示状态

```javascript
updateState : function() {
	var self = this;
    var toolType = {
      "h1" : "title_one", "h2" : "title_two", "h3" : "title_three", "p" : "title_content", "sub" : "subscript", "sup" : "superscript"
    }
    //查找当前光标所在的节点是否存在于以下节点中
    var outNode = self.cmd.commonNode({'h1,h2,h3,h4,p,sup,sub' : '*'});
    var range = self.cmd.range;
    if (outNode) {
      //存在就在工具条上显示相应的状态，其中用到的函数是KE自己的直接调用就可以更新状态
      K.each(toolType, function(k, v) {
		    outNode.name == k ? self.toolbar.select(toolType[k]) : self.toolbar.unselect(toolType[k]);
      });
    } else {
      //如果都没有，就将取消所有状态显示
      K.each(toolType, function(k, v) {
        self.toolbar.unselect(toolType[k]);
      });
    }
		return self;
}

```