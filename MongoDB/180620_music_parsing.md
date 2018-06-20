## 180620

- 문제 해결  
실행시 오류나는 사람들 : maven 의존성 문제  
패키지 그대로 가져와서 작동하는 프로젝트(spring_15_WEBmybatis)에 붙여넣기 하고  
pom.xml에 mongodb와 json 관련 dependencies만 넣어 주기  

- music.jsp  
(modal 투명한 것, 하드코딩 고쳐야 함)  

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@page import="org.slf4j.Logger"%>
<%@page import="org.slf4j.LoggerFactory"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt"%>
<%
	Logger log = LoggerFactory.getLogger(this.getClass());
	log.debug("------------------------------------");
	log.debug("this.getClass()=" + this.getClass());
	log.debug("------------------------------------");
%>
<%--CONTEXT --%>
<c:set var="CONTEXT" value="${pageContext.request.contextPath}" />

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1">
<!-- 위 3개의 메타 태그는 *반드시* head 태그의 처음에 와야합니다; 어떤 다른 콘텐츠들은 반드시 이 태그들 *다음에* 와야 합니다 -->
<title>MUSIC</title>

<!-- 부트스트랩 -->
<link href="${CONTEXT}/resources/css/bootstrap.min.css" rel="stylesheet">

<!-- IE8 에서 HTML5 요소와 미디어 쿼리를 위한 HTML5 shim 와 Respond.js -->
<!-- WARNING: Respond.js 는 당신이 file:// 을 통해 페이지를 볼 때는 동작하지 않습니다. -->
<!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
      <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->


</head>
<body>
<style type = "text/css">
	.row {
		margin: 0px auto;
		width: 800px;
	}
	h2 {
		text-align: center;
	}
	
</style>
<div class = "container">
	<h2>MUSIC TOP 200</h2>
	<!--  검색  -->
	<div class = "row">
		<div class = "form-group">
			<div class = "col-sm-10">
				<input type = "text" class = "form-control" id = "data" placeholder = "검색어를 입력" />
			</div>
				<input type = "button" value = "검색" id = "searchBtn" class = "btn btn-sm btn-success"/>
		</div>
	</div>
  	<!--  검색  -->
  
    <!--  Table -->
    	<table class = "table table-hover" id = "user-table">
    		<thead>
    			<tr class = " danger">
    				<th>순위</th>
    				<th></th>
    				<th>곡명</th>
    				<th>가수</th>
    				<th></th>
    			</tr>
    		</thead>
    		<tbody id = "music">
    		
    		</tbody>
    	</table>
    <!--  Table -->
</div>

	<!--  동영상 Layer popup -->
<div id = "dialog" title = "" style = "display:none;">
		<table class = "table">
			<tr>
				<td width="50%" class = "text-center" rowspan = "3">
						<embed src = ""  width = "300" height = "250" id = "key" >
				</td>
				<td colspan = "2"class = "text-center" id = "title">
				</td>
			</tr>
			<tr>
				<td width = "20%" class = "text-right">순위</td>
				<td width = "30%" class = "text-left" id = "rank"></td>
			</tr>
			<tr>
				<td width = "20%" class = "text-right">가수명</td>
				<td width = "30%" class = "text-left" id = "singer"></td>
			</tr>

		</table>
</div>
	<!--  동영상 Layer popup -->

<script src="${CONTEXT}/resources/js/jquery-1.12.4.js"></script>
<script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>
<script type="text/javascript">

function music_show(rank){
	var keys = "htttp://www.youtube.com/embed/IHNzOHi8sJs";
	
    $("#dialog").dialog({
    	modal:true,
    	width:'600',
    	height:'450'
    	
    });
	
	$("#key").attr("src",keys);
	$("#title").text("뚜두뚜두");
	$("#rank").text("1");
	$("#singer").text("BLACKPINK");
	
}

$(document).ready(function(){
		alert('ready?');	
		
		//--ajax
		$.ajax({
	         type:"GET",
	         url:"${CONTEXT}/music/do_selectList.do",
	         dataType:"html",
	         async: false,
	         data:{
	        	 "data" : $("#data").val()
	         },
	         success: function(data){//통신이 성공적으로 이루어 졌을때 받을 함수

	         	var list = $.parseJSON(data);
	         	var listLen  =list.length;
	         	
	         	console.log("listLen=" + listLen);
	         	
	         	if(listLen == 0){
	         		alert("데이터가 없습니다");
	         	} else {
	         		$.each(list,function(key,value){
	         			//console.log(value.rank + "|" + value.singer + "|" + value.title);
	         			
	         			$("#music").append(
	         					"<tr>"
	         					+"<td>"+value.rank+"</td>"
	         					+"<td>"+"<img src=http:" + value.poster + " width = '30' height = '30'>" + "</td>"
	         					+"<td>"+value.title+"</td>"
	         					+"<td>"+value.singer+"</td>"
	         					+"<td>"+"<input type='button' class = 'btn btn-sm btn-primary' value='동영상' onclick='music_show("+ value.rank + ")'></td>"
	         					+"</tr>"
	         			);//append

	         		});//each
	         	}//ifelse
	         	
	         	
	         },
	         complete: function(data){//무조건 수행
	          
	         },
	         error: function(xhr,status,error){
	          
	         }
	        }); //--ajax

});

</script>

</body>

</html>
```
