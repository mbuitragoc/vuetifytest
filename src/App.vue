<template>
  <v-app>
    <v-toolbar color="light blue" max-height="48px" elevation="1" class="pa-1">
      <v-toolbar-title>Chats</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-text-field
        @focus="searchClosed = false"
        @blur="searchClosed = true"
        class="mt-3 expandingSearch"
        :class="{ closed: searchClosed && !search }"
        placeholder="Filter"
        dense
        prepend-inner-icon="mdi-magnify"
        v-model="search"
        outlined
        rounded
      >
      </v-text-field>
      <div class="d-flex align-center">
        <v-menu
          v-model="ShowNotifications"
          :close-on-content-click="true"
          :nudge-width="300"
          offset-x
          max-height="400px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn rounded outlined class="mb-3 ml-4" v-bind="attrs" v-on="on"
              ><v-icon>mdi-message-badge</v-icon></v-btn
            >
          </template>
          <v-card>
            <notificationsItem
              v-for="notf in notifications"
              :key="notf._id"
              :sender="notf.u"
              :time="notf.ts"
              :message="notf.msg"
              :in-menu="true"
            ></notificationsItem>
          </v-card>
        </v-menu>
      </div>
    </v-toolbar>

    <v-main class="messageContainer">
      <div v-if="notifications.length>0" class="notfList mt-1">
        <draggable :sort="false">
          <notificationsItem
            v-for="notf in notifications"
            :key="notf._id"
            :sender="notf.u"
            :time="notf.ts"
            :message="notf.msg"
            :in-menu="false"
          ></notificationsItem>
        </draggable>
      </div>
      <div class="chatList mx-1 mb-1 py-1 rounded">
        <chatListItem
          v-for="(channel, key) in activeChats"
          :key="key"
          :name="channel.name"
          :unread="channel.unread || null"
          :lastMessage="channel.lastMessage"
          :lm="channel.lm"
          :type="channel.t"
          :chatActive="activateChatView"
        />
      </div>

      <div class="chatView">
        <v-navigation-drawer
          v-show="activeChatView === true"
          width="100%"
          height="715px"
          v-model="activeChatView"
          fixed
          right
          class="mt-12"
        >
          <template>
            <iframe
              class="mt-2 px-4"
              src="http://192.168.42.89:3000/channel/general?layout=embedded"
              frameborder="0"
              width="100%"
              height="100%"
              id="Chat"
            ></iframe>
          </template>
        </v-navigation-drawer>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import chatListItem from "./components/chatListItem";
import notificationsItem from "./components/notificationsItem.vue";
import axios from "axios";
import draggable from "vuedraggable";

export default {
  name: "App",
  components: {
    chatListItem,
    notificationsItem,
    draggable,
  },
  data: () => ({
    channels: [],
    notifications: [],
    search: "",
    searchClosed: true,
    activeChatView: false,
    ShowNotifications: false,
  }),
  methods: {
    filterSearchItems(arr, query) {
      return arr.filter((el) =>
        el.name.toLowerCase().includes(query.toLowerCase())
      );
    },
    activateChatView() {
      this.activeChatView = true;
    },
    updateUnRead(unReadCount, channelId, arr) {
      for (let i = 0; i < arr.length; i++) {
        if (arr[i]._id === channelId) {
          arr[i].unread = unReadCount;
        }
      }
    },
    async getChannels() {
      let url = "http://192.168.42.89:3000/api/v1/rooms.get";
      let config = {
        method: "get",
        headers: {
          "X-Auth-Token": "i8tJIjKQ3Tazybics55Gv10xd-PZ5XVyA1AGvVb7558",
          "X-User-Id": "797vWWZ5MBsFW5Paz",
        },
      };
      axios.get(url, config).then((res) => (this.channels = res.data.update));
    },
    async getNotifications() {
      let today = new Date();
      today.setHours(0, 0, 0, 0);
      let url = "http://192.168.42.89:3000/api/v1/channels.history";
      let config = {
        method: "get",
        headers: {
          "X-Auth-Token": "i8tJIjKQ3Tazybics55Gv10xd-PZ5XVyA1AGvVb7558",
          "X-User-Id": "797vWWZ5MBsFW5Paz",
        },
        params: {
          roomId: "hdSoFgo9zyodbL7Tg",
          oldest: today.toISOString(),
        },
      };
      axios
        .get(url, config)
        .then((res) => (this.notifications = res.data.messages));
    },
  },
  mounted() {
    window.addEventListener("message", (e) => {
      if (e.data.eventName === "unread-changed-by-subscription") {
        this.updateUnRead(e.data.data.unread, e.data.data.rid, this.channels);
      }
    });
    this.getChannels();
    this.getNotifications();
  },
  computed: {
    activeChats() {
      // filter out direct messages and empty channels
      let activeChats = this.channels.filter(
        (elem) =>
          elem.t !== "d" && elem.msgs !== 0 && elem._id !== "hdSoFgo9zyodbL7Tg"
      );
      // sort by date without instance to improve perf
      let sortedChats = activeChats.sort(
        (a, b) => Date.parse(b.lm) - Date.parse(a.lm)
      );
      // user is searching from the bar
      if (this.search.length > 0) {
        return this.filterSearchItems(sortedChats, this.search);
      }
      return sortedChats;
    },
  },
};
</script>
<style scoped lang="sass">
.messageContainer
  background-color: #aae0ff
.chatList
  // height: 60%
  // overflow: scroll
  background-color: white
.v-input.expandingSearch
  transition: max-width 0.3s
  &.closed
      max-width: 70px
</style>
