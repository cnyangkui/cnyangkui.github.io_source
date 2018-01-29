---
title: d3-heatmap
date: 2018-01-26 12:05:30
tags: D3
categories: [ Engineering, Visualization ]
---

{% asset_img  heatmap.png %}
{% asset_img  heatmap2.png %}

Heatmap热图，通过渐变的色带很优雅地表现了空间数据之间的差异。通过色块颜色深浅程度，可以直观的看到具有相似性的空间数据，对空间数据进行简单的分组。

<!--more-->

## Sketch

{% raw %}
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8">
</head>
<body>
<div id="heatmap" style="width:1500;height:410;overflow: auto"></div>
<script type='text/javascript' src='http://d3js.org/d3.v4.min.js'></script>
<script type="text/javascript">
let width = 1500
let height = 410
let svg = d3.select('#heatmap')
  .append('svg')
  .attr('width', width)
  .attr('height', height)
let fontSize = 8.5
let rectSize = 8.5
let padding = 1
let namewidth = 55
let margin = {top: 70, bottom: 20, left: 60, right: 20}
let color = d3.scaleLinear()
  .domain([0, 500])
  .range(['white', 'blue'])
let heatmap = svg.append('g')
  .attr('class', 'heatmap')
  .attr('transform', 'translate(' + margin.left + ', ' + margin.top + ')')
d3.csv('heatmap.csv', (error, data) => {
  if (error) {
    throw error
  }
  let rowProperty = []
  Object.keys(data[0]).forEach(property => {
    rowProperty.push(property)
  })
  // 创建多个行
  let gRow = heatmap.selectAll('g.row')
    .data(data)
    .enter()
    .append('g')
    .attr('class', 'row')
    .attr('transform', (d, i) => {
      return 'translate(0, ' + ((rectSize + padding) * i) + ')'
    })
  // 在每行矩形前添加文字标注
  let rowText = gRow.append('text')
    .attr('font-size', fontSize)
    .attr('text-anchor', 'end')
    .attr('dy', '1em')
    .text((d, i) => 'rowname' + (i+1))
  // 添加小矩形
  let heatRect = gRow.selectAll('rect.heat')
    .data(d => {
      let tmp = []
      Object.keys(d).forEach(k => {
        if (k !== 'name') {
          tmp.push({'server': d.name, 'key': k, 'value': d[k]})
        }
      })
      return tmp
    })
    .enter()
    .append('rect')
    .attr('class', 'heat')
    .attr('x', (d, i) => {
      return (rectSize + padding) * i
    })
    .attr('width', rectSize)
    .attr('height', rectSize)
    .attr('transform', (d, i) => {
      return 'translate(2, 2)'
    })
    .style('fill', (d, i) => {
      return color(+d.value)
    })
  // 创建显示高亮的行条
  let highlightRow = gRow.append('rect')
    .attr('class', 'highlightRow')
    .attr('x', 0)
    .attr('width', (rectSize + padding) * (rowProperty.length - 1) + namewidth)
    .attr('height', rectSize)
    .attr('transform', 'translate(' + (-namewidth) + ', 0)')
    .attr('fill', 'white')
    .attr('fill-opacity', 0.2)
    .datum(d => d.name)
  // 鼠标放到行上显示高亮
  gRow.on('mouseover', (d, i) => {
    highlightRow.style('fill', (row) => {
      if (row === d.name) {
        return 'red'
      }
    })
  })
  .on('mouseout', (d, i) => {
    highlightRow.style('fill', (row) => {
      if (row === d) {
        return 'white'
      }
    })
  })
  // 创建多个列
  let gCol = heatmap.append('g')
    .attr('class', 'cols')
    .selectAll('g.col')
    .data(data.columns.filter(d => d !== 'name'))
    .enter()
    .append('g')
    .attr('class', 'col')
    .attr('transform', (d, i) => {
      return 'translate('+ ((rectSize + padding) * i) +', ' + (-namewidth) + ')'
     })
  // 创建每一列的标题
  let colText = gCol.append('text')
    .attr('font-size', fontSize)
    .attr('transform', (d, i) => {
      return 'translate(' + rectSize +', ' + namewidth + ')rotate(-90)'
    })
    .text((d, i) => 'colname' + (i+1))
  // 创建显示高亮的竖条
  let highlightCol = gCol
    .append('rect')
    .attr('class', 'highlightCol')
    .attr('fill', 'white')
    .attr('fill-opacity', 0.2)
    .attr('width', rectSize)
    .attr('height', (rectSize + padding) * data.length + namewidth)
    .attr('transform', (d, i) => {
      return 'translate(2, 0)'
    })
    // 鼠标放到列上显示高亮
    .on('mouseover', (d, i) => {
        highlightCol.style('fill', (col) => {
          if (col === d) {
            return 'red'
          }
        })
      })
    .on('mouseout', (d, i) => {
      highlightCol.style('fill', (col) => {
        if (col === d) {
          return 'white'
        }
      })
    })
})
</script>
</body>
</html>
{% endraw %}

## Codes

{% codeblock lang:html %}
<div id="heatmap" style="width:1500;height:410;overflow: auto"></div>
{% endcodeblock %}

数据使用csv文件存储，数据格式如下：

{% codeblock lang:csv %}
name,colname1,colname2,colname3,colname4,colname5,colname6,colname7,...
rowname1,0,0,0,0,0,0,0,...
rowname2,0,0,0,0,0,0,0,...
rowname3,214,156,137,96,180,122,113,...
rowname4,449,381,381,392,341,350,328,...
rowname5,162,125,165,130,156,136,143,...
...,...,...,...,...,...,...,...,...
{% endcodeblock %}

`d3.csv`读取数据，函数的第一个参数是文件url地址。

创建svg后，首先绘制每一行。每一行都是一个g元素，在各行的g元素中绘制该行的rowname文本和所有的小矩形rect，然后在该行绘制一个横向的高亮矩形条。接着绘制每一列。每一列仍然是一个g元素，在各列的g元素中绘制一个colname文本和一个高亮的纵向矩形条。高亮矩形条默认颜色设置为白色，设置透明度样式。在高亮矩形条rect上添加`mouseover`和`mouseout`监听来改变颜色和恢复颜色。

{% codeblock lang:javascript %}
let width = 1500
let height = 440
let svg = d3.select('div')
  .append('svg')
  .attr('width', width)
  .attr('height', height)
let fontSize = 8.5	// 文字大小
let rectSize = 8.5	// 矩形大小
let padding = 1		// 矩形间隔
let namewidth = 55	// 行列属性文字长度
let margin = {top: 70, bottom: 20, left: 60, right: 20}

let color = d3.scaleLinear()
  .domain([0, 1000])
  .range(['white', 'blue'])

let heatmap = svg.append('g')
  .attr('class', 'heatmap')
  .attr('transform', 'translate(' + margin.left + ', ' + margin.top + ')')

d3.csv('heatmap.csv', (error, data) => {
  if (error) {
    throw error
  }
  let rowProperty = []	// 每一列的属性名
  Object.keys(data[0]).forEach(property => {
    rowProperty.push(property)
  })

  // 创建多个行
  let gRow = heatmap.selectAll('g.row')
    .data(data)
    .enter()
    .append('g')
    .attr('class', 'row')
    .attr('transform', (d, i) => {
      return 'translate(0, ' + ((rectSize + padding) * i) + ')'
    })

  // 在每行矩形前添加文字标注
  let rowText = gRow.append('text')
    .attr('font-size', fontSize)
    .attr('text-anchor', 'end')
    .attr('dy', '1em')
    .text(d => d.name)
  
  // 添加小矩形
  let heatRect = gRow.selectAll('rect.heat')
    .data(d => {
      let tmp = []
      Object.keys(d).forEach(k => {
        if (k !== 'name') {
          tmp.push({'server': d.name, 'key': k, 'value': d[k]})
        }
      })
      return tmp
    })
    .enter()
    .append('rect')
    .attr('class', 'heat')
    .attr('x', (d, i) => {
      return (rectSize + padding) * i
    })
    .attr('width', rectSize)
    .attr('height', rectSize)
    .attr('transform', (d, i) => {
      return 'translate(2, 2)'
    })
    .style('fill', (d, i) => {
      return color(+d.value)
    })

  // 创建显示高亮的行条
  let highlightRow = gRow.append('rect')
    .attr('class', 'highlightRow')
    .attr('x', 0)
    .attr('width', (rectSize + padding) * (rowProperty.length - 1) + namewidth)
    .attr('height', rectSize)
    .attr('transform', 'translate(' + (-namewidth) + ', 0)')
    .attr('fill', 'white')
    .attr('fill-opacity', 0.2)

  // 鼠标放到行上显示高亮
  gRow.on('mouseover', (d, i) => {
    highlightRow.style('fill', (row) => {
      if (row === d) {
        return 'red'
      }
    })
  })
  .on('mouseout', (d, i) => {
    highlightRow.style('fill', (row) => {
      if (row === d) {
        return 'white'
      }
    })
  })

  // 创建多个列
  let gCol = heatmap.append('g')
    .attr('class', 'cols')
    .selectAll('g.col')
    .data(data.columns.filter(d => d !== 'name'))
    .enter()
    .append('g')
    .attr('class', 'col')
    .attr('transform', (d, i) => {
      return 'translate('+ ((rectSize + padding) * i) +', ' + (-namewidth) + ')'
     })

  // 创建每一列的标题
  let colText = gCol.append('text')
    .attr('font-size', fontSize)
    .attr('transform', (d, i) => {
      return 'translate(' + rectSize +', ' + namewidth + ')rotate(-90)'
    })
    .text(d => d)

  // 创建显示高亮的竖条
  let highlightCol = gCol
    .append('rect')
    .attr('class', 'highlightCol')
    .attr('fill', 'white')
    .attr('fill-opacity', 0.2)
    .attr('width', rectSize)
    .attr('height', (rectSize + padding) * data.length + namewidth)
    .attr('transform', (d, i) => {
      return 'translate(2, 0)'
    })
    // 鼠标放到列上显示高亮
    .on('mouseover', (d, i) => {
        highlightCol.style('fill', (col) => {
          if (col === d) {
            return 'red'
          }
        })
      })
    .on('mouseout', (d, i) => {
      highlightCol.style('fill', (col) => {
        if (col === d) {
          return 'white'
        }
      })
    })
})
{% endcodeblock %}