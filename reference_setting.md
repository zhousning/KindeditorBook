# 参考文献设置

## 简介

参考文献格式要求：间断的用,   连续的用-

​	1,2,3   ——>   [1-3]

​	4,6,8   ——>   [4,6,8]

​	组合  [1-3,4,6,8]

注：mainbody.js

​	

## 创建工具

```javascript
  var insert_references = document.getElementById("insert-references")
  if(insert_references){
    //给插入参考文献按钮绑定事件
    insert_references.onclick = function(){
      var references = document.getElementsByName("references");
      var ref_selected = [], //被选中的参考文献列表
          ref_backup = null;//使用这个作为处理目标，不使用原内容
      var cmd = kindeditor.edit.cmd;
      var range = cmd.range;
      var ref_insert = null;//分割后的结果
      ref_selected = getSelectedCheckBox(references); 
      //如果没有被选中的参考文献，则终止执行
      if(ref_selected.length == 0){
        return;
      }

      ref_backup = ref_selected; 
      ref_backup.push(1000);//添加哨兵
      ref_insert = refSegment(ref_backup);
	 //将分割后的结果作为普通文本节点插入到光标所在位置
      var ref_node = document.createTextNode(ref_insert);
      range.insertNode(ref_node); 
      //以插入的文本节点作为选区
      range.selectNode(ref_node);
      //使用KE提供的superscript上标函数设置上标样式
      cmd.superscript();
      //将光标定位在选区的结尾
      cmd.sel.collapseToEnd();
	
      //清空复选框
      var references = document.getElementsByName("references");
      clearCheckBox(references);
    };
  }
```

## 分割算法

```javascript
function refSegment(ref_selected){
  var start = end = ref_selected[0], str = "", result = "[";
  if(ref_selected.length !== 1){
    for(var j=1; j<ref_selected.length; j++){
      if(ref_selected[j]-ref_selected[j-1] == 1){
        end = ref_selected[j];
      }else{
        if(end - start > 0){
          str += start + "-" + end + ","; 
        }else{
          str += start + ",";
        }
        start = end = ref_selected[j];
      }
    }
  }else{
    result += ref_selected[0] + ",";
  }
  result += str;
  result = result.slice(0,result.length-1);
  result += "]";
  return result;
}

哨兵：1000
flag：start，end，使用flag作为拼接结果
index：j
如：1,2,3,5,7,8,1000
  start
  end
```