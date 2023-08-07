<template>
  <div class="container mt-4">
    <div class="main-row">
      <div style="display: inline-block" class="mr-4">
        <div
            class="image-container position-relative mb-3"
            style="width: 768px; height: 432px;"
            @mousemove="moveMagnifier"
            @mouseenter="mouseEnter"
            @mouseleave="mouseLeave"
            @wheel="zooming"
        >
          <img ref="image" :src="imageSrc" alt="City" class="img-fluid" />
          <div id ='magnifier' v-if="showMagnifier" :style="magnifier"></div>
        </div>
      </div>
      <div id="zoom-container" style="display: inline-block">
        <div class="zoom-control mb-4">
          <div class="d-flex align-items-center" style="display: inline-block">
            <i class="bi bi-search"></i>
            <label for="zoom" class="label m-2">
              Zoom: {{ zoom }}x
            </label>
          </div>
          <div class="vertical-input" style="display: inline-block">
            <input
                id="zoom"
                class="form-range"
                type="range"
                min="1"
                max="10"
                step="1"
                v-model="zoom"
            />
          </div>
        </div>
      </div>
  </div>
    <div class="color-controls d-flex">
      <div v-for="(color, index) in getRGB" :key="index" class="mx-4">
        <i class="bi bi-brightness-low-fill" :style="{ color: color }"></i>
        <label class="label m-2">{{ color }}: {{ colorValues[color] }}%</label>
        <input
            :id="color"
            class="form-range"
            type="range"
            step="1"
            min="0"
            max="100"
            :value="colorValues[color]"
            @input="changeColor(color, $event.target.value)"
        />
      </div>
    </div>
  </div>
</template>

<script>

export default {
  data() {
    return {
      imageSrc: require('@/assets/hague.jpg'),
      zoom: 2,
      showMagnifier: false,
      magnifier: {},
      isMagnifierEntered:false,
      activeColor: "red",
      colorValues: {
        red: 100,
        green: 100,
        blue: 100,
      },
    };

  },
  computed: {
    getRGB() {
      return Object.keys(this.colorValues);
    },
  },
  mounted() {
    window.addEventListener("keydown", this.keyDown);
  },
  methods: {
    mouseEnter() {
      this.showMagnifier = true;
    },

    mouseLeave() {
      this.showMagnifier = false;
      this.isMagnifierEntered = false;
      this.magnifier = {};
    },

    changeColor(color, value) {
      this.colorValues[color] = value;
      this.activeColor = color;
      this.reColor();
    },
    keyDown(e) {
      e.preventDefault();
      if (e.code === "KeyR") {
        this.activeColor = "red";
      } else if (e.code === "KeyG") {
        this.activeColor = "green";
      } else if (e.code === "KeyB") {
        this.activeColor = "blue";
      }else{
        this.arrowsKeyDown(e)
      }
    },

    arrowsKeyDown(e) {
      if (e.code === 'ArrowLeft') {
          this.colorValues[this.activeColor] = Math.max(0, this.colorValues[this.activeColor] - 1);
      } else if (e.code === 'ArrowRight') {
          this.colorValues[this.activeColor] = Math.min(100,  parseInt(this.colorValues[this.activeColor]) + 1);
      }else return;
      this.reColor();
      this.magnifier = {...this.magnifier,
        backgroundColor: `rgb(${this.colorValues.red}%, ${this.colorValues.green}%, ${this.colorValues.blue}%)`,
      }
    },

    moveMagnifier(e) {

      const magnifierSize = 100;
      const magnifierCursorDistance = 50;
      let cursorX = e.offsetX;
      let cursorY = e.offsetY;
      if (e.target.getAttribute('id') === 'magnifier') {
        cursorX = e.target.offsetLeft;
        cursorY = e.target.offsetTop;
      }
      const imgWidth = this.$refs.image.width;
      const imgHeight = this.$refs.image.height;

      const magnifierX = this.calculateMagnifierPosition(cursorX, magnifierCursorDistance, magnifierSize, imgWidth);
      const magnifierY = this.calculateMagnifierPosition(cursorY, magnifierCursorDistance, magnifierSize, imgHeight);
      
      const bgPosX = this.calculateBackgroundPosition(cursorX, this.zoom, imgWidth, magnifierSize);
      const bgPosY = this.calculateBackgroundPosition(cursorY, this.zoom, imgHeight, magnifierSize);
      
      this.magnifier = {
        backgroundImage: `url(${this.imageSrc})`,
        backgroundPosition: `${bgPosX}px ${bgPosY}px`,
        backgroundSize: `${imgWidth * this.zoom}px ${imgHeight * this.zoom}px`,
        backgroundRepeat: "no-repeat",
        border: "1px solid black",
        borderRadius: "50%",
        width: `${magnifierSize}px`,
        height: `${magnifierSize}px`,
        position: "absolute",
        left: `${magnifierX}px`,
        top: `${magnifierY}px`,
        backgroundBlendMode: "multiply",
        backgroundColor: `rgb(${this.colorValues.red}%, ${this.colorValues.green}%, ${this.colorValues.blue}%)`,
      };
      if (!this.isMagnifierEntered) {
        this.isMagnifierEntered = true;
      } else {
        this.magnifier = {
          ...this.magnifier,
          transitionDuration: "0.5s",
          transitionTimingFunction: "ease-out",
        };
      }
    },

    calculateMagnifierPosition(cursor, magnifierCursorDistance, magnifierSize,imageSize) {
      const magnifierOffset = cursor + magnifierCursorDistance + magnifierSize > imageSize ? -magnifierCursorDistance*2 : magnifierCursorDistance;
      return cursor + magnifierOffset;
    },

    calculateBackgroundPosition(cursor, zoom, imageSize, magnifierSize) {
      return Math.min(0, Math.max(-(cursor * zoom - magnifierSize / 2), -(imageSize * zoom - magnifierSize)));
    },

    zooming(e) {
      e.preventDefault();
      const scrollDirectionValue = Math.sign(e.deltaY);
      const newZoom = this.zoom - scrollDirectionValue;
      this.zoom = Math.min(10, Math.max(1, newZoom));
      this.moveMagnifier(e);
    },

    reColor() {
      const canvas = document.createElement("canvas");
      canvas.width = 768;
      canvas.height = 432;
      const starterXAndY = 0;
      const activeColorIndex = this.getActiveColorIndex();
      const hundredthOfActiveColorValue = this.colorValues[`${this.activeColor}`] * 0.01

      const context = canvas.getContext("2d");
      context.drawImage(this.$refs.image, starterXAndY, starterXAndY, canvas.width, canvas.height);

      const imageData = context.getImageData(starterXAndY, starterXAndY, canvas.width, canvas.height);
      const data = imageData.data;

      if (!this.originalData) {
        this.originalData = new Uint8ClampedArray(data);
      }

      for (let i = 0; i < data.length; i += 4) {
        data[i+activeColorIndex] = this.originalData[i+activeColorIndex] * hundredthOfActiveColorValue;
      }

      context.putImageData(imageData, starterXAndY, starterXAndY);
      this.$refs.image.src = canvas.toDataURL();
    },

    getActiveColorIndex(){
      return this.getRGB.indexOf(this.activeColor);
    }
  },
};
</script>

<style>

.form-range[id="red"]::-webkit-slider-runnable-track {
  background-color: red;
}

.form-range[id="green"]::-webkit-slider-runnable-track {
  background-color: green;
}

.form-range[id="blue"]::-webkit-slider-runnable-track {
  background-color: blue;
}

.form-range[id="zoom"]::-webkit-slider-runnable-track {
  background-color: yellow;
}

.form-range {
  width: 100%;
}

.vertical-input {
  margin-left:20%;
  transform: rotate(-90deg);
  width:140%
}

#zoom-container{
  position: relative;
  left:4cm
}

</style>