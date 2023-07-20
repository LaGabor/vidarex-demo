<template>
  <div class="container mt-4">
    <div class="row">
      <div class="col-md-8">
        <div
            class="image-container position-relative"
            style="width: 768px; height: 432px;"
            @mousemove="moveMagnifier"
            @mouseenter="mouseEnter"
            @mouseleave="mouseLeave"
            @wheel="zooming"
        >
          <img ref="image" :src="imageSrc" alt="City" class="img-fluid" />
          <div v-if="showMagnifier" :style="magnifier"></div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="zoom-control mb-4">
          <div class="d-flex align-items-center">
            <i class="bi bi-search"></i>
            <label for="zoom" class="label m-2">
              Zoom: {{zoom}}x
            </label>
          </div>
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
        <div class="color-controls">
          <div v-for="(color, index) in getRGB" :key="index">
            <i class="bi bi-brightness-low-fill" :style="{ color: color }"></i>
            <label class="label m-2">
              {{ color }}: {{ colorValues[color]}}%
            </label>
            <input
                :id="color"
                class="form-range"
                type="range"
                step="1"
                :min="0"
                :max="100"
                :value="colorValues[color]"
                @input="changeColor(color, $event.target.value)"
            />
          </div>
        </div>
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
      this.magnifier = {};
    },

    changeColor(color, value) {
      if (color === "red") {
        this.colorValues.red = value;
      } else if (color === "green") {
        this.colorValues.green = value;
      } else if (color === "blue") {
        this.colorValues.blue = value;
      }
      this.activeColor = color;
    },
    keyDown(e) {
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
        if (this.activeColor === "red") {
          this.colorValues.red = Math.max(0, this.colorValues.red - 1);
        } else if (this.activeColor === "green") {
          this.colorValues.green = Math.max(0, this.colorValues.green - 1);
        } else if (this.activeColor === "blue") {
          this.colorValues.blue = Math.max(0, this.colorValues.blue - 1);
        }
      } else if (e.code === 'ArrowRight') {
        if (this.activeColor === "red") {
          this.colorValues.red = Math.min(100, this.colorValues.red + 1);
        } else if (this.activeColor === "green") {
          this.colorValues.green = Math.min(100, this.colorValues.green + 1);
        } else if (this.activeColor === "blue") {
          this.colorValues.blue = Math.min(100, this.colorValues.blue + 1);
        }
      }
    },

    moveMagnifier(e) {

      const magnifierSize = 100;
      const magnifierCursorDistance = 50;
      const { left: imgLeft, top: imgTop } = this.$refs.image.getBoundingClientRect();
      const cursorX = e.pageX - imgLeft;
      const cursorY = e.pageY - imgTop;
      const imgWidth = this.$refs.image.width;
      const imgHeight = this.$refs.image.height;

      const magnifierX = this.calculateMagnifierPosition(cursorX, magnifierCursorDistance, magnifierSize,imgWidth);
      const magnifierY = this.calculateMagnifierPosition(cursorY, magnifierCursorDistance, magnifierSize,imgHeight);

      const bgPosX = this.calculateBackgroundPosition(cursorX, this.zoom, imgWidth, magnifierSize);
      const bgPosY = this.calculateBackgroundPosition(cursorY, this.zoom, imgHeight, magnifierSize);

      this.magnifier = {
        cursor: "none",
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
      };
    },
    calculateMagnifierPosition(cursor, magnifierCursorDistance, magnifierSize,imageSize) {
      const magnifierOffset = cursor + magnifierCursorDistance + magnifierSize > imageSize ? -magnifierCursorDistance : magnifierCursorDistance;
      return cursor - magnifierSize / 2 + magnifierOffset;
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
  },
};
</script>

<style scoped>

.image-container {
  overflow: hidden;
}

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

</style>