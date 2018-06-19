#### ajax 통신할 때 로딩중 띄우기  

- html  
```html
<div id = "loadingImage" style="display:none"><img src="...."></div>
<!-- ajax가 로딩되면 데이터를 넣을 위치 -->
```


- js  

```javascript
<script type = "text/javascript">
function showList(){

$("#loadingImage").show();

$.ajax({
  	type:"GET",
		async: false,
		url:"...",
		dataType:"html",
		data:null,
 		success:function(data){//성공
    //
    },
 		complete:function(data){//무조건
 		    $("#loadingImage").hide();
		},
 		error:function(xhr,status,error){//실패시
    }
}

</script>
```
