<template>
  <div class="page">
    <!--    <img class="back-image" src="assets/example.png" alt="example-pic">-->
    <canvas
        :width=canvasSize.width
        :height=canvasSize.height
        id="drawer"
        @click="mouseClick"
        @mousemove="mouseMove"
        @mousedown="mouseDown"
        @mouseup="mouseUp"
    >Canvas fallback text
    </canvas>
  </div>
  <button @click="clearButtonClick">Очистить область</button>
  <button @click="buttonClick">{{constructActionName}}</button>
</template>

<script>
export default {
  name: 'MainComponent',
  props: {},
  data: function () {
    return {
      vueCanvas: null,
      canvasSize: {width: 300, height: 300},
      pointsArray: [],
      regionMode: false,
      dragMode: false,
      selectedElem: null,
      constructActionName: "Построить область"
    };
  },
  mounted() {
    let c = document.getElementById("drawer");
    if (!c.getContext) return;
    this.vueCanvas = c.getContext("2d");
    this.clearCanvas(this.vueCanvas);
  },
  methods: {
    drawPoint: function (ctx, x, y) {
      ctx.beginPath();
      ctx.arc(x, y, 7, 0, Math.PI * 2, true);
      ctx.fillStyle = "#000000";
      ctx.fillStyle = "#FFFFFF";
      ctx.fill();
      ctx.arc(x, y, 4, 0, Math.PI * 2, false);
      ctx.fillStyle = "#2F80ED";
      ctx.fill();
    },
    innerPathPoint: function (path, x, y) {
      path.arc(x, y, 4, 0, Math.PI * 2, true);
    },
    outerPathPoint: function (path, x, y) {
      path.arc(x, y, 7, 0, Math.PI * 2, true);
    },
    innerPathSelPoint: function (path, x, y) {
      path.arc(x, y, 14, 0, Math.PI * 2, true);
    },
    outerPathSelPoint: function (path, x, y) {
      path.arc(x, y, 11, 0, Math.PI * 2, true);
    },
    drawPointsArray: function (ctx, regionMode, selElem) {
      if (this.pointsArray.length === 0) return;
      this.clearCanvas(ctx);
      let innerPointsPath = new Path2D();
      let outerPointsPath = new Path2D();
      let innerPointsSelPath = new Path2D();
      let outerPointsSelPath = new Path2D();
      let regionPath = new Path2D();
      regionPath.moveTo(this.pointsArray[0].x, this.pointsArray[0].y);
      innerPointsPath.moveTo(this.pointsArray[0].x, this.pointsArray[0].y);
      outerPointsPath.moveTo(this.pointsArray[0].x, this.pointsArray[0].y);
      this.pointsArray.forEach((elem) => {
        regionPath.lineTo(elem.x, elem.y);
        if (elem === selElem) {
          innerPointsSelPath.moveTo(elem.x, elem.y);
          this.innerPathSelPoint(innerPointsSelPath, elem.x, elem.y);
          outerPointsSelPath.moveTo(elem.x, elem.y);
          this.outerPathSelPoint(outerPointsSelPath, elem.x, elem.y);
          ctx.fillStyle = "#2F80ED";
          ctx.fill(outerPointsSelPath);
          ctx.fillStyle = "#FFFFFF";
          ctx.fill(innerPointsSelPath);
          return;
        }
        innerPointsPath.moveTo(elem.x, elem.y);
        this.innerPathPoint(innerPointsPath, elem.x, elem.y);
        outerPointsPath.moveTo(elem.x, elem.y);
        this.outerPathPoint(outerPointsPath, elem.x, elem.y);
      });
      if (regionMode) {
        ctx.fillStyle = "rgba(47,128,237,0.22)";
        ctx.fill(regionPath);
        ctx.strokeStyle = "#2F80ED";
        ctx.lineWidth = 3;
        ctx.setLineDash([4, 2]);
        regionPath.closePath();
        ctx.stroke(regionPath);
      }
      ctx.fillStyle = "#2F80ED";
      ctx.fill(outerPointsPath);
      ctx.fillStyle = "#FFFFFF";
      ctx.fill(innerPointsPath);
      ctx.fillStyle = "#FFFFFF";
      ctx.fill(innerPointsSelPath);
      ctx.fillStyle = "#2F80ED";
      ctx.fill(outerPointsSelPath);
    },
    clearButtonClick: function () {
      this.regionMode = false;
      this.pointsArray = [];
      this.clearCanvas(this.vueCanvas);
    },
    buttonClick: function () {
      this.constructActionName = this.regionMode ? "Построить область" : "Отменить построение";
      this.regionMode = !this.regionMode;
      this.drawPointsArray(this.vueCanvas, this.regionMode);
    },
    clearCanvas: function (ctx) {
      ctx.clearRect(0, 0, this.canvasSize.width, this.canvasSize.height);
      ctx.rect(0, 0, this.canvasSize.width, this.canvasSize.height);
      ctx.fillStyle = "#DCDCDC";
      ctx.fill();
    },
    mouseUp: function (event) {
      console.log(event);
      if (!this.selectedElem) {
        let rect = event.target.getBoundingClientRect();
        let x = event.clientX - rect.left;
        let y = event.clientY - rect.top;
        // console.log("Coordinate x: " + x, "Coordinate y: " + y);
        this.pointsArray.push({x, y});
        this.drawPoint(this.vueCanvas, x, y);
      }
      this.dragMode = false;
    },
    mouseDown: function (event) {
      console.log(event);
      this.dragMode = true;
    },
    mouseMove: function (event) {
      let rect = event.target.getBoundingClientRect();
      let x = event.clientX - rect.left;
      let y = event.clientY - rect.top;
      // console.log("Coordinate x: " + x, "Coordinate y: " + y);
      let radius = 7;
      if(this.selectedElem && this.dragMode){
        this.selectedElem.x = x;
        this.selectedElem.y = y;
      }
      for (let elem of this.pointsArray) {
        if (x <= elem.x + radius && x >= elem.x - radius && y <= elem.y + radius && y >= elem.y - radius) {
          this.selectedElem = elem;
          return this.drawPointsArray(this.vueCanvas, this.regionMode, elem);
        }
      }
      this.selectedElem = null;
      this.dragMode = false;
      return this.drawPointsArray(this.vueCanvas, this.regionMode);
    },
  }
}
</script>
<style scoped>
#drawer {
  border: 1px solid black;
}

.back-image {
  max-height: 500px;

}

h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
