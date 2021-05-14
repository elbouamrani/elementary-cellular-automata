<template>
  <div class="screen">
    <div class="array" v-for="(array, arrayKey) in world" :key="arrayKey">
      <Pixel
        v-for="(pixel, pixelKey) in array"
        :key="pixelKey"
        :state="pixel"
      />
    </div>
  </div>
</template>

<script>
import Pixel from "./Pixel";
export default {
  name: "Screen",
  props: ["world"],
  components: {
    Pixel,
  },
};
</script>

<style>
.array {
  display: flex;
}
</style>
