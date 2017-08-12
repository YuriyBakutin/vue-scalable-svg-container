<template>
  <div class="canvasWrap">
    <div
     class="workframe"
     ref="workframe"
     v-bind:style="frameSizeString"
     v-on:scroll="svgViewBoxUpdate">
      <div
       class="scrollableRect"
       v-bind:style="scrollableRectSizeString">
      </div>
    </div>
    <div class="svgWrap" ref="svgWarp">
      <svg
       xmlns="http://www.w3.org/2000/svg"
       v-bind:width="frameWidth"
       v-bind:height="frameHeight"
       v-bind:viewBox="svgViewBoxString">
        <slot></slot>
      </svg>
    </div>
  </div>
</template>

<script>
export default {
  name: 'svgcanvas',
  data: function () {
    return {
      windowWidth: 0,
      windowHeight: 0,

      frameSizeWidth: 0,
      frameSizeHeight: 0,

      frameWidth: 0,
      frameHeight: 0,

      frameOffsetLeft: 0,
      frameOffsetTop: 0,

      scrollableRectWidth: 0,
      scrollableRectHeight: 0,

      svgViewBoxLeft: 0,
      svgViewBoxTop: 0,
      svgViewBoxWidth: 0,
      svgViewBoxHeight: 0,

      scaleRatio: 1,

      scrollLeft: 0,
      scrollTop: 0
    }
  },
  props: {
    baseSvgWidth: {
      type: [Number, String],
      default: 2000
    },
    baseSvgHeight: {
      type: [Number, String],
      default: 1000
    },
    baseSvgLeft: {
      type: [Number, String],
      default: null
    },
    baseSvgTop: {
      type: [Number, String],
      default: null
    },
    initialScaleRatuo: {
      type: [Number, String],
      default: 1
    },
    onwheelStepScaleRatio: {
      type: [Number, String],
      default: 1.1
    },
    maxScaleRatio: {
      type: [Number, String],
      default: 16
    }
  },
  computed: {
    frameSizeString: function () {
      if (this.frameSizeWidth && this.frameSizeHeight) {
        return `width: ${this.frameSizeWidth}px; ` +
               `height: ${this.frameSizeHeight}px;`
      }
    },
    scrollableRectSizeString: function () {
      return `width: ${this.scrollableRectWidth}px;` +
             `height: ${this.scrollableRectHeight}px;`
    },
    svgViewBoxString: function () {
      return `${this.svgViewBoxLeft} ` +
             `${this.svgViewBoxTop} ` +
             `${this.svgViewBoxWidth} ` +
             `${this.svgViewBoxHeight}`
    }
  },
  methods: {
    setFrameSizes () {
      this.windowWidth = document.documentElement.clientWidth
      this.frameSizeWidth = this.windowWidth - this.frameOffsetLeft

      this.windowHeight = document.documentElement.clientHeight
      this.frameSizeHeight = this.windowHeight - this.frameOffsetTop

      this.$nextTick(function () {
        this.frameWidth = this.$refs.workframe.clientWidth
        this.frameHeight = this.$refs.workframe.clientHeight

        this.svgViewBoxUpdate()
      })
    },
    svgViewBoxUpdate () {
      this.svgViewBoxLeft = (this.baseSvgLeft
                             ? this.baseSvgLeft
                             : -this.baseSvgWidth / 2) +
                            this.$refs.workframe.scrollLeft /
                            this.scaleRatio
      this.svgViewBoxTop = (this.baseSvgTop
                            ? this.baseSvgTop
                            : -this.baseSvgHeight / 2) +
                            this.$refs.workframe.scrollTop /
                            this.scaleRatio
      this.svgViewBoxWidth = this.frameWidth / this.scaleRatio
      this.svgViewBoxHeight = this.frameHeight / this.scaleRatio
    },
    scaleRatioUpdate (event) {
      let framePickerX = event.clientX - this.frameOffsetLeft
      let framePickerY = event.clientY - this.frameOffsetTop

      let stepScaleRatio

      if (event.deltaY > 0) {
        stepScaleRatio = 1 / this.onwheelStepScaleRatio
      } else {
        stepScaleRatio = this.onwheelStepScaleRatio
      }

      let canvasPickerX = framePickerX + this.$refs.workframe.scrollLeft
      let canvasPickerY = framePickerY + this.$refs.workframe.scrollTop

      let newScaleRatio = this.scaleRatio * stepScaleRatio

      if (newScaleRatio > this.maxScaleRatio) return

      let newScrollableRectWidth = Math.round(this.baseSvgWidth *
                                           newScaleRatio)
      if (newScrollableRectWidth < this.frameWidth && event.deltaY > 0) return

      let newScrollableRectHeight = Math.round(this.baseSvgHeight *
                                            newScaleRatio)
      if (newScrollableRectHeight < this.frameHeight && event.deltaY > 0) return

      let scrollLeft = Math.round(canvasPickerX * stepScaleRatio -
                                  framePickerX)
      if (scrollLeft < 0) {
        scrollLeft = 0
      }
      if (newScrollableRectWidth - this.scrollLeft < this.frameWidth) {
        this.scrollLeft = newScrollableRectWidth - this.frameWidth
      }

      let scrollTop = Math.round(canvasPickerY * stepScaleRatio -
                                 framePickerY)
      if (scrollTop < 0) {
        scrollTop = 0
      }
      if (newScrollableRectHeight - this.scrollTop < this.frameHeight) {
        this.scrollTop = newScrollableRectHeight - this.frameHeight
      }

      this.scaleRatio = newScaleRatio
      this.scrollableRectWidth = newScrollableRectWidth
      this.scrollableRectHeight = newScrollableRectHeight

      this.$refs.workframe.scrollLeft = scrollLeft
      this.$refs.workframe.scrollTop = scrollTop

      this.svgViewBoxUpdate()
    }
  },
  mounted () {
    this.$nextTick(function () {
      let br = this.$refs.workframe.getBoundingClientRect()
      this.frameOffsetLeft = Math.round(br.left)
      this.frameOffsetTop = Math.round(br.top)
      this.setFrameSizes()

      this.scrollableRectWidth = this.baseSvgWidth * this.scaleRatio
      this.scrollableRectHeight = this.baseSvgHeight * this.scaleRatio

      window.addEventListener('resize', this.setFrameSizes.bind(this))

      this.$refs.svgWarp.addEventListener('wheel',
                                          this.scaleRatioUpdate.bind(this))
    })
  },
  beforDestroy () {
    window.removeEventListener('resize', this.setFrameSizes)
    this.$refs.svgWarp.removeEventListener('wheel',
                                          this.scaleRatioUpdate.bind(this))
  }
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
}
.canvasWrap {
  position: relative;
}
.workframe {
  position: relative;
  width: 100%;
  height: 100%;
  background: #77f;
  overflow-x: scroll;
  overflow-y: scroll;
/*  height: 100px; */

  border: 0;
}
.scrollableRect {
  background-color: black;
}
.svgWrap {
  position: absolute;
  top: 0;
  left: 0;
}
</style>
