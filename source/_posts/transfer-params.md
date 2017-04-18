---
title: 前后台传参方式
date: 2017-04-18 16:07:05
categories: js
tags: 
	- ajax
---
记录一下几种前台传参到后台的方式: 
<!--more-->
1.  
```
function xxx(item,entity) {
   cargoEntity = entity;
    $.ajax(
      {
        type: 'POST',
        url: '${ctx}/pipeiliebiao/tBizPplb/goodsMatchList' ,
        data:{
         item : item
        },
       success: function(data){
    if('' == data) {
     alert("没有匹配信息 ");
     $("#shipList").html('');
     return false;
    }
       for(var a = 0 ; a < data.length; a++) {
         var item = data[a];
         shipEntity=data[0];
         var str = "<tr><td><input type=\"radio\" value = \""+item.id+"\" name = \"goods\" ></td>";
          str += "<td>"+item.salesMan+"</td>";
       str += "<td>"+item.partnerName+"</td>";
      str += "<td>"+item.hl+"</td>";
      str += "<td>"+item.zhd+"</td>";
      str += "<td>";
      str += item.kbksrq;
      str += "</td>";
      str += "<td>"+item.hcxhg+"</td>";
      str += "<td>"+item.yj+"元/t</td> "
     str += "<td>"+(item.ppzt==0?'匹配':'未匹配')+"</td>"
     str += "<td>"+item.contactName+"</td>"
     str += "<td>"+item.fbsj+"</td>"
     str += "</tr>";
     $("#shipList").html(str);
        }
       },
       error:function(){
        alert("没有匹配信息 ");
       
       },
    // dataType: 'json'
   });
  };
```

2.  
```
$.post('${ctx}/pipeiliebiao/tBizPplb/profitCalPage',{item:item});
```

这俩种是通过ajax方法往后台传值,进行前后台局部的动态交互,
但是这样就必须有回调函数,回调函数返回之后才会进行页面跳转,所以希望传值到后台跳转到别的页面是不行的,因为后台处理是再回调函数之前执行的. 

3.单纯的传参到后台进行处理:
一般有form表单模式,不过这个再js里面无法用.
可以这样:

```
window.location.href="${ctx}/pipeiliebiao/tBizPplb/profitCalPage?item="+item;
```

这样就能单纯的将参数传到后台,后台就可以处理后不管前端的东西了.
