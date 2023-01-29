---
title: jquery怎样获取所有选中的checkbox的值
date: 2023-01-29 20:45:00 +0800
categories: [前端]
tags: []
pin: false
---

> 撰写时间：2022-01-18，整理时间：2023-01-29

界面结构如下

```html
<input name="selector[]" id="ad_Checkbox1" class="ads_Checkbox" type="checkbox" value="1" />
<input name="selector[]" id="ad_Checkbox2" class="ads_Checkbox" type="checkbox" value="2" />
<input name="selector[]" id="ad_Checkbox3" class="ads_Checkbox" type="checkbox" value="3" />
<input name="selector[]" id="ad_Checkbox4" class="ads_Checkbox" type="checkbox" value="4" />
<input type="button" id="save_value" name="save_value" value="Save" />
```

添加点击事件

```javascript
$(function(){
  $('#save_value').click(function(){
       
  });
});
```

在事件的回调方法中添加以下代码

```javascript
$('input[type=checkbox]:checked').map(function(_, el) {
    return $(el).val();
}).get();
```
