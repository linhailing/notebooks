# js使用技巧 

+ 阻止冒泡事件
```js
 <script type="text/javascript">  
        function fun1 () {  
            alert("parent");  
        }  
        function funx(){  
            alert("child");  
        }
        function stopEvt(e) {  
            e.stopPropagation();//阻止点击事件向上冒泡  
        }   
    </script>
<div onclick="fun1()" style="height:100px;background-color:black;color:white;text-align:center;border:1px solid red">  
        父  
        <div onclick="funx();stopEvt(event)" style="background-color:green;margin-top:20px;border:1px solid red;height:30px">  
            子  
        </div>  
    </div>  
``````


