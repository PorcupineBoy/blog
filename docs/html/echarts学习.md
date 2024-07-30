#### echarts柱形图上方显示数值

```
option = {
    xAxis: {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
        type: 'value'
    },
    series: [{
        data: [120, 200, 150, 80, 70, 110, 130],
        type: 'bar',
        itemStyle: {        //上方显示数值
                normal: {
                    label: {
                        show: true, //开启显示
                        position: 'top', //在上方显示
                        textStyle: { //数值样式
                            color: 'black',
                            fontSize: 16
                        }
                    }
                }
            
        }
    }]
};

```

```
option-xAxis.data : 横坐标数据[]
option-yAxis: 可为[], 标识多个y轴
	 [{},{},{}]   区分左右y轴。
	       .axisLabel:{
	       		formatter:function(value){return value+"单位"}
	       }
option-serise:[] :图线 line,bar
			{
			type:'line',
			color:"#48D1FF"，
			yAxisIndex:0 //映射哪个Y轴 ，多个Y轴可用。
			data:[] //图像映射的值
			}
			{
				type:'bar'
				yAxisIndex:1 //映射右边Y轴
			}
			 ,{
        color: "#48D1FF",
        name: '平均在线时长',
        type: 'bar',
        data: [],
        smooth: true,

        yAxisIndex: 1,
      },
      {
        barGap: "-100%", /!*这里设置包含关系，只需要这一句话*!/
        color: "#305072",
        name: '云存上传时长',
        type: 'bar',
        data: [],
        smooth: true,
        yAxisIndex: 1
      }
```

