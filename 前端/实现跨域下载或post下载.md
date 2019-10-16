

如果在自己的页面，访问其它域名实现下载可使用此方式。

还有再自己页面通常下载是使用window.location.href 类似于get的方式去下载，产生的问题是，传输汉字的时候，编码不好调整，我这边不论如何编码在后台都乱码，或者uat是正常的，一放到线上就乱码。调整为post方式



```javascript
/**
通过在页面构造一个ifram来实现跨域访问或post方式下载
**/
function downLoadFile(options){
//    var config = $.extend(true, { method: 'post' }, options);  
   var $iframe = $('<iframe id="down-file-iframe" />'); 
   var $form = $('<form target="down-file-iframe" method="post" />');  
   $form.attr('action', "${dmsexport}/comprehensiveQry_exportComprehensiveQry.do");  
   for (var key in options) {  
//        $form.append('<input type="hidden" name="' + key + '" value="' + options[key] + '" />');
       $form.append("<input type='hidden' name='" + key + "' value='" + options[key] + "' />");
   }  
   $iframe.append($form); 
   $(document.body).append($iframe);  
   $form[0].submit();  
   $iframe.remove(); 
};
```

