<template>
<div class="content">
<canvas 
ref="canvas" 
width="800" 
height="600" 
@mousedown="handleMouseDown"
@mouseup = "handleMouseUp"
></canvas>
</div>
</template>

<script lang="ts">
interface point{
    x:number;
    y:number;
}
import { defineComponent } from 'vue';
export default defineComponent({
  name: 'App',
  data(){
    return {
      points:[] as point[],
      drag_point:null,
      drag_shift:null,
      drag_also:null,
      drag_also_shift:null,
      state:0
    }
  },
  methods:{
    handleMouseDown(e:MouseEvent){
      function getDistance(p1:point, p2:point):number{
          const x = p1.x - p2.x
          const y = p1.y - p2.y
          return Math.sqrt(x * x + y * y)
      }
      
      
      const {offsetX, offsetY} = e
      const mouse:point = {x:offsetX,y:offsetY}
      var prev = null
      for(const point of this.points)
      {
        if(prev != null)
        {
          const REVP = this.reverse(point,prev);
          if(getDistance(point,mouse)<10)
          {
            this.drag_point = point
            this.drag_shift = {x:point.x - mouse.x, y:point.y - mouse.y}
            this.$refs.canvas.addEventListener('mousemove',this.dragStep)
            this.state = 0
            
            return
          }
          if(getDistance(prev,mouse)<10)
          {
            this.drag_point = prev
            this.drag_shift = {x:prev.x - mouse.x, y:prev.y - mouse.y}
            this.$refs.canvas.addEventListener('mousemove',this.dragStep)
            this.drag_also = point
            this.drag_also_shift = {x:point.x - mouse.x, y:point.y - mouse.y}
            this.state = 2
            return
          }
          if(getDistance(REVP,mouse)<10)
          {
            this.state = 3
            this.drag_point = point
            const REV = this.reverse(point,prev);
            this.drag_shift = {x:REV.x - mouse.x, y:REV.y - mouse.y}
            this.drag_also = prev
            this.$refs.canvas.addEventListener('mousemove',this.dragStep)
            return
          }
          prev = null
        }
        else
        {
          prev = point
        }
      }
      if(this.state == 0)
      {
        this.points.push({x:offsetX,y:offsetY})
        const HANDLE = {x:offsetX,y:offsetY}
        this.points.push(HANDLE)
        this.drag_point = HANDLE
        this.drag_shift = {x:offsetX,y:offsetY}
        this.state = 1
        this.$refs.canvas.addEventListener('mousemove',this.dragStep)
      }
      
      this.update()
    },
    reverse(point:point,center:point):point
    {
      return {
        x:((point.x - center.x) * -1) + center.x,
        y:((point.y - center.y) * -1) + center.y,
      }
    },
    dragStep(e:MouseEvent)
    {
      const {offsetX, offsetY} = e
      switch(this.state)
      {
        case 0:
        {
          this.drag_point.x = offsetX + this.drag_shift.x
          this.drag_point.y = offsetY + this.drag_shift.y
        }
        break
        case 1:
        {
          this.drag_point.x = ((offsetX - this.drag_shift.x) * -1) + this.drag_shift.x
          this.drag_point.y = ((offsetY - this.drag_shift.y) * -1) + this.drag_shift.y
        }
        break
        case 2:
        {
          this.drag_point.x = offsetX + this.drag_shift.x
          this.drag_point.y = offsetY + this.drag_shift.y
          this.drag_also.x = offsetX + this.drag_also_shift.x
          this.drag_also.y = offsetY + this.drag_also_shift.y
        }
        break
        case 3:
        {
          this.drag_point.x = ((offsetX + this.drag_shift.x - this.drag_also.x) * -1) + this.drag_also.x 
          this.drag_point.y = ((offsetY + this.drag_shift.y - this.drag_also.y) * -1) + this.drag_also.y 
        }
        break;
      }
      this.update();
    },
    handleMouseUp(e:MouseEvent)
    {
      this.$refs.canvas.removeEventListener('mousemove',this.dragStep)
      this.state = 0
    },
    
    update(){
      const CANVAS = this.$refs.canvas as HTMLCanvasElement;
      const CTX = CANVAS.getContext("2d") as CanvasRenderingContext2D;
      

      function draw_circle(position:point,radius:number)
      {
        CTX.beginPath();
        CTX.arc(position.x, position.y, radius, 0, 2 * Math.PI);
        CTX.fill();
        CTX.stroke();
      }

      function draw_line(from:point,to:point)
      {
        CTX.beginPath();
        CTX.moveTo(from.x, from.y);
        CTX.lineTo(to.x, to.y);
        CTX.stroke();
      }

      function draw_bezier(p1:point,p2:point,p3:point,p4:point)
      {
        CTX.beginPath();
        CTX.moveTo(p1.x, p1.y);
        CTX.bezierCurveTo(p2.x, p2.y, p3.x, p3.y, p4.x, p4.y);
        CTX.stroke();
      }

      CTX.clearRect(0,0,CANVAS.width,CANVAS.height);
      CTX.strokeStyle = "#fff";
      CTX.lineWidth = 1
      for(var i = 2; i < this.points.length;i+=2)
      {
        draw_bezier(this.points[i-2],this.reverse(this.points[i-1],this.points[i-2]),this.points[i+1],this.points[i])
      }
      
      CTX.fillStyle = "rgb(0,50,128)";
      CTX.strokeStyle = "rgb(0,128,255)";
      
      var prev = null
      for(const point of this.points)
      {
        if(prev != null)
        {
          CTX.lineWidth = 2
          const REVP = this.reverse(point,prev);
          draw_line(point,prev)
          draw_line(point,REVP)
          CTX.lineWidth = 1
          draw_circle(point,5)
          draw_circle(prev,5)
          draw_circle(REVP,5)
          prev = null
        }
        else
        {
          prev = point
        }
      }
      
      
    }
  },
  mounted() {
    this.$refs.canvas.width = window.innerWidth;
    this.$refs.canvas.height = window.innerHeight;
    this.update();
  },
});
</script>

<style scoped>
canvas{
border:1px solid var(--bright);
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
</style>
