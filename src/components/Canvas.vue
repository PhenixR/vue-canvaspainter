<template>
  <div class="root">
      <div class="container">
        <canvas 
            class="canvas"
            width="800px"
            height="600px"
            @mousedown="canvasDown($event)"
            @mousemove="canvasMove($event)"
            @mouseup="canvasUp($event)"
            @mouseleave="canvasOut($event)"></canvas>
        <div class="tools">
            <div class="pen_color">
                <ul>
                    <li
                      v-for="item in colors"
                      @click="setColor(item)"
                      :style="{ background:item}"></li>
                </ul>
            </div>
            <div class="pen_width">
                <ul>
                    <li
                      v-for="pen in brushs"
                      @click="setWidth(pen.lineWidth)"></li>
                </ul>
            </div>
            <div class="control"></div>
        </div>
      </div>
  </div>
</template>

<style>
.container {
    display: flex;
    flex-direction: row;
    border:2px solid black;
}
.canvas {
    border:2px solid black;
}
.tools {
    border:2px solid black;
}
.pen_color ul {
    display: flex;
    padding-left: 0px;
}
.pen_color ul li{
    display: flex;
    width: 13px;
    height: 13px;
    border: 3px #fff solid;
    margin: 8px;
    cursor: pointer;
}
.pen_width ul {
    display: flex;
}
.pen_width ul li{
    display: flex;
    width: 13px;
    height: 13px;
    border: 3px rgb(92, 60, 60) solid;
    margin: 8px;
    cursor: pointer;
  }
</style>


<script>
export default {
  name:'Canvas',
  data:function() {
      return{
          colors: ['#fef4ac', '#0018ba', '#ffc200', '#f32f15', '#cccccc', '#5ab639'],
          brushs: [{
          className: 'small_pen',
          lineWidth: 3
        }, {
          className: 'middle_pen',
          lineWidth: 6
        }, {
          className: 'big_pen',
          lineWidth: 12
        }],
          config: {
          lineWidth: 1,
          lineColor: '#f2849e',
          shadowBlur: 2
        },
        contaxt: {},
        canvasMoveUse: false,
      }   
  },
   created () {
      this.$emit('setNav', '在线涂鸦画板')
    },
  mounted() {
      const canvas = document.querySelector('.canvas')
      this.context = canvas.getContext('2d')
      this.setCanvasStyle()
  },
  methods: {
        canvasMove (e) {
        if (this.canvasMoveUse) {
            var canvasX = e.clientX - e.target.parentNode.offsetLeft + document.body.scrollLeft
            var canvasY = e.clientY - e.target.parentNode.offsetTop + document.body.scrollTop
            this.context.lineTo(canvasX, canvasY)
            this.context.stroke()
        }
      },
      // mouseup
      canvasUp (e) {
        this.canvasMoveUse = false
      },
      canvasOut(e) {
          this.canvasMoveUse = false
      },
      // mousedown
      canvasDown (e) {
        console.log('canvasDown')
        this.canvasMoveUse = true
        // client是基于整个页面的坐标
        // offset是cavas距离顶部以及左边的距离
        const canvasX = e.clientX - e.target.parentNode.offsetLeft + document.body.scrollLeft
        const canvasY = e.clientY - e.target.parentNode.offsetTop + document.body.scrollTop
        this.context.beginPath()
        this.setCanvasStyle()
        // 清除子路径
        this.context.beginPath()
        this.context.moveTo(canvasX, canvasY)
        console.log('moveTo', canvasX, canvasY)
      },
      setColor(color) {
          this.config.lineColor = color
      },
      setWidth (type) {
        this.config.lineWidth = type
      },
      setCanvasStyle () {
      this.context.lineWidth = this.config.lineWidth
      this.context.shadowBlur = this.config.shadowBlur
      this.context.shadowColor = this.config.lineColor
      this.context.strokeStyle = this.config.lineColor
    },
    }
}
</script>

