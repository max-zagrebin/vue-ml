<template>
  <div class="page">

    <div class="background">
      <div class="canvas-header">
        <div class="header-text">
          Зал Рафаэля {{ debugX }} {{ debugY }}
        </div>
      </div>
      <canvas
          :width=canvasSize.width
          :height=canvasSize.height
          id="drawer"
          @mousemove="mouseMove"
          @mousedown="mouseDown"
          @mouseup="mouseUp"
          @mouseleave="mouseLeave"
          @contextmenu.prevent="mouseRightButton"
      >Canvas fallback text
      </canvas>
    </div>
    <div class="control-panel">
      <button class="simple-button" @click="constructButtonClick">{{ constructActionName }}</button>
      <button class="simple-button" @click="clearButtonClick">Очистить область</button>
      <div class="description">
        <p>ЛКМ - добавление точки</p>
        <p>ПКМ - удаление точки</p>
        <p>Зажатие ЛКМ - перемещение существующей точки</p>
        <p>{{ debugX }} {{ debugY }}</p>
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
      img: null,
      debugX: 0,
      debugY: 0
    };
  },
  mounted() {
    let c = document.getElementById("drawer");
    if (!c.getContext) return;
    this.canvasElem = c;
    this.vueCanvas = c.getContext("2d");
    this.img = new Image();
    this.img.src = 'assets/example.png';
    this.img.onload = () => {
      this.vueCanvas.drawImage(this.img, 0, 0);
    }
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
    drawPointsArray: function (ctx, regionMode, selElem, linePoint) {
      if (this.pointsArray.length === 0) return;
      this.clearCanvas(ctx);

      if (linePoint) {
        // console.log("LINE POINT PRESENT");
        let innerPointsLinePath = new Path2D();
        let outerPointsLinePath = new Path2D();
        innerPointsLinePath.moveTo(linePoint.x, linePoint.y);
        this.innerPathPoint(innerPointsLinePath, linePoint.x, linePoint.y);
        outerPointsLinePath.moveTo(linePoint.x, linePoint.y);
        this.outerPathPoint(outerPointsLinePath, linePoint.x, linePoint.y);
        ctx.fillStyle = "#2F80ED";
        ctx.fill(outerPointsLinePath);
        ctx.fillStyle = "#FFFFFF";
        ctx.fill(innerPointsLinePath);
        ctx.fillStyle = "#FFFFFF";
      }

      let innerPointsPath = new Path2D();
      let outerPointsPath = new Path2D();
      let innerPointsSelPath = new Path2D();
      let outerPointsSelPath = new Path2D();
      let regionPath = new Path2D();
      regionPath.moveTo(this.pointsArray[0].x, this.pointsArray[0].y);
      innerPointsPath.moveTo(this.pointsArray[0].x, this.pointsArray[0].y);
      outerPointsPath.moveTo(this.pointsArray[0].x, this.pointsArray[0].y);
      this.pointsArray.forEach((elem) => {
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


      for (let elem of this.pointsArray) {
        regionPath.lineTo(elem.x, elem.y);
      }
      // this.pointsArray.forEach((elem) => {
      //
      //
      // });
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
    },
    mouseUp: function () {


      this.dragMode = false;
    },
    mouseDown: function (event) {
      console.log(event);
      let rect = event.target.getBoundingClientRect();
      let x = Math.abs(event.clientX - rect.left);
      let y = Math.abs(event.clientY - rect.top);

      let mouseElem = {x, y};
      console.log("MOUSE DOWN");
      console.log(mouseElem);

      let targetIndex = this.pointsArray.length;
      // let minHypo = Number.MAX_VALUE;
      // let minElem = this.pointsArray[0];
      // for (let elem of this.pointsArray) {
      //   let diffX = mouseElem.x - elem.x;
      //   let diffY = mouseElem.y - elem.y;
      //   // let hypo = Math.sqrt(Math.pow(diffX,2) + Math.pow(diffY,2));
      //   let hypo = Math.hypot(diffX, diffY);
      //   console.log("CHECK ELEM");
      //   console.log(elem);
      //   console.log("CURRENT HYPO");
      //   console.log(hypo);
      //   if (hypo < minHypo) {
      //
      //     minHypo = hypo;
      //     minElem = elem;
      //     console.log("NEW MIN");
      //     console.log(hypo);
      //   }
      //   // console.log("DISTANCE");
      //   // console.log(elem);
      //
      //   // if (x <= elem.x + radius && x >= elem.x - radius && y <= elem.y + radius && y >= elem.y - radius) {
      //   //   this.selectedElem = elem;
      //   //   this.canvasElem.style.cursor = "pointer";
      //   //   return this.drawPointsArray(this.vueCanvas, this.regionMode, elem);
      //   // }
      // }
      // console.log("TOTAL CLOSEST //////////");
      // console.log(minElem);
      // console.log(minHypo);
      // let targetIndex = this.pointsArray.length;
      // if (this.pointsArray.length > 2) {
      //   targetIndex = this.pointsArray.indexOf(minElem);
      //   // if (this.pointsArray.length >= targetIndex + 1) {
      //   // if (this.pointsArray[targetIndex] === this.pointsArray.length - 1) {
      //   //   targetIndex = 0;
      //   // }
      //   console.log("PREV ELEM");
      //   console.log(this.pointsArray[targetIndex - 1] ?? this.pointsArray[this.pointsArray.length - 1]);
      //   console.log("NEXT ELEM");
      //   console.log(this.pointsArray[targetIndex + 1] ?? this.pointsArray[0]);
      //   let prevElem = this.pointsArray[targetIndex - 1];
      //   let nextElem = this.pointsArray[targetIndex + 1];
      //
      //   // let prevHypodiffX = mouseElem.x - elem.x;
      //   // let diffY = mouseElem.y - elem.y;
      //   // // let hypo = Math.sqrt(Math.pow(diffX,2) + Math.pow(diffY,2));
      //   // let hypo = Math.hypot(diffX, diffY);
      //
      //
      //   let prevHypo = Math.hypot(mouseElem.x - prevElem.x, mouseElem.y - prevElem.y);
      //   let nextHypo = Math.hypot(mouseElem.x - nextElem.x, mouseElem.y - nextElem.y);
      //   console.log("PREV HYPO");
      //   console.log(prevHypo);
      //   console.log("NEXT HYPO");
      //   console.log(nextHypo);
      //   if (prevHypo > nextHypo) {
      //     targetIndex += 1;
      //   }
      // }


      if (!this.selectedElem) {
        let rect = event.target.getBoundingClientRect();
        let x = event.clientX - rect.left;
        let y = event.clientY - rect.top;
        let newElem = {x, y};
        console.log("ADD AT TARGET INDEX " + targetIndex);
        this.pointsArray.splice(targetIndex, 0, newElem);
        // this.pointsArray.push(newElem);
        this.canvasElem.style.cursor = "pointer";
        this.drawPointsArray(this.vueCanvas, this.regionMode, this.pointsArray[this.pointsArray.length - 1]);
      }
      this.dragMode = true;
    },
    mouseLeave: function () {
      // console.log(event);
      this.dragMode = false;
    },


    mouseMove: function (event) {
      let rect = event.target.getBoundingClientRect();
      let x = event.clientX - rect.left;
      let y = event.clientY - rect.top;
      this.debugX = x;
      this.debugY = y;
      // console.log("Coordinate x: " + x, "Coordinate y: " + y);
      let radius = 7;

      let linePoint;

      // console.log("POINTS ARRAY");
      let curPoint;
      let prevPoint;
      linePoint = {x, y};
      // if (this.pointsArray.length > 0) {
      //   let point = this.pointsArray[0];
      //
      //   // console.log("MOUSE");
      //   // console.log(linePoint);
      //   // console.log("POINT");
      //   // console.log(point);
      //   console.log("MOUSE ANGLE");
      //   console.log(mouseAngle);
      // }
      let anglesMeet = false;
      for (let i = 1; i < this.pointsArray.length; i++) {
        // console.log(this.pointsArray[i]);
        curPoint = this.pointsArray[i];
        prevPoint = this.pointsArray[i - 1];
        // let angleDeg = (Math.atan2(curPoint.y - prevPoint.y, curPoint.x - prevPoint.x) * 180 / Math.PI).toFixed(0);
        // let mouseAngle = (Math.atan2(curPoint.y - linePoint.y, curPoint.x - linePoint.x) * 180 / Math.PI).toFixed(0);
        console.log("CUR POINT");
        console.log(curPoint.x);
        console.log(curPoint.y);
        console.log("MOUSE POINT");
        console.log(linePoint.x);
        console.log(linePoint.y);

        // let angleDeg = Math.tan((curPoint.y - prevPoint.y) / (curPoint.x - prevPoint.x));
        // let mouseAngle = Math.tan((curPoint.y - linePoint.y )/ (curPoint.x - linePoint.x));
        let angleDeg = Math.atan2(curPoint.y - prevPoint.y, curPoint.x - prevPoint.x);
        let mouseAngle = Math.atan2(curPoint.y - linePoint.y, curPoint.x - linePoint.x);
        console.log("DOTS ANGLE ////");
        console.log(angleDeg);
        console.log("MOUSE ANGLE1");
        console.log(mouseAngle);
        console.log("DIFF");
        console.log(Math.abs(angleDeg - mouseAngle));


        if (Math.abs(angleDeg - mouseAngle) < 0.02) {
          anglesMeet = true;
          console.log(Math.tan(angleDeg));  //(curPoint.y - linePoint.y) / (curPoint.x - linePoint.x)
          // console.log(Math.tan(angleDeg)*(curPoint.x - linePoint.x));
          console.log(Math.tan(angleDeg) / (curPoint.x - linePoint.x) + linePoint.x);
          console.log(Math.tan(angleDeg) / (curPoint.y - linePoint.y) + linePoint.y);
          // console.log(Math.tan(angleDeg)/(curPoint.x - prevPoint.x) + prevPoint.x);
          // console.log(Math.tan(angleDeg)/(curPoint.y - prevPoint.y) + prevPoint.y);
          // console.log(Math.tan(angleDeg)*(curPoint.x - prevPoint.x));
          // console.log(((curPoint.x - linePoint.x )/Math.tan(angleDeg) - prevPoint.x));
          // console.log(Math.tan(angleDeg)*(curPoint.y - linePoint.y));
          // console.log(Math.tan(mouseAngle));
          // console.log(Math.tan(angleDeg));
          // console.log(Math.tan(mouseAngle));
          // console.log(curPoint.yangleDeg / curPoint.y);
          // console.log(angleDeg / curPoint.x);
          // linePoint = {x, y};
          let saveX = linePoint.x;
          let saveY = linePoint.y;
          console.log("TG");
          console.log(Math.tan(angleDeg));
          console.log((curPoint.y - prevPoint.y) / (curPoint.x - prevPoint.x));
          console.log(curPoint.x - prevPoint.x);
          console.log(curPoint.y - prevPoint.y);
          console.log(curPoint.x - linePoint.x);
          console.log(curPoint.y - linePoint.y);
          // linePoint.x = Math.tan(angleDeg) / (curPoint.x - prevPoint.x) + Math.round(saveX);
          // linePoint.y = Math.tan(angleDeg) / (curPoint.y - prevPoint.y) + Math.round(saveY);
          linePoint.x = saveX - (curPoint.x - prevPoint.x);

          linePoint.y = (curPoint.y - prevPoint.y) * linePoint.x / (curPoint.x - prevPoint.x);
          // linePoint.y = saveY;
          console.log(linePoint);
          // linePoint = {x :, y : };
          break;
        }
      }
      if (!anglesMeet) {
        linePoint = undefined;
      }
      // for (let elem of this.pointsArray) {
      //   if()
      // }


      if (this.selectedElem && this.dragMode) {
        this.selectedElem.x = x;
        this.selectedElem.y = y;
      }
      for (let elem of this.pointsArray) {
        if (x <= elem.x + radius && x >= elem.x - radius && y <= elem.y + radius && y >= elem.y - radius) {
          this.selectedElem = elem;
          this.canvasElem.style.cursor = "pointer";

          return this.drawPointsArray(this.vueCanvas, this.regionMode, elem, linePoint);
        }
      }
      this.canvasElem.style.cursor = "default";
      this.selectedElem = null;
      this.dragMode = false;
      return this.drawPointsArray(this.vueCanvas, this.regionMode, undefined, linePoint);
    },
    mouseRightButton: function (event) {
      let rect = event.target.getBoundingClientRect();
      let x = event.clientX - rect.left;
      let y = event.clientY - rect.top;
      let radius = 7;
      for (let elem of this.pointsArray) {
        if (x <= elem.x + radius && x >= elem.x - radius && y <= elem.y + radius && y >= elem.y - radius) {
          this.pointsArray.splice(this.pointsArray.indexOf(elem), 1);
          return;
        }
      }
    }
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
  font-family: 'Segoe UI', sans-serif;
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
}

#drawer {
  border: 1px solid black;
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
