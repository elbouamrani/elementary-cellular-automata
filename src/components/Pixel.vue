<template>
    <div class="pixel" :style="style()"></div>
</template>

<script>
export default {
    name: "Pixel",
    props: ["state"],
    methods: {
        style() {
            return { backgroundColor: this.state == 1 ? "black" : "white" };
        },
    },
};
</script>

<style>
.pixel {
    height: var(--pixel-size);
    width: var(--pixel-size);
    border: 0.2pt solid rgb(49, 49, 49);
}
</style>