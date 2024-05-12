<script>
import { onMounted, ref } from 'vue'
import NoiseMap from 'noise-map'
export default {
  data() {
    return {
      bitja: [],
      hrana: [],
      bitjaArray: [],
      master: null,
      N: "50",
      H: "25",
      E: "0",
      S: "10",
      V: "5",
      Z: "10",
      st: 0,
      heightmap: null,
      img: null,
    };


  },
  mounted() {
    this.lw = this.$refs.el.getContext("2d");

    // const img = new Image();
    //img.src = "../assets/terrain1.png"; 
    this.img = document.getElementById("terr");
    setTimeout(()=>{
      this.lw.drawImage(this.img, 0, 0, 600, 600);
    })
 
   
    //this.drawTerrain()
  },
  methods: {
    quit() {
      this.master.quit();
    },
    start() {
      // Implement the start logic
    },
    newCycle() {
      this.startDrawing()

    },
    drawTerrain() {
      var generator = new NoiseMap.MapGenerator();
      generator.setSeed(1000);
      this.heightmap = generator.createMap(600, 600, { type: 'perlin' });
      this.heightmap.draw(this.lw, 600, 600, NoiseMap.STYLE.REALISTIC,false);
    },
    startDrawing() {
      var draw = () => {
        if (Date.now() < (this.timestamp + 45)) return requestAnimationFrame(draw);
        this.lw.clearRect(0, 0, 600,600);
        this.lw.drawImage(this.img, 0, 0, 600, 600);
        this.bitja.forEach((dot, index, object) => {
          let _x = this.randomNum();
          let _y = this.randomNum()
   
          if(this.checkIfOnLand(dot.x + _x,dot.y +_y) && dot.tipBitja == 0){
            dot.x += _x
            dot.y += _y
          }
          else if(!this.checkIfOnLand(dot.x + _x,dot.y +_y) && dot.tipBitja == 1){
            dot.x += _x
            dot.y += _y 
          }


          if (!this.checkIfBitjeTouchedAnother(dot)) {
            this.drawDot(dot);
          }
          else {
            object.splice(index, 1);
          }

        });
        this.checkIfBitjeTouchedFood()
        this.hrana.forEach(element => {
          this.drawDot(element)
        });
        this.timestamp = Date.now();
        requestAnimationFrame(draw);
      }
      draw();
    },
    checkIfBitjeTouchedFood() {
    // Check if bitje touched food
      const bitje = this.bitja;
      const hrana = this.hrana;

      for (let index = 0; index < bitje.length; index++) {
        for (let index2 = 0; index2 < hrana.length; index2++) {
          const bitjeX = bitje[index].x + bitje[index].zaznava;
          const bitjeY = bitje[index].y + bitje[index].zaznava;

          if (this.circle(bitjeX, bitjeY, hrana[index2].x, hrana[index2].y, bitje[index].velikost + bitje[index].zaznava, 5)) {
            this.bitja[index].hrana = 1000;
            this.hrana.splice(index2, 1);
          }
        }
      }
    },
    checkIfBitjeTouchedAnother(_bitje) {
      // Check if bitje touched another bitje
      for (let index = 0; index < this.bitja.length; index++) {
        let bitje = this.bitja[index];
        // Check if other bitje is of the same gender and size
        if (_bitje.spol !== bitje.spol && _bitje.plen === bitje.plen) {
          if (this.circle(bitje.x + bitje.zaznava, bitje.y + bitje.zaznava, _bitje.x, _bitje.y, 25, 25)) {
            //console.log("touched");
            return true;
          }
        }
        // Check if other bitje is of different gender and same size
        else if (_bitje.plen !== bitje.plen) {
          if (this.circle(bitje.x + bitje.zaznava, bitje.y + bitje.zaznava, _bitje.x, _bitje.y, 25, 25)) {
            if (_bitje.plen === 1) {
              _bitje.alive = false;
              bitje.hrana = 1000;
            } else {
              bitje.alive = false;
              _bitje.hrana = 1000;
            }
          }
        }
      }
    },

    // Function for circle detection
    circle(x1, y1, x2, y2, r1, r2) {
      let d = Math.sqrt(Math.pow((x1 - x2), 2) + Math.pow((y1 - y2), 2));
      if (d <= r1 - r2 || d <= r2 - r1 || d < r1 + r2 || d === r1 + r2) {
        return true;
      } else {
        return false;
      }
    },

    randomNum() {
      return Math.random() < 0.5 ? -1 : 1;
    },

    drawDot(dot) {
      this.lw.beginPath();
      this.lw.arc(dot.x, dot.y, parseInt(this.V), 0, 2 * Math.PI);
      this.lw.fillStyle = dot.fillColor;
      this.lw.fill();
      this.lw.closePath();
    },
    getRandomPosition(ctx, tipBitja) {
      let targetColor;
      const targetColorSea = [67, 110, 54]; // Red color
      const targetColorLand = [12, 69, 98]; // Green color
      if(tipBitja == 0){
        targetColor = targetColorSea
      }
      else{
        targetColor = targetColorLand
      }
      while (true) {
        const x = Math.floor(Math.random() * 600);
        const y = Math.floor(Math.random() * 600);
        const c = ctx.getImageData(x, y, 1, 1).data;
        const red = c[0];
        const green = c[1];
        const blue = c[2];
        if (red === targetColor[0] && green === targetColor[1] && blue === targetColor[2]) {
          return { x, y };
        }
      }
    },
    checkIfOnLand(x,y){
      const ctx = this.$refs.el.getContext("2d");
      const targetColorSea = [67, 110, 54]; // Red color
      const targetColorLand = [12, 69, 98]; // Green color
      const c = ctx.getImageData(x, y, 1, 1).data;
      const red = c[0];
      const green = c[1];
      const blue = c[2];
      
      if(red == 67 && green == 110 && blue == 54){
        return true
      }
      else{
        return false
      }


    },
    generateFood(){
      const ctx = this.$refs.el.getContext("2d");
      for (let i = 0; i < parseInt(this.H); i++) {
        let fillColor = null;
        let _x,_y
        const tipBitja = Math.floor(Math.random() * 2);
        console.log(tipBitja)
        if (tipBitja === 0) {
          const { x, y } = this.getRandomPosition(ctx, tipBitja);
          _x = x;
          _y = y;
          fillColor = "#EADDCA";
        } else {
          const { x, y } = this.getRandomPosition(ctx, tipBitja);
          _x = x
          _y = y
          fillColor = "#E1C16E";
        }

        this.hrana.push({
          tipBitja: tipBitja,
          x: _x,
          y: _y,
          zeja: 1000,
          hrana: 1000,
          plen: Math.floor(Math.random() * 4),
          spol: Math.floor(Math.random() * 2),
          velikost: parseInt(this.V),
          hitrost: parseInt(this.S),
          zaznava: parseInt(this.Z),
          alive: true,
          razmnozevanje: 1000,
          hasChild: 0
        });
        this.hrana[i].fillColor = fillColor
      }
      this.hrana.forEach(element => {
        this.drawDot(element)
      });
    },
    generateBitja() {
      const ctx = this.$refs.el.getContext("2d");
      for (let i = 0; i < parseInt(this.N); i++) {
        let fillColor = null;
        let _x,_y
        const tipBitja = Math.floor(Math.random() * 2);
        console.log(tipBitja)
        if (tipBitja === 0) {
          const targetColor = [12, 69, 98]; // Green color
          const { x, y } = this.getRandomPosition(ctx, tipBitja);
          _x = x;
          _y = y;
          fillColor = "green";
        } else {
          const targetColor = [67, 110, 54]; // Red color
          const { x, y } = this.getRandomPosition(ctx, tipBitja);
          _x = x
          _y = y
          fillColor = "red";
        }

        this.bitja.push({
          tipBitja: tipBitja,
          x: _x,
          y: _y,
          zeja: 1000,
          hrana: 1000,
          plen: Math.floor(Math.random() * 4),
          spol: Math.floor(Math.random() * 2),
          velikost: parseInt(this.V),
          hitrost: parseInt(this.S),
          zaznava: parseInt(this.Z),
          alive: true,
          razmnozevanje: 1000,
          hasChild: 0
        });
        this.bitja[i].fillColor = fillColor
      }
      this.bitja.forEach(element => {
        this.drawDot(element)
      });

      this.generateFood()
    },
    //DEBUG
    handleClick(event) {
      const canvas = this.$refs.el
      const rect = canvas.getBoundingClientRect();
      const x = event.clientX - rect.left;
      const y = event.clientY - rect.top;
      this.getPixelColor(x, y);
    },
    getPixelColor(x, y) {
      const ctx = this.$refs.el.getContext("2d");
      const imageData = ctx.getImageData(x, y, 1, 1);
      const pixel = imageData.data;
      const red = pixel[0];
      const green = pixel[1];
      const blue = pixel[2];
      console.log(`Pixel color at (${x}, ${y}): rgba(${red}, ${green}, ${blue})`);
    },
  }
};

</script>

<template>
  <div>
  <div>
    <div>
      <label>St bitij</label>
      <input v-model="N" />
    </div>
    <div>
      <label>St hrane</label>
      <input v-model="H" />
    </div>
    <div>
      <label>Izbira terena</label>
      <input v-model="E" />
    </div>
    <div>
      <label>hitrost</label>
      <input v-model="S" />
    </div>
    <div>
      <label>velikost</label>
      <input v-model="V" />
    </div>
    <div>
      <label>zaznava</label>
      <input v-model="Z" />
    </div>
    <label>ST</label>
    <label>{{ st }}</label>
    <button @click="generateBitja">Generiraj bitja</button>
    <button @click="newCycle">Začni</button>
  </div>
  <canvas @click="handleClick" id="c" ref="el" width="600" height="600" style="background-color: whitesmoke;" />
  <img id="terr" style="display: none;" src="../assets/_terrain1.jpg">
</div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
