<template>
  <v-app>
    <v-toolbar color="light blue" max-height="48px">
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
    </v-toolbar>

    <v-main>
      <notificationsItem></notificationsItem>
      <div class="chatList">
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
          v-model="activeChatView"
          bottom
          fixed
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
import axios from "axios";
import notificationsItem from "./components/notificationsItem.vue";

export default {
  name: "App",
  components: {
    chatListItem,
    notificationsItem,
  },
  data: () => ({
    channels: [],
    search: "",
    searchClosed: true,
    activeChatView: false,
    notifications: [],
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
    getChannels() {
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
    getNotifications() {
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
          oldest: today.toISOString(), // Only returns the notifications of Today
        },
      };
      axios
        .get(url, config)
        .then((res) => (this.notifications = res.data.messages));
    },
  },
  mounted() {
    this.getChannels();
    this.getNotifications();
    window.addEventListener("message", (e) => {
      if (e.data.eventName === "unread-changed-by-subscription") {
        console.log(
          e.data.data.name + ": Number of unreads " + e.data.data.unread
        );
        this.updateUnRead(e.data.data.unread, e.data.data.rid, this.channels);
      }
    });
  },
  computed: {
    activeChats() {
      // filter out direct messages and empty channels
      let activeChats = this.channels.filter(
        (elem) => elem.t !== "d" && elem.msgs !== 0
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

.chatList
  height: 90%
  overflow: scroll

.chatView
  position: absolute

.v-input.expandingSearch
  transition: max-width 0.3s
  &.closed
      max-width: 70px
      .v-input__slot
          background: transparent !important
</style>
