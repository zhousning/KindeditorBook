

# 自动排版

## 简介

​	注：在阅读本章之前，请仔细看下编辑区内容，参照数据结构规则学习

​	自动排版通过遍历body下所有子节点(不包括孙子节点)，使用正则表达式匹配每一个节点，是标题就设置标题格式，是多媒体标题就用相应数据结构组成的html代码。

body下的内容示例：

```html
	<div id="MathJax_Message" style="display: none;"></div>//由mathjax添加

	<p>第一章 绪论</p>  //也可能是h标签包裹

   	  	<p>1.1 研究背景和研究意义

		<h2>1.2 国内外研究动态

			<p>1.2.1 国内研究动态

			<h3>1.2.2 国外研究动态

		<h2>1.3 论文结构安排

			<p>第1章 XXXX

			<p>第2章 XXXX

			<p>第3章 XXXX

```

识别一级标题时会出现正文当中也存在符合一级标题格式的内容，如以上结构，我们只认为最开始存在的匹配一级标题的才是一级标题，其他都不是，所以需要单独对一级标题设置。

​	

## 自动排版函数

```javascript
   /*
   * 自动排版
   * 从body下的第一个子节点开始，一个一个遍历，使用正则匹配查找需要排版内容
   */
  setAutoLayout : function() {
		var self = this;
    //mainbody当中一级标题：<p>或<hX>  第一章/1./1、/一/一./一、XXXXX  </p>或</hX>
    var mainbodyH1Reg = /<(?:p|h\d)[^>]*>(?:&nbsp;|\s)*((?:第(?:&nbsp;|\s)*[\d一二三四五六七八九十]+(?:&nbsp;|\s)*章|\d[\.](?:&nbsp;|\s)*[^\d]|\d[、]|[一二三四五六七八九十][\.、]?)(?:&nbsp;|\s)*.{1,35})(?:&nbsp;|\s)*<\/(?:p|h\d)[^>]*>/;
    //ending当中一级标题：<p>或<hX> 摘要/结论/参考文献/研究成果/致谢  </p>或</hX>
    var endH1Reg = /<(?:p|h\d)[^>]*>(?:&nbsp;|\s)*((?:摘(?:&nbsp;|\s)*要|Abstract|结(?:&nbsp;|\s)*论|参(?:&nbsp;|\s)*考(?:&nbsp;|\s)*文(?:&nbsp;|\s)*献|研(?:&nbsp;|\s)*究(?:&nbsp;|\s)*成(?:&nbsp;|\s)*果|致(?:&nbsp;|\s)*谢))(?:&nbsp;|\s)*<\/(?:p|h\d)[^>]*>/;
    //i遍历除一级标题之外的其他节点的节点索引
    var i, html = "", inhtml = "";
    var childs = self.edit.doc.body.children;
    //搜索一级标题
    for (var j=0; j<childs.length; j++) {
      var outerhtml = childs[j].outerHTML;
      //排除公式插件插入的内容和空行段落
      if (/MathJax_Message/.test(outerhtml) || /<(?:p|h\d)[^>]*>(?:\s|&nbsp;|<br\s*[\/]?>)*<\/(?:p|h\d)[^>]*>/.test(outerhtml)) {
        html += outerhtml;
        continue;
      }
      //排除以上两种内容之后，再遍历到的就是正文内容
      if (mainbodyH1Reg.test(outerhtml) || endH1Reg.test(outerhtml)) {
        //取得匹配的一级标题正文内容，不含外围标签
        inhtml = RegExp.$1;
        html += '<h1 data-type="text" class="title-one">'+inhtml+'</h1>';
        //已经匹配到一级标题，将下一个节点作为遍历除一级标题之外的其他内容的开始节点，由下边循环开始遍历
        i = j+1;
        break;
      } else {
        //如果没有匹配到一级标题，说明没有添加一级标题，这个时候就将当前遍历到的节点作为除一级标题之外的内容的开始节点，由下边循环开始遍历
        i = j;
        break;
      }
    }
    for (i; i<childs.length; i++) {
      var outerhtml = childs[i].outerHTML;
      //如果节点是div，说明是多媒体节点，多媒体节点无需正则匹配
      if (childs[i].nodeName == "DIV") {
        html += outerhtml;
        continue;
      }
      //二级标题，<p>或<hX> 1.11/1．1/（一）/(一) XXXXX  </p>或</hX>  .有可能是中英文
      else if (/<(?:p|h\d)[^>]*>(?:&nbsp;|\s)*((?:\d{1}(?:&nbsp;|\s)*[\.．](?:&nbsp;|\s)*\d{1,2}|[\(（](?:&nbsp;|\s)*[一二三四五六七八九十]+(?:&nbsp;|\s)*[\)）])(?:&nbsp;|\s)*[^\.．].{1,35})(?:&nbsp;|\s)*<\/(?:p|h\d)[^>]*>/.test(outerhtml)) {
        inhtml = RegExp.$1;
        html += '<h2 data-type="text" class="title-two">'+inhtml+'</h2>';
      }
      //三级标题，<p>或<hX> 1.11.1/1．11.1 XXXXX  </p>或</hX>  .有可能是中英文
      else if (/<(?:p|h\d)[^>]*>(?:&nbsp;|\s)*(\d{1}(?:&nbsp;|\s)*[\.．](?:&nbsp;|\s)*\d{1,2}(?:&nbsp;|\s)*[\.．](?:&nbsp;|\s)*\d{1,2}(?:&nbsp;|\s)*[^\.．].{1,35})(?:&nbsp;|\s)*<\/(?:p|h\d)[^>]*>/.test(outerhtml)) {
        inhtml = RegExp.$1;
        html += '<h3 data-type="text" class="title-three">'+inhtml+'</h3>';
      }
      //文本是图或表标题 图1、图1-1、图1.1 图1. 1 XXXX
      else if (/<(?:p|h\d)[^>]*>(?:&nbsp;|\s)*(([图表])(?:&nbsp;|\s)*\d{1,2}(?:&nbsp;|\s)*[-\.．]?(?:&nbsp;|\s)*\d{0,2}(?:&nbsp;|\s)*.{1,35})(?:&nbsp;|\s)*<\/(?:p|h\d)[^>]*>/.test(outerhtml)) {
        inhtml = RegExp.$1;
        //添加默认内容
        html += _wrapMultimediaDiv(self, inhtml, RegExp.$2);
      }
      //这块处理的是图1 XXXX的情况，上边已经处理过了，想不起来还有什么特殊情况才添加的这个
      else if (/<(?:p|h\d)[^>]*>(?:&nbsp;|\s)*(([图表])(?:&nbsp;|\s)*\d{1,2}(?:&nbsp;|\s)*.{1,35})(?:&nbsp;|\s)*<\/(?:p|h\d)[^>]*>/.test(outerhtml)) {
        inhtml = RegExp.$1;
        html += _wrapMultimediaDiv(self, inhtml, RegExp.$2);
      }
      //文本是公式：（公式 1-1）、（公式 1.1）括号.有可能是中英文
      else if (/<(?:p|h\d)[^>]*>(?:&nbsp;|\s)*([\(（](?:&nbsp;|\s)*(公式)?(?:&nbsp;|\s)*\d{1,2}(?:&nbsp;|\s)*[-\.．](?:&nbsp;|\s)*\d{1,2}(?:&nbsp;|\s)*[\)）])(?:&nbsp;|\s)*<\/(?:p|h\d)[^>]*>/.test(outerhtml)) {
        inhtml = RegExp.$1;
        html += _wrapMultimediaDiv(self, inhtml, "公式");
      }
      else {
        //如果以上都不匹配说明是正文
        html += outerhtml;
      }
    }
    self.html(html);
  }
```
