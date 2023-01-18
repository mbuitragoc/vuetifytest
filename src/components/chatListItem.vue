<template>
  <div
    class="d-flex align-item-center mt-1 mx-1 message rounded"
    @click="
      chatActive();
      goTo();
    "
  >
    <v-avatar tile :color="AvatarColor" class="rounded">
      <v-icon dark> mdi-account-group </v-icon>
    </v-avatar>
    <div class="d-flex flex-column pl-3 overflow-hidden">
      <div
        class="text-capitalize text-h6 font-weight-medium text--primary chatTitle"
      >
        <v-icon v-if="type === 'p'">mdi-lock</v-icon> {{ name }}
      </div>
      <div class="text-truncate text-body-2">
        {{ LastMessageSender }}: {{ LastMessageInfo }}
      </div>
    </div>
    <div class="d-flex flex-column ml-auto justify-space-around">
      <div class="text-caption font-weight-light text-subtitle-2">
        {{ LastMessageTime }}
      </div>
      <v-badge
        :value="unread"
        :content="unread"
        inline
        color="red darken-2"
        class="align-self-end"
      ></v-badge>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    name: String,
    unread: Number,
    lastMessage: Object,
    chatActive: Function,
    type: String,
  },
  data: () => ({}),
  methods: {
    isBeforeToday(date) {
      const today = new Date();
      today.setHours(0, 0, 0, 0);
      return date < today;
    },
    getLastDate() {
      let today = new Date();
      return new Date(
        today.getFullYear(),
        today.getMonth(),
        today.getDay() - 7
      );
    },
    getWeekDay(date) {
      switch (date) {
        case 0:
          return "Domingo";
        case 1:
          return "Lunes";
        case 2:
          return "Martes";
        case 3:
          return "Miercoles";
        case 4:
          return "Jueves";
        case 5:
          return "Viernes";
        case 6:
          return "Sabado";
      }
    },
    goTo() {
      let type = this.parseChannelType(this.type);
      let url = "/" + type + "/" + this.name + "/?layout=embedded";
      document
        .querySelector("#Chat")
        .contentWindow.postMessage(
          { externalCommand: "go", path: url },
          "http://192.168.42.89:3000"
        );
    },
    parseChannelType(type) {
      switch (type) {
        case "c":
          return "channel";
        case "p":
          return "group";
      }
    },
  },
  computed: {
    LastMessageSender() {
      let firstName = this.lastMessage.u.name.split(" ")[0];
      let lastName = this.lastMessage.u.name.split(" ")[1][0];
      return firstName + " " + lastName;
    },
    LastMessageTime() {
      let now = new Date();
      let messageTime = new Date(this.lastMessage.ts);
      let lastWeekDate = this.getLastDate();
      let monthYear = { day: "numeric", month: "short" };
      let hourMin = { hour: "2-digit", minute: "2-digit" };

      //Message correspond to last year and older
      if (messageTime.getFullYear() < now.getFullYear()) {
        return messageTime.toLocaleDateString();
      }
      //Message is older than last week
      if (lastWeekDate > messageTime) {
        return messageTime.toLocaleDateString("es-ES", monthYear);
      }
      //Time of Message is from yesterday
      if (now.getDate() - messageTime.getDate() === 1) {
        return "Ayer";
      }
      //Time of Message corresponds to today
      if (!this.isBeforeToday(messageTime)) {
        return messageTime.toLocaleTimeString("es-ES", hourMin);
      }
      //As Message is within last week show the week day making it easier for the user
      return this.getWeekDay(messageTime.getDay());
    },
    LastMessageInfo(){
      let lastMessage = this.lastMessage
      if(lastMessage.md.length > 1){
        return lastMessage.md[1].value[0].value
      }
      return lastMessage.msg
    },
    AvatarColor() {
      // returns a random color for the avatar item
      return (
        "#" +
        Math.floor(Math.abs(Math.sin(Math.random()) * 16777215))
          .toString(16)
          .padStart(6, 0)
      );
    },
  },
};
</script>

<style scoped>
/* @import url("https://fonts.googleapis.com/css2?family=Montserrat:wght@100;200;300;400;500;600;700;800;900&family=Source+Sans+Pro:wght@200;300;400;600;700;900&display=swap"); */
.message {
  padding: 5px;
  background-color: #f7f7f7;
}

.chatTitle {
  font-family: "Montserrat", sans-serif;
}
</style>
