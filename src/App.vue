<template>
  <div>
    <div>
      -------------------------<br>
      Some content is placed here, a window-menu, for example<br>
      -------------------------
    </div>
    <svgcanvas>
      <g>
        <rect
           id="id0"
           fill="none"
           stroke="red"
           stroke-width="1"
           width="2000"
           height="1000"
           x="-1000"
           y="-500" />
        <rect
           id="id1"
           fill="gray"
           stroke="yellowgreen"
           stroke-width="1"
           width="200"
           height="200"
           x="-200"
           y="-200" />
        <rect
           id="id2"
           fill="gray"
           stroke="yellowgreen"
           stroke-width="1"
           width="200"
           height="200"
           x="200"
           y="-200" />
        <rect
           id="id3"
           fill="gray"
           stroke="yellowgreen"
           stroke-width="1"
           width="200"
           height="200"
           x="-400"
           y="0" />
        <rect
           id="id4"
           fill="gray"
           stroke="yellowgreen"
           stroke-width="1"
           width="200"
           height="200"
           x="0"
           y="0" />
      </g>
    </svgcanvas>
  </div>
</template>

<script>
import svgcanvas from './components/SvgCanvas'

export default {
  name: 'app',
  components: {
    svgcanvas
  },
  methods: {
    menuEventHandler (item) {
      console.log('itemActivateName: ' + item.itemName)
    }
  }
}
</script>

<style>
#app {
  margin: 0;
}
</style>
