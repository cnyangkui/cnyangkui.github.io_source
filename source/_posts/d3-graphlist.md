---
title: D3-v3图表绘制
categories: [ Engineering, Visualization ]
tags: D3
date: 2017/11/15 17:30:00
---

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/07/layout.png)

本节内容将描述饼状图、力导向图、弦图、集群图、树状图、打包图、分区图、圆形分区图、直方图、捆图、堆栈图、矩阵树图、地图的绘制过程，使用d3-v3，参考[D3.js入门系列](http://www.ourd3js.com/wordpress/396/)

温馨提示：对于有D3基础的人，本节内容能够帮助其快速掌握各图表的绘制。若没有掌握基础知识，不建议直接学习本节内容。

对图表绘制的重点内容进行了总结，下述图表绘制步骤相似，总结如下：

> 1.使用布局，转换数据格式
> 2.如需绘制复杂path，需要创建对应的路径生成器
> 3.依次绘制各个元素，如需绘制path可能需要调用路径生成器

布局内容总结：

> D3总共提供了12个布局：饼状图（Pie）、力导向图（Force）、弦图（Chord）、树状图（Tree）、集群图（Cluster）、捆图（Bundle）、打包图（Pack）、直方图（Histogram）、分区图（Partition）、堆栈图（Stack）、矩阵树图（Treemap）、层级图（Hierarchy）。
>
> 12 个布局中，层级图（Hierarchy）不能直接使用。集群图、打包图、分区图、树状图、矩阵树图是由层级图扩展来的。如此一来，能够使用的布局是11个（有5个是由层级图扩展而来）。这些布局的作用都是将某种数据转换成另一种数据，而转换后的数据是利于可视化的。

<!--more-->

## 饼状图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/07/9111.png)

#### 1.数据格式
```javascript
var dataset = [ 30 , 10 , 43 , 55 , 13 ];
```

#### 2.使用pie布局,转换数据

```javascript
var pie = d3.layout.pie();
var piedata = pie(data);
```

转换后的数据：

> data、startAngle、endAngle、padAngle、value

#### 3.绘制

弧生成器计算路径（svg的path）

```javascript
var arc = d3.svg.arc()  //弧生成器
	.innerRadius(innerRadius)   //设置内半径
	.outerRadius(outerRadius);  //设置外半径
```

绘制路径path，需要调用弧生成器

```javascript
.attr("d", function(d){
    return arc(d);   //调用弧生成器，得到路径值
})
```

或者

```javascript
.attr("d", arc)
```

绘制文本text，计算路径中心位置，放入文本值

```javascript
.attr("transform",function(d){
    return "translate(" + arc.centroid(d) + ")";
})
```

## 力导向图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/07/force1.png)

#### 1.数据格式

```javascript
var nodes = [ { name: "桂林" }, { name: "广州" },
	{ name: "厦门" }, { name: "杭州" },
    { name: "上海" }, { name: "青岛" },
    { name: "天津" } ];

var edges = [ { source : 0 , target: 1 } , { source : 0 , target: 2 } ,
	{ source : 0 , target: 3 } , { source : 1 , target: 4 } ,
	{ source : 1 , target: 5 } , { source : 1 , target: 6 } ];
```

#### 2.force布局，转换数据

```javascript
var force = d3.layout.force()
      .nodes(nodes) //指定节点数组
      .links(edges) //指定连线数组
      .size([width,height]) //指定作用域范围
      .linkDistance(150) //指定连线长度
      .charge([-400]); //相互之间的作用
```

转换后的数据：

> 节点：index(节点的索引号)、px、py(节点上一时刻的坐标)、x、y(现在的坐标)、weight(节点权重)、name
>
> 连线：source、target

然后，使力学作用生效：

```javascript
force.start();    //开始作用
```

#### 3.绘制

* line,连线
* circle,节点
* text,文字

绘制节点时，使节点拖动

```javascript
.call(force.drag);  //使得节点能够拖动
```

力导向图布局force有一个事件tick，每进行到一个时刻，都要调用它，更新的内容就写在它的监听器里就好。

```javascript
force.on("tick", function(){ //对于每一个时间间隔
    //更新连线坐标
    svg_edges.attr("x1",function(d){ return d.source.x; })
        .attr("y1",function(d){ return d.source.y; })
        .attr("x2",function(d){ return d.target.x; })
        .attr("y2",function(d){ return d.target.y; });

    //更新节点坐标
    svg_nodes.attr("cx",function(d){ return d.x; })
        .attr("cy",function(d){ return d.y; });

    //更新文字坐标
    svg_texts.attr("x", function(d){ return d.x; })
       .attr("y", function(d){ return d.y; });
 });
```

## 弦图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/07/chord1.png)

#### 1.数据格式

```javascript
var city_name = [ "北京" , "上海" , "广州" , "深圳" , "香港"  ];
    
var population = [
	[ 1000,  3045, 4567, 1234, 3714 ],
    [ 3214,  2000, 2060, 124, 3234 ],
    [ 8761,  6545, 3000, 8045, 647 ],
    [ 3211,  1067, 3214, 4000, 1006 ],
    [ 2146,  1034, 6745, 4764, 5000 ]
];
```

#### 2.chord布局

```javascript
var chord_layout = d3.layout.chord()
	.padding(0.03) //节点之间的间隔
  	.sortSubgroups(d3.descending) //排序
  	.matrix(population); //输入矩阵
```

数据转换

```javascript
var groups = chord_layout.groups();
var chords = chord_layout.chords();
```

转换后的数据：

> 节点：angle、startAngle、endAngle、index、name、value
>
> 连线： source、target

#### 3.绘制

首先绘制外部圆环，使用arc弧生成器

```javascript
var outer_arc = d3.svg.arc()
	.innerRadius(innerRadius)
	.outerRadius(outerRadius);
```

绘制path，使用数据为groups

```javascript
.attr("d", outer_arc)	//调用arc路径生成器
```

绘制text,使用数据为groups。首先旋转适当角度，然后向上移动外半径个长度，135-225度范围的文字倒置

```javascript
.each( function(d,i) { 
   d.angle = (d.startAngle + d.endAngle) / 2; 
   d.name = city_name[i];
})
.attr("transform", function(d) {
   return "rotate(" + ( d.angle * 180 / Math.PI ) + ")" +
   	"translate(0,"+ -1.0*(outerRadius+10) +")" +
   	(( d.angle > Math.PI*3/4 && d.angle < Math.PI*5/4 ) ? "rotate(180)" : "");
})
```

然后，绘制弦，使用chord生成器生成路径

```javascript
var inner_chord = d3.svg.chord()
	.radius(innerRadius);
```

绘制path,使用数据为chords

```javascript
.attr("d", inner_chord)	//调用chord路径生成器
```

## 集群图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/08/cluster.png)

#### 1.数据格式

```javascript
{
	"name":"中国",
	"children":
	[
	    { 
	      "name":"浙江" , 
	      "children":
	      [
	            {"name":"杭州" },
	            {"name":"宁波" },
	            {"name":"温州" },
	            {"name":"绍兴" }
	      ] 
	    },
	    { 
	        "name":"广西" , 
	        "children":
	        [
	            {"name":"桂林"},
	            {"name":"南宁"},
	            {"name":"柳州"},
	            {"name":"防城港"}
	        ] 
	    },
	    ......
	]
}
```

#### 2.定义集群图布局，转换数据

```javascript
var cluster = d3.layout.cluster()
	.size([width, height - 200]);	//设定尺寸

{% endhighlight %}
```

使用数据

```javascript
d3.json("city.json", function(error, root) {
  var nodes = cluster.nodes(root);
  var links = cluster.links(nodes);
}
```

转换后的数据：

> 节点：children(Array)、depth、name、x、y
>
> 连线：source、target

#### 3.绘制

首先，创建对角线生成器

```javascript
var diagonal = d3.svg.diagonal()
	.projection(function(d) { return [d.y, d.x]; });
```

绘制连线，使用生成器

```javascript
.attr("d", diagonal)   //使用对角线生成器
```

然后绘制节点circle。


## 树状图

和集群图写法基本相同，使用布局不同。

```javascript
var tree = d3.layout.tree()
	.size([width, height-200])
	.separation(function(a, b) { return (a.parent == b.parent ? 1 : 2); });
```

转换后的数据与集群图相同。

## 打包图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/08/pack.png)

数据格式与树状图相同，布局如下：

```javascript
var pack = d3.layout.pack()
	.size([ width, height ])
	.radius(20);
```

数据转换写法也类似。

转换后的数据：

> 节点：children、depth、name、r(半径)、value、x、y(圆心位置)
>
> 连线：source、target

然后分别绘制circle和text。


## 分区图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/11/301.png)

数据类型与集群图、树状图、打包图相同。用于表示包含与被包含关系的。布局如下：

```javascript
var partition = d3.layout.partition()
	.sort(null)	//sort设定内部的顶点的排序函数，null表示不排序
	.size([width,height])	//size设定转换后图形的范围
	.value(function(d) { return 1; });	//value设定表示分区大小的值
```

value设定表示分区大小的值。这里的意思是：如果数据文件中用size值表示结点大小，那么这里可写成return d.size。

数据转换写法也类似。（nodes、links）

转换后的数据：

> 节点：children、depth、dx(宽度)、dy(高度)、name、value、x、y(坐标)
>
> 连线：source、target

然后绘制rect和text。rect的width、height属性分别对应数据属性dx、dy。


## 圆形分区图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/11/311.png)

#### 1.数据格式

与前面相同。

#### 2.布局

```javascript
var partition = d3.layout.partition()
	.sort(null)
	.size([2 * Math.PI, radius * radius])  
	.value(function(d) { return 1; });
```

数据转换写法也类似。（nodes、links）

转换后的数据：

> 节点：children、depth、dx(角度)、dy(向外宽度)、name、value、x(绕圆心方向的起始位置)、y(由圆心向外方向的起始位置)
>
> 连线：source、target

#### 3.绘制

分别绘制path和text。

首先创建弧生成器。

```javascript
var arc = d3.svg.arc()
	.startAngle(function(d) { return d.x; })
	.endAngle(function(d) { return d.x + d.dx; })
	.innerRadius(function(d) { return Math.sqrt(d.y); })
	.outerRadius(function(d) { return Math.sqrt(d.y + d.dy); });
```

绘制path使调用弧生成器。

```javascript
.attr("d", arc)
```

绘制text时要进行一下转换：

```javascript
.attr("transform",function(d,i){
	//第一个元素（最中间的），只平移不旋转
	if( i == 0 )
		return "translate(" + arc.centroid(d) + ")";
	
	//其他的元素，既平移也旋转
	var r = 0;
	if( (d.x+d.dx/2)/Math.PI*180 < 180 )  // 0 - 180 度以内的
		r = 180 * ((d.x + d.dx / 2 - Math.PI / 2) / Math.PI);
	else  // 180 - 360 度以内的
		r = 180 * ((d.x + d.dx / 2 + Math.PI / 2) / Math.PI);
		
	//既平移也旋转
	return  "translate(" + arc.centroid(d) + ")" +
			"rotate(" + r + ")";
}) 
```

## 直方图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/12/501.png)

#### 1.数据格式：数组

#### 2.布局

```javascript
var histogram = d3.layout.histogram()
	.range([-50, 50])	//区间的范围
	.bins(bins_num)	//矩形数量
	.frequency(true);	//若值为true，则统计的是个数；若值为false，则统计的是概率
```

数据转换

```javascript
var data = histogram(dataset);
```

转换后的数据：

> x(区间的起始位置)、dx(区间的宽度)、y(
> 如果frequency为true，则是落到此区间的数值的数量；如果frequency为false，则是落到此区间的概率)

#### 3.绘制

矩形rect、坐标轴line、刻度line、文字text。

## 捆图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2015/03/201.png)

#### 1.数据格式

```javascript
var cities = {
     name: "",
     children:[
        {name: "北京"},{name: "上海"},{name: "杭州"},
        {name: "广州"},{name: "桂林"},{name: "昆明"},
        {name: "成都"},{name: "西安"},{name: "太原"}
     ]
 };
 var railway = [
    {source: "北京", target: "上海"},
    {source: "北京", target: "广州"},
    {source: "北京", target: "杭州"},
    {source: "北京", target: "西安"},
    {source: "北京", target: "成都"},
    {source: "北京", target: "太原"},
    {source: "北京", target: "桂林"},
    {source: "北京", target: "昆明"},
    {source: "北京", target: "成都"},
    {source: "上海", target: "杭州"},
    {source: "昆明", target: "成都"},
    {source: "西安", target: "太原"}
]; 
```

#### 2.布局

需要使用集群图布局和捆图布局。

```javascript
var cluster = d3.layout.cluster()
	.size([360, width/2-50]);	//首先使用集群图布局，360表示角度，width/2-50 表示捆图圆半径

	var bundle = d3.layout.bundle();	//捆图布局

	var nodes = cluster.nodes(cities);	//数据转换
	console.log(nodes);

	function map(nodes, links) {
		var hash = [];
		for(var i=0; i<nodes.length; i++) {
			hash[nodes[i].name] = nodes[i];
		}
		var resultLinks = [];
		for(var i=0; i<links.length; i++) {
			resultLinks.push({source:hash[links[i].source], target:hash[links[i].target]});
		}
		return resultLinks;
	}

	var oLinks = map(nodes, railway);
	console.log(oLinks);

	var links = bundle(oLinks);	//数据转换
	console.log(links);
```

转换后的数据：

> nodes-节点：name、depth、parent、x、y
>
> oLinks-节点连线：source、target
>
> links-捆图线条数据：Array(Object(name、depth、parent、children、x、y))

#### 3.绘制

创建放射式的线段生成器：

```javascript
var line = d3.svg.line.radial()
	.interpolate("bundle")	//在line.interpolate()所预定义的插值模式中，有一种就叫做bundle，正是为捆图准备的。
	.tension(.85)	//拉伸度0.85。变小线的弯曲度会变小，会使线与线之间的距离增大
	.radius(function(d) {return d.y;})
	.angle(function(d) {return d.x/180*Math.PI;})
```

首先绘制path，调用线段生成器:

```javascript
.attr("d", line);	//调用线段生成器
```

接着创建g，在g中绘制circle和text

绘制g需要旋转、平移：

```javascript
.attr("transform", function(d) {
	return "rotate("+(d.x-90)+")"+
		"translate("+(0, d.y)+")";
});
```

## 堆栈图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2015/03/301.png)

#### 1.数据格式

```javascript
var dataset = [
	{ name: "PC" , 
	  sales: [	{ year:2005, profit: 3000 },
				{ year:2006, profit: 1300 },
				{ year:2007, profit: 3700 },
				{ year:2008, profit: 4900 },
				{ year:2009, profit: 700 }] },
	{ name: "SmartPhone" , 
	  sales: [	{ year:2005, profit: 2000 },
				{ year:2006, profit: 4000 },
				{ year:2007, profit: 1810 },
				{ year:2008, profit: 6540 },
				{ year:2009, profit: 2820 }] },
	{ name: "Software" , 
	  sales: [	{ year:2005, profit: 1100 },
				{ year:2006, profit: 1700 },
				{ year:2007, profit: 1680 },
				{ year:2008, profit: 4000 },
				{ year:2009, profit: 4900 }]}
	];
```

#### 2.布局

创建堆栈图布局

```javascript
var stack = d3.layout.stack()
	.values(function(d) {return d.sales;})
	.x(function(d) {return d.year;})
	.y(function(d) {return d.profit});
```

数据转换

```javascript
var data = stack(dataset);
```

转换后的数据：

> name、Array(y0(纵坐标)、y(高度)、(year、profit)(属性名由dataset属性名决定))

#### 3.绘制

分别绘制矩形rect、圆形circle、文字text、坐标轴axis，绘制过程与柱形图相似。

## 矩阵树图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2015/04/400.png)

#### 1.数据格式

```javascript
var dataSet = {
	"name": "中国",
	"children":
	[
		{
			"name": "浙江",
			"children": 
			[
				{"name":"杭州", "gdp":8343},
				{"name":"宁波", "gdp":7128},
				{"name":"温州", "gdp":4003},
				{"name":"绍兴", "gdp":3620},
				{"name":"湖州", "gdp":1803},
				{"name":"嘉兴", "gdp":3147},
				{"name":"金华", "gdp":2958},
				{"name":"衢州", "gdp":1056},
				{"name":"舟山", "gdp":1021},
				{"name":"台州", "gdp":3153},
				{"name":"丽水", "gdp":983}
			]
		},
		......
	]
}
```

#### 2.布局

创建矩阵树图布局

```javascript
var treemap = d3.layout.treemap()
	.size([width, height])
	.value(function(d) {return d.gdp;})
```

数据转换

```javascript
var nodes = treemap.nodes(dataSet);
var links = treemap.links(nodes);
```

转换后的数据：

> 节点：parent、children、depth、value、x、y、dx、dy
>
> 连线：source、target

#### 3.绘制

分别绘制矩形rect、文字text。

## 地图

![](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/08/103.png)

#### 1.投影，转换经纬度数据，使能够在二维平面上显示

```javascript
var projection = d3.geo.mercator()	//投影函数
	.center([107, 31])	//地图中心位置，经纬度
	.scale(850)	//缩放比例
    .translate([width/2, height/2]);
```

#### 2.创建地图路径生成器

```javascript
var path = d3.geo.path()
	.projection(projection);
```

#### 3.绘制

```javascript
.data(root.features)	//数据绑定
...
.attr("d", path)	//调用路径生成器
```