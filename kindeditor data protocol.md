# Data Protocol

# 说明

1. 数据要求：两层嵌套，只保留数据协议需要标签，清除非必要标签
2. 浏览器：是浏览器显示的数据内容	数据库：是数据库里存储的数据内容
3. [ ] 标识浏览器需要，数据库不需要的部分

## 文本

```html
------ 浏览器 ------											------ 数据库 ------
<h1 data-type="text" [ class="title-one" ]>一级标题</h1>  	<h1 data-type="text">一级标题</h1>
<h2 data-type="text" [ class="title-two" ]>二级标题</h2> 	<h2 data-type="text">二级标题</h2>
<h3 data-type="text" [ class="title-three" ]>三级标题</h3>	<h3 data-type="text">三级标题</h3>
<p>正文</p>												 <p>正文</p>
<sup>上标</sup>											 <sub>下标</sub>
```



##  多媒体

### 工具条(不保存)

```html
<div class="kemd-operate-container">
	<a class="kemd-operate-button kemd-edit-button"> </a> 
	<a class="kemd-operate-button kemd-zoomin-button"> </a> 
	<a class="kemd-operate-button kemd-zoomout-button"> </a> 
	<a class="kemd-operate-button kemd-moveup-button"> </a> 
	<a class="kemd-operate-button kemd-movedown-button"> </a> 
	<a class="kemd-operate-button kemd-delete-button"> </a>
</div>
```

### 表格

```html
------ 浏览器 ------
<div data-type="table" data-meta="html" contenteditable="false" [ class="kemd-container" ]>
	= 工具条
	<p data-type="table_caption" [ class="kemd-multimedia-caption kemd-table-caption" ]>表1-1 表格标题</p>
	<table data-type="table_entity" width="100%" contenteditable="true" [ class="kemd-multimedia kemd-table" ]>	
		<tbody>
			<tr>
              	<td><p>表格内容</p></td>
				<td><p><br></p></td>
			</tr>
		</tbody>
	</table>
</div>

------ 数据库 ------
<div data-type='table' data-meta='html' contenteditable='false'>
  <p data-type='table_caption'>表2-1 333</p>
  <table data-type='table_entity' width='100%' contenteditable='true'>
  	<tr>
		<td><p><br/></p></td>
		<td><p><br/></p></td>
	</tr>
  </table>
</div>
```

### 图片

```html
------ 浏览器 普通图片------
<div data-type="image" data-meta="image_com" contenteditable="false" [ class="kemd-container" ]>
  = 工具条
  <img data-type="image_entity" src="xxx.jpg" data-src="xxx.jpg" title="444" width="60%" [ data-ke-src="xxx.jpg" class="kemd-multimedia kemd-image" ]>
  <p data-type="image_caption" class="kemd-multimedia-caption kemd-image-caption">图1-1 图片标题</p>
</div>

------ 数据库 普通图片 ------
<div data-type='image' data-meta='image_com' contenteditable='false'>
  <img data-type='image_entity' src='xxx.jpg' data-src='xxx.jpg' title='3333' width='60%'/>
  <p data-type='image_caption'>图2-1 3333</p>
</div>

-------------------------------------------------------------------------------------------------------

------ 浏览器 visio图片 浏览器支持显示------
<div data-type="image" data-meta="image_win" contenteditable="false" [ class="kemd-container" ]>
  <img data-type="image_entity" src="xxx.emf" data-src="xxx.emf" title="图1-1 图片标题"  width="60%" data-size="5780405;3347720;;" [ data-ke-src="xxx.emf" class="kemd-multimedia kemd-image" ]>
  <p data-type="image_caption" [ class="kemd-multimedia-caption kemd-image-caption" ]>图1-1 图片标题</p></div>

------ 浏览器 visio图片 浏览器不支持显示------
<div data-type="image" data-meta="image_win" contenteditable="false" [ class="kemd-container" ]>
  =	工具条
  <img data-type="image_entity" src="default_win.jpg" data-src="xxx.emf"  title="图1-1 图片标题" width="60%"  data-size="5780405;3347720;;" [ data-ke-src="default_win.jpg" class="kemd-multimedia kemd-image" ]>
  <p data-type="image_caption" [ class="kemd-multimedia-caption kemd-image-caption" ]>图1-1 图片标题</p></div>

------ 数据库 visio图片 ------
<div data-type='image' data-meta='image_win' contenteditable='false'>
  <img data-type='image_entity' src='xxx.emf' data-src='xxx.emf' title='图1-1 图片标题' width='60%' data-size="5780405;3347720;;"/>
  <p data-type='image_caption'>图1-1 图片标题</p>
</div>
```

### 公式

```html
------ 浏览器 普通公式 ------
<div data-type="formula" data-meta="mathml" contenteditable="false" [ class="kemd-container" ]>
  	= 工具条
	<div data-type="formula_entity" [ class="kemd-multimedia kemd-formula" ]>公式内容</div>
	<p data-type="formula_caption" [ class="kemd-multimedia-caption kemd-formula-caption" ]>(1-1)</p>
</div>

------ 数据库 ------
<div data-type='formula' data-meta='mathml' contenteditable='false'>
  <div data-type='formula_entity'>公式内容</div>
  <p data-type='formula_caption'>(1-1)</p>
</div>

------ 备注：浏览器 公式图片 ------
以前使用图片代替公式显示，现在直接渲染显示公式，这个是作为备注，知道就可以
```

