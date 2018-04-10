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
                <h4>画笔颜色</h4>
                <ul>
                    <li
                      v-for="item in colors"
                      @click="setColor(item)"
                      :style="{ background:item}"></li>
                </ul>
            </div>
            <div class="pen_width">
                <h4>画笔大小</h4>
                <ul>
                    <li
                      v-for="pen in brushs"
                      :class="[pen.className,{'active': config.lineWidth === pen.lineWidth}]"
                      @click="setWidth(pen.lineWidth)"></li>
                </ul>
            </div>
            <div class="canvas_control">
                <h4>操作</h4>
                <div class="canvas_control_btn">
                    <span 
                        v-for="control in controls" 
                        :title="control.title" 
                        :class="control.className" 
                        @click="controlCanvas(control.action)"
                    ></span>
                </div>
            </div>
        </div>
      </div>
  </div>
</template>

<style>
.container {
    display: flex;
    flex-direction: row;

}
.canvas {
    border:2px solid black;
}
.tools {
    border:2px solid black;
}
ul {
    padding-left: 0px;
}
h4 {
    padding-left: 4em;
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
    padding: 0 50px;
    display: flex;
    justify-content: space-around;
}
.pen_width ul li{
    display: flex;
    width: 13px;
    height: 13px;
    margin: 8px;
    cursor: pointer;
  }
.small {
    font-size: 12px;
}
.middle {
    font-size: 14px;
}
.big {
    font-size: 16px;
}
.canvas_control_btn{
    padding: 0 50px;
    display: flex;
    justify-content: space-around;
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
          className: 'small fas fa-pencil-alt',
          lineWidth: 3
        }, {
          className: 'middle fas fa-pencil-alt',
          lineWidth: 6
        }, {
          className: 'big fas fa-pencil-alt',
          lineWidth: 12
        }],
          config: {
          lineWidth: 1,
          lineColor: '#f2849e',
          shadowBlur: 2
        },
        context: {},
        canvasMoveUse: false,
        preDrawAry: [],
        nextDrawAry: [],
        middleAry:[]
      }   
  },
   created () {
      this.$emit('setNav', '在线涂鸦画板')
    },
  mounted() {
      const canvas = document.querySelector('.canvas')
      this.context = canvas.getContext('2d')
      this.setCanvasStyle()
      this.initDraw()
  },
  computed: {
      controls() {
        return [{
          title: '上一步',
          action: 'prev',
          className: this.preDrawAry.length ? 'active fa fa-reply' : 'fa fa-reply'
        }, {
          title: '下一步',
          action: 'next',
          className: this.nextDrawAry.length ? 'active fa fa-share' : 'fa fa-share'
        }, {
          title: '清除',
          action: 'clear',
          className: (this.preDrawAry.length || this.nextDrawAry.length) ? 'active fa fa-trash' : 'fa fa-trash'
        }]
      }
  },
  methods: {
      initDraw() {
          var preData = this.context.getImageData(0, 0, 800, 600)
          this.middleAry.push(preData)
      },
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
          var preData = this.context.getImageData(0, 0, 800, 600)
          if(!this.nextDrawArylength) {
              this.middleAry.push(preData)
          }else{
              this.middleAry = []
              this,middleAry = this.middleAry.concat(this.preDrawAry)
              this.middleAry.push(preData)
              this.nextDrawAry = []
          }
          this.canvasMoveUse = false
      },
      canvasOut(e) {
          this.canvasMoveUse = false
      },
      // mousedown
      canvasDown (e) {
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
        var preData = this.context.getImageData(0, 0, 800, 600)
        this.preDrawAry.push(preData)
      },
      setColor(color) {
          this.config.lineColor = color
      },
      setWidth (type) {
        this.config.lineWidth = type
      },
            controlCanvas (action) {
        switch (action) {
          case 'prev':
            if (this.preDrawAry.length) {
              const popData = this.preDrawAry.pop()
              const midData = this.middleAry[this.preDrawAry.length + 1]
              this.nextDrawAry.push(midData)
              this.context.putImageData(popData, 0, 0)
            }
            break
          case 'next':
            if (this.nextDrawAry.length) {
              const popData = this.nextDrawAry.pop()
              const midData = this.middleAry[this.middleAry.length - this.nextDrawAry.length - 2]
              this.preDrawAry.push(midData)
              this.context.putImageData(popData, 0, 0)
            }
            break
          case 'clear':
            this.context.clearRect(0, 0, this.context.canvas.width, this.context.canvas.height)
            this.preDrawAry = []
            this.nextDrawAry = []
            this.middleAry = [this.middleAry[0]]
            break
        }
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

