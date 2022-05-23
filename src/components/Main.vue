<template>
  <div class="page">
    <div class="background">
      <div class="canvas-header">
        <div class="header-text">
          Зал Рафаэля
        </div>
      </div>
      <canvas
          :width=canvasSize.width
          :height=canvasSize.height
          id="drawer"
          @click="mouseClick"
          @mousemove="mouseMove"
          @mousedown="mouseDown"
          @mouseup="mouseUp"
          @mouseleave="mouseLeave"
      >Canvas fallback text
      </canvas>

    </div>
    <div class="control-panel">
      <button class="simple-button" @click="clearButtonClick">Очистить область</button>
      <button class="simple-button" @click="constructButtonClick">{{ constructActionName }}</button>
      <div class="description">
        <p>ЛКМ - добавление точки</p>
        <p>Зажатие ЛКМ - перемещение существующей точки</p>
      </div>
      <ul class="pointList">
        <li v-for="(point, index) in pointsArray" :key="`point-${index}`">{{ point.x }}, {{ point.y }}</li>
      </ul>
    </div>
  </div>


</template>

<script>
export default {
  name: 'MainComponent',
  props: {},
  data: function () {
    return {
      canvasElem: null,
      vueCanvas: null,
      canvasSize: {width: 1920 / 2, height: 1080 / 2},
      pointsArray: [],
      regionMode: false,
      dragMode: false,
      selectedElem: null,
      constructActionName: "Построить область",
      img: null
    };
  },
  mounted() {
    let c = document.getElementById("drawer");
    if (!c.getContext) return;
    this.canvasElem = c;
    this.vueCanvas = c.getContext("2d");


    // var ctx = document.getElementById('canvas').getContext('2d');
    this.img = new Image();
    this.img.src = 'assets/example.png';
    // console.log(this.vueCanvas);
    console.log(this.vueCanvas);
    this.img.onload = () => {
      console.log(this.vueCanvas);
      this.vueCanvas.drawImage(this.img, 0, 0);
    }

    // img.onload = function () {
    //
    // };
    // this.clearCanvas(this.vueCanvas);


  },
  methods: {
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
      this.constructActionName = "Построить область";
      console.log(this.constructActionName);
      this.pointsArray = [];
      this.clearCanvas(this.vueCanvas);
    },
    constructButtonClick: function () {
      this.constructActionName = this.regionMode ? "Построить область" : "Отменить построение";
      this.regionMode = !this.regionMode;
      this.drawPointsArray(this.vueCanvas, this.regionMode);
    },
    clearCanvas: function (ctx) {
      ctx.clearRect(0, 0, this.canvasSize.width, this.canvasSize.height);
      ctx.rect(0, 0, this.canvasSize.width, this.canvasSize.height);
      this.vueCanvas.drawImage(this.img, 0, 0);
      // ctx.fillStyle = "#DCDCDC";
      // ctx.fill();
    },
    mouseUp: function (event) {
      console.log(event);
      if (!this.selectedElem) {
        let rect = event.target.getBoundingClientRect();
        let x = event.clientX - rect.left;
        let y = event.clientY - rect.top;
        // console.log("Coordinate x: " + x, "Coordinate y: " + y);
        let newElem = {x, y};
        this.pointsArray.push(newElem);
        this.canvasElem.style.cursor = "pointer";
        this.drawPointsArray(this.vueCanvas, this.regionMode, this.pointsArray[this.pointsArray.length - 1]);
        // this.drawPoint(this.vueCanvas, x, y);
      }
      this.dragMode = false;
    },
    mouseDown: function (event) {
      console.log(event);
      this.dragMode = true;
    },
    mouseLeave: function (event) {
      console.log(event);
      this.dragMode = false;
    },
    mouseMove: function (event) {
      let rect = event.target.getBoundingClientRect();
      let x = event.clientX - rect.left;
      let y = event.clientY - rect.top;
      // console.log("Coordinate x: " + x, "Coordinate y: " + y);
      let radius = 7;
      if (this.selectedElem && this.dragMode) {
        this.selectedElem.x = x;
        this.selectedElem.y = y;
      }
      for (let elem of this.pointsArray) {
        if (x <= elem.x + radius && x >= elem.x - radius && y <= elem.y + radius && y >= elem.y - radius) {
          this.selectedElem = elem;
          this.canvasElem.style.cursor = "pointer";
          return this.drawPointsArray(this.vueCanvas, this.regionMode, elem);
        }
      }
      this.canvasElem.style.cursor = "default";
      this.selectedElem = null;
      this.dragMode = false;
      return this.drawPointsArray(this.vueCanvas, this.regionMode);
    },
  }
}
</script>
<style scoped>

.background {
  position: relative;
}
.canvas-header {
  position: absolute;
  width: 100%;
  text-align: left;
  background: rgba(31, 54, 61, 0.8);
}

.header-text {
  font-family: 'Segoe UI';
  font-style: normal;
  font-weight: 600;
  font-size: 22px;
  line-height: 29px;
  color: #FFFFFF;
  text-shadow: 0px 2px 3px rgba(0, 0, 0, 0.5);
  padding: 5px 0px 5px 25px;

}

.page {
  display: flex;
  flex-flow: row nowrap;
  column-gap: 30px;
  /*height: 1000px;*/
}

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

.control-panel {
  display: flex;
  flex-flow: column nowrap;
  align-items: center;
  row-gap: 20px;
  text-align: start;
  width: 500px;
  border: 2px solid #1e1e1e;
  padding: 10px;

}

.pointList {
  width: 150px;
  height: 300px;
  overflow: scroll;
}

.simple-button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 200px;
}
</style>
