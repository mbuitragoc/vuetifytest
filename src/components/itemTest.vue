<template>
  <div
    class="itemContainer d-flex justify-center mt-1"
    v-click-outside="clikOutside"
  >
    <div v-show="substract" class="substract" @click="substractHandle">-</div>
    <div
      class="item"
      v-touch="{
        left: () => {
          this.add = true;
          this.substract = false;
        },
        right: () => {
          this.add = false;
          this.substract = true;
        },
      }"
    >
      Item Info
    </div>
    <div v-show="add" class="add" @click="addHandle">+</div>
  </div>
</template>

<script>
export default {
  data: () => ({
    add: false,
    substract: false,
    active: false,
  }),
  methods: {
    substractHandle() {
      this.substract = false;
    },
    addHandle() {
      this.add = false;
    },
    clikOutside() {
      (this.add = false), (this.substract = false);
    },
  },
};
</script>

<style scoped>
.item {
  background-color: blue;
  padding: 10px;
}
.substract {
  background-color: red;
  padding: 10px;
}
.add {
  background-color: greenyellow;
  padding: 10px;
}
</style>
