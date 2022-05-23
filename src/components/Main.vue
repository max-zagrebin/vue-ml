<template>
  <div class="page">
    <!--    <img class="back-image" src="assets/example.png" alt="example-pic">-->
    <canvas width="150" height="150" id="drawer" @click="clickHandler">Canvas fallback text</canvas>
  </div>

</template>

<script>
export default {
  name: 'MainComponent',
  props: {},
  data: function () {
    return {
      vueCanvas: null,
      pointsArray:[]
    };
  },
  mounted() {
    let c = document.getElementById("drawer");
    if (!c.getContext) return;
    let ctx = c.getContext("2d");
    this.vueCanvas = ctx;


    // this.vueCanvas.rect(20, 20, 100, 100);
    //
    //
    // ctx.fillStyle = "rgba(0, 0, 200, 0.1)";
    // ctx.fillRect (30, 30, 55, 50);
    // this.vueCanvas.beginPath();
    // this.vueCanvas.moveTo(75,25);
    // var rectangle = new Path2D();
    // rectangle.rect(10, 10, 50, 50);
    // ctx.stroke(rectangle);


  },
  methods: {
    clickHandler: function (event) {
      // console.log("JOPA");
      console.log(event);
      // console.log(event.pageX);
      // console.log(event.pageY);
      let rect = event.target.getBoundingClientRect();
      let x = event.clientX - rect.left;
      let y = event.clientY - rect.top;
      console.log("Coordinate x: " + x,
          "Coordinate y: " + y);
      this.pointsArray.push({x, y});
      this.drawPoint(this.vueCanvas, x, y);
    },
    drawPoint: function (ctx, x, y) {
      ctx.beginPath();
      ctx.arc(x, y, 7, 0, Math.PI * 2, true);
      ctx.fillStyle = "#000000";
      ctx.fillStyle = "#FFFFFF";
      ctx.fill();
      ctx.arc(x, y, 4, 0, Math.PI * 2, false);
      ctx.fillStyle = "#2F80ED";
      ctx.fill();
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
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
