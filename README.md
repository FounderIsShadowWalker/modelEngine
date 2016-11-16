#模版引擎
模版引擎这个工具 由来已久，据作者的记忆，接触模版引擎大约在两年前，初学 Node 的时候，接触了两款名声大噪的模版引擎 ejs 和 jade。 跳出前端来讲，我相信大家都学过jsp吧，jsp 本身就有模版引擎的功能。

放到现在，前端飞速发展，backbone 有 mustache，angular 把 模版引擎的功能 潜入指令中，vue 和 react 也是大同小异(react 不熟)，自己之前闲来无事，也写过一款模版引擎，就是混入了递归 对数据进行迭代，十分粗糙，今天意外看到了两款十分精致的模版引擎，其精妙处实在让人咂舌(大神求轻拍)

>jane的作品

直接上代码
<pre>
 var TemplateEngine = function(tpl, data){
        var re = /<%([^<%]+?)%>/g,
            reExp = /(^()?(if|for|else|switch|case|break|{|}))(.*)?/,
            code = 'var r=[];\n',
            cursor = 0,
            match;
            
   var add = function(line, js){
            js? code += line.match(reExp) ? line + '\n' : 'r.push(' + line + ');\n' :
                    code += 'r.push("' + line.replace(/"/g, '\\"') + '");\n';
            return add;
        }
        while(match = re.exec(tpl)){
            add(tpl.slice(cursor, match.index))(match[1], true);
            cursor = match.index + match[0].length;
        };

  add(tpl.substr(cursor, tpl.length - cursor));
        code += 'return r.join("")';
        return new Function(code.replace(/[\t\r\n]/, '')).apply(data);
  }
</pre>

使用也异常简单

	var template =
            'My skills:' +
            '<%if(this.showSkills) {%>' +
            '<%for(var index in this.skills) {%>' +
            '<a href="#"><%this.skills[index]%></a>' +
            '<%}%>' +
            '<%} else {%>' +
            '<p>none</p>' +
            '<%}%>';

  		console.log(TemplateEngine(template, {
        skills: ["js", "html", "css"],
        showSkills: true
    }));
    
  	对应的结果 My skills:<a href="#">js</a><a href="#">html</a><a href="#">css</a>  
  	
  	
##实现思路 	
首先我们简单只用正则替换，在应对this或者for，if，else 的时候，是需要js 运行环境的。所以要么 eval，要么new Function 动态编译，显然后面的方法好，可以返回一个fn多灵活，而正则表达式 就完美解决了 if else for 等动态语句，至于this，在fn里自然有this了，但是传给new Function的 始终是个 字符串啊，就算运行的时候会剥掉‘’(保证了 this 可以存活在字符串中)。但是for if else 怎么办？ “console.log(111);”for 会报错. 所以作者思路如下:

+ 拿数组采集 不涉及 for if else switch 的 普通语句
+ 数组采集的 带this的 数据项 不带'
+  返回 数组join后的模版 




第二版
> 引用马超 同学的 https://github.com/machao/tmpl

模版引擎短小精悍，内里玄妙无穷，其实仔细一想也稀疏平常
直接上代码
<pre>
(function($) {
    var tmplCache={}, fnCache={}, guid=0, toString = Object.prototype.toString, compile = function( tmpl, sp ){
        //默认分隔符
        var f = sp || "%",
            //动态创建函数，并增加数据源引用（data/my）
            fn = new Function("var p=[],my=this,data=my,print=function(){p.push.apply(p,arguments);};p.push('" +
                // Convert the template into pure JavaScript
                tmpl
                    .replace(/[\r\t\n]/g, " ")
                    .split("<" + f).join("\t")
                    .replace(new RegExp("((^|" + f + ">)[^\\t]*)'", "g"), "$1\r")
                    .replace(new RegExp("\\t=(.*?)" + f + ">", "g"), "',$1,'")
                    .split("\t").join("');")
                    .split(f + ">").join("p.push('")
                    .split("\r").join("\\'") + "'); return p.join('');");
        return fn;
    };
    //对外接口
    $.template = function(tmpl, data, sp) {
        sp = sp||"%";''
        var fn = toString.call(tmpl) === "[object Function]" ? tmpl
                : !/\W/.test(tmpl) ? fnCache[tmpl+sp] = fnCache[tmpl+sp] || compile(document.getElementById(tmpl).innerHTML, sp)
                : (function(){
                    for(var id in tmplCache)if( tmplCache[id] === tmpl ) return fnCache[id];
                    return (tmplCache[++guid] = tmpl, fnCache[guid] = compile(tmpl, sp));
                })();
        return data ? fn.call(data) : fn;
    };
})(window.jQuery || window.Zepto || window);
</pre>

##实现思路
1. 找到 <% 替换成制表符(作者认为这里是 结束)
2. 替换掉 %>可能 出现的 ' 引号
3. 找到 =   %>  这里是要被push 换成 ， ，不带引号的部分，如第一种方法的思路
4. 将上面的 <% 换成 ');  也就是push的 收尾
5. 将 %> 替换成 push('; 也就是push的开始

**其实作者思路很简单 作者始终就在分隔<% %> 只是我们习惯 <% 这里作为开始 作者在前面补了 push 的开始， 后面不上了 push 的 收尾 完美契合 想象一下 
%> <% %> <% 这样是不是很奇怪呢，^-^**

作者还引入了缓存的功能，因为比较简单，这里不做介绍啦
PS: 作者还引入了Qunit 测试，可见作者态度严谨，以此致敬。


文件目录

method1: 思路1
method2: 思路2

***
##### 我是爱读源码的顺顺
![Image](https://github.com/FounderIsShadowWalker/modelEngine/blob/master/founder.jpg)

