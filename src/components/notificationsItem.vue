<template>
  <div
    v-if="swiped == false || inMenu == true"
    v-touch="{
      left: () => (this.swiped = true),
      right: () => (this.swiped = true),
    }"
    class="notificationContainer mb-1 mx-1 px-2 pt-1 rounded"
  >
    <div class="notificationInfo d-flex">
      <div class="notificationSender">
        <v-chip label small :color="userColor"
          ><v-icon class="mr-2">mdi-account-circle</v-icon
          >{{ messageSender }}</v-chip
        >
      </div>
      <div class="notificationDate ml-auto font-weight-light text-subtitle-2">
        {{ messageDate }}
      </div>
    </div>
    <div class="px-2 mt-1">
      {{ message }}
    </div>
  </div>
</template>

<script>
export default {
  props: {
    sender: Object,
    time: String,
    message: String,
    inMenu: Boolean,
  },
  data: () => ({
    swiped: false,
  }),
  computed: {
    messageSender() {
      let firstName = this.sender.name.split(" ")[0];
      let lastName = this.sender.name.split(" ")[1][0];
      return firstName + " " + lastName;
    },
    messageDate() {
      let hourMin = { hour: "2-digit", minute: "2-digit" };
      let date = new Date(this.time);
      return date.toLocaleTimeString("es-ES", hourMin);
    },
    userColor() {
      if (this.sender.username !== "mbuitragoc") {
        return "#797fb9";
      }
      return "#3fbfde";
    },
  },
};
</script>

<style scoped>
.notificationContainer {
  background-color: #fbfbfb;
}
</style>
