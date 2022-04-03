<template>
<canvas 
ref="canvas" 
width="640" 
height="480" 
style="border:1px solid var(--bright);"
@click="handleClick"
@mousemove="handleMouseMove"
></canvas>
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
      point:{x:20,y:20} as point
    }
  },
  methods:{
      handleClick(e:MouseEvent){
        this.point.x = e.offsetX;
        this.point.y = e.offsetY;
        this.update();
    },
    update(){
      const CANVAS = this.$refs.canvas as HTMLCanvasElement;
      const CTX = CANVAS.getContext("2d") as CanvasRenderingContext2D;
      CTX.clearRect(0,0,CANVAS.width,CANVAS.height);
      CTX.strokeStyle = "#fff";
      CTX.fillStyle = "#fff";
      function fill_circle(position:point,radius:number)
      {
        CTX.beginPath();
        CTX.arc(position.x, position.y, radius, 0, 2 * Math.PI);
        CTX.fill();
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

      fill_circle(this.point,5)
      fill_circle({x:20,y:100},5)
      draw_line(this.point,{x:20,y:100})

      fill_circle({x:200,y:20},5)
      fill_circle({x:200,y:100},5)
      draw_line({x:200,y:20},{x:200,y:100})
      
      draw_bezier(this.point,{x:20,y:100},{x:200,y:100},{x:200,y:20})
    }
  },
  mounted() {
    this.update();
  },
});
</script>

<style scoped>
</style>