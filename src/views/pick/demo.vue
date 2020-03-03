<template>
    <div style="width: 650px; margin: 0 auto">
        <input v-show="uploadButton" type="file" name="xlfile" id="xlf" @change="doFile"/>
        <!-- 横向过道数量 -->
        <!-- <input type="text" :value="hen" id="hen"> 竖向过道数量
        <input type="text" :value="shu" id="shu"> 竖向每排货架数量
        <input type="text" :value="num" id="num"> -->
        <!-- <input type="button" @click="layout()" id="layout" value="生成布局"> -->
        <div class="wrapper">
            <canvas id="canvas"></canvas>
            <div class="top">
                <blockItem classname="gray" :itemwidth="blockWidth" :margin="margin" v-for="(item, index) in ((shu-1) *3 +2)" :key="index" :class="{ 'transparent' : index===0}"></blockItem>
            </div>
            <template v-for="(item) in hen-1">
                <div class="container" :style="{'width': shu*3*(blockWidth+2*margin) + 'px', 'height': num * (blockWidth + margin * 2 ) + 'px'}">
                    <div class="huojia shu">
                        <blockItem classname="item" :itemwidth="blockWidth" :margin="margin" v-for="(item, index) in arr.slice(0, num)" :num="getNum(item)" :class="{ 'active' : items.includes(item)}" :key="index"></blockItem>
                    </div>
                    
                    <div style="display: flex; flex: 1; "  class="xutest">
                    <template class="main-item" v-for="(item, index1) in shu-1">
                        <div class="guodao shu" style="flex: 1">
                            <blockItem :class="{ 'linehight' : (index <= shubian - 1) && index1 == (hengbian - 1) / 3 }" classname="gray" :itemwidth="blockWidth" :margin="margin" v-for="(item, index) in num" :key="index"></blockItem>
                        </div>
                        <div class="huojia shu" style="flex: 2">
                            <blockItem classname="item" :itemwidth="blockWidth" :margin="margin" v-for="(item, index) in arr.slice(index1*num*2+num, index1*num * 2 + num*3)" :num="getNum(item)" :class="{ 'active' : items.includes(item)}" :key="index"></blockItem>
                        </div>
                    </template>
        </div>
        <div class="guodao shu" :style="{'width': blockWidth + 2 * margin + 'px'}">
            <blockItem classname="gray" :itemwidth="blockWidth" :margin="margin" v-for="(item, index) in num" :key="index"></blockItem>
        </div>
        <div class="huojia shu" :style="{'width': blockWidth + 2 * margin + 'px'}">
            <blockItem classname="item" :itemwidth="blockWidth" :margin="margin" v-for="(item, index) in arr.slice(count-num, count)" :num="getNum(item)" :class="{ 'active' : items.includes(item)}" :key="index"></blockItem>
        </div>
    </div>
    <div class="top">
        <blockItem :class="{ 'linehight' : index <= hengbian }" classname="gray" :itemwidth="blockWidth" :margin="margin" v-for="(item, index) in ((shu-1) *3 +2)" :key="index"></blockItem>
    </div>
    </template>
    </div>
    </div>
</template>
<script>
import blockItem from './blockItem'
import XLSX from 'xlsx'
export default {
    name: 'demo',
    components: {
        blockItem
    },
    data() {
        return {
            hen: 2,
            shu: 5,
            num: 10,
            items: [],
            arr: [],
            blockWidth: 32,
            margin: 5,
            hengbian: 0,
            shubian: 0,
            points:[[]],
            uploadButton: true
        }
    },
    computed: {
        wrapperWidth: function() {
            return (this.shu - 1) * 33 * this.blockWidth;
        },
        count: function() {
            return this.shu * this.num * 2;
        }
    },
    created() {
        var arr1 = new Array(this.count);
        for (var i = 0; i < arr1.length; i++) {
            arr1[i] = i + 1;
        }
        this.$data.arr = arr1;
        
        // var xlf = document.getElementById('xlf');
        // if(!xlf.addEventListener) return;

        // xlf.addEventListener('change', do_file, false); //添加监听



    },
    methods: {
        getNum(num) {
            let index = this.items.indexOf(num);
            index = index === -1 ? '' : index + 1
            return index
        },
        processWB(wb) {
            var ws = wb.Sheets[wb.SheetNames[0]];   //获取第几个sheet页
            let points = XLSX.utils.sheet_to_json(ws, {header:1});    //header 从第几行开始解析
            console.log('二维数组',points);      //得到一个二维数组。
            this.points = points;

            let row = points.length;
            let col = points[0].length;
            let items = points[row-1][col-1];
            this.items = items.split(',').map((e) => {
                return parseInt(e);
            });
            console.log('需要拣选的数字列表',this.items);
            this.layout()
            this.uploadButton = false
        },
        doFile(files) {
            var reader = new FileReader();
            let self = this;
            reader.onload = function(e) {
                self.processWB(XLSX.read(e.target.result, {type: 'binary'}));
            };
            reader.readAsBinaryString(files.target.files[0]);
        },
        layout() {
            var hen = this.hen;
            var shu = this.shu;
            var num = this.num;
            var arr1 = new Array(shu * num * 2);
            for (let i = 0; i < arr1.length; i++) {
                arr1[i] = i + 1;
            }
            this.$data.arr = arr1;
            console.log(this.num);

            canvas.width = this.shu * 3 * (this.blockWidth + 2 * this.margin);
            canvas.height = this.num * (this.blockWidth + this.margin * 2) + 60;
            let count = this.$data.num;
            let pointData = [];
            let len = this.points.length - 1;
            for(let i = 0; i < len - 1; i++) {
                let pp = JSON.parse(this.points[i][0]);
                pointData.push(pp); 
            }
            let lastpoint1 = JSON.parse(this.points[len-1][0]);
            let lastpoint2 = JSON.parse(this.points[len-1][1]);
            pointData.push(lastpoint1); 
            pointData.push(lastpoint2); 
            //console.log(pointData);
            this.drawPath(pointData, canvas.height);
        },
        //canvas绘图
        drawPath(pointData, height) {

            let pointArr = [];
            console.log('pointData', pointArr);
            pointData.forEach((item) => {
                let x = (item[0] - 1) * 42 + 20;
                let y = height - 42 * item[1];
                pointArr.push([x, y]);
            });
            console.log('pointArr', pointArr);
            var canvas = document.getElementById("canvas");
            var context = canvas.getContext('2d');
            context.lineWidth = 2;
            context.strokeStyle = "red";
            if (pointArr.length >= 2) {
                for (let i = 0; i <= pointArr.length - 2; i++) {
                    drawLine(pointArr[i], pointArr[i + 1]);
                }
            }

            function drawLine(point1, point2) {
                context.beginPath();
                context.moveTo(point1[0], point1[1]);
                context.lineTo(point2[0], point2[1]);
                context.stroke();
                context.closePath();
            }
        }
    }
        
}
</script>
<style>
    * {
        margin: 0;
        padding: 0;
        text-decoration: none;
    }
    
    body {
        background: #fff;
        font-size: 12px;
    }
    
    .active {
        background: red;
        line-height: 30px;
        font-size: 20px;
    }
    
    .top-item {
        background: #ccc;
        flex: 1;
    }
    
    .container {
        width: 100%;
        display: flex;
        flex-direction: row;
    }
    
    .huojia {
        display: flex;
        flex-direction: column-reverse;
        flex-wrap: wrap;
        text-align: center;
        box-sizing: content-box
    }
    
    .guodao {
        display: flex;
        flex-direction: column-reverse;
        box-sizing: content-box
    }
    
    .item {
        border: 1px solid #b6d8fc;
        box-sizing: content-box;
        background-color: #b6d8fc
    }
    
    .gray {
        background: #f3f3f3;
        border: 1px solid #f3f3f3;
        box-sizing: content-box
    }
    
    .top {
        display: flex;
        flex-direction: row;
        box-sizing: content-box
    }
    
    .transparent {
        background: transparent;
        border-color: transparent;
        box-sizing: content-box
    }
    /* .top:nth-child(3)>.gray:nth-child(-n+5) {
        background-color: aqua;
    }
    
    .xutest>.guodao:nth-of-type(3)>.gray:nth-child(-n+8) {
        background-color: aqua;
    } */
    
    .linehight {
        background-color: #c71319;
    }
    
    .wrapper {
        position: relative;
    }
    
    #canvas {
        position: absolute;
        top: 0;
        left: 0;
        z-index: 99;
    }
</style>