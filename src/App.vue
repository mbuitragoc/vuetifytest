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
            <v-btn
              aria-label="Notifications"
              rounded
              outlined
              class="mb-3 ml-4"
              v-bind="attrs"
              v-on="on"
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
      <!-- <itemTest></itemTest> -->
      <div v-if="notifications.length > 0" class="notfList mt-1">
        <draggable :sort="false">
          <notificationsItem
            v-for="notf in notifications"
            :key="notf._id"
            :sender="notf.u"
            :time="notf.ts"
            :message="notf.msg"
            :in-menu="false"
          >
          </notificationsItem>
        </draggable>
      </div>
      <div class="chatList mx-1 my-2 py-1 rounded">
        <chatListItem
          v-for="(channel, key) in activeChats"
          :key="key"
          :name="channel.name"
          :unread="channel.unread"
          :lastMessage="channel.lastMessage"
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
              title="Chat"
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
// import itemTest from "./components/itemTest.vue";
import axios from "axios";
import draggable from "vuedraggable";

const notificationChannelId = "hdSoFgo9zyodbL7Tg";
const userId = "797vWWZ5MBsFW5Paz";
const authToken = "i8tJIjKQ3Tazybics55Gv10xd-PZ5XVyA1AGvVb7558";
const serverUrl = "http://192.168.42.89:3000/api/v1/";

export default {
  name: "App",
  components: {
    chatListItem,
    notificationsItem,
    draggable,
    // itemTest,
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

    getChannels() {
      let url = serverUrl + "rooms.get";
      let config = {
        method: "get",
        headers: {
          "X-Auth-Token": authToken,
          "X-User-Id": userId,
        },
      };
      axios.get(url, config).then((res) => {
        this.setUnread(res.data.update);
        this.channels = res.data.update;
      });
    },
    setUnread(channelList) {
      for (let i = 0; i < channelList.length; i++) {
        channelList[i].unread = 0;
      }
    },

    async updateUnRead(unReadCount, channelId) {
      for (let i = 0; i < this.channels.length; i++) {
        if (this.channels[i]._id === channelId) {
          let lastMessage = await this.getLastMessage(channelId);
          this.channels[i].unread = unReadCount;
          this.channels[i].lastMessage = lastMessage;
        }
      }
    },
    async getLastMessage(roomId) {
      let channel = this.channels.find((el) => el._id === roomId);
      let config = {
        method: "get",
        headers: {
          "X-Auth-Token": authToken,
          "X-User-Id": userId,
        },
        params: {
          roomId: roomId,
          count: 1,
        },
      };
      if (channel.t === "p") {
        let url = serverUrl + "groups.history";
        let response = await axios.get(url, config);
        return response.data.messages[0];
      } else {
        let url = serverUrl + "channels.history";
        let response = await axios.get(url, config);
        return response.data.messages[0];
      }
    },

    getNotifications() {
      let today = new Date();
      today.setHours(0, 0, 0, 0);
      let url = serverUrl + "channels.history";
      let config = {
        method: "get",
        headers: {
          "X-Auth-Token": authToken,
          "X-User-Id": userId,
        },
        params: {
          roomId: notificationChannelId,
          oldest: today.toISOString(),
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
    window.addEventListener(
      "message",
      (e) => {
        if (e.data.eventName === "unread-changed-by-subscription") {
          if (e.data.data.t !== "d") {
            this.updateUnRead(e.data.data.unread, e.data.data.rid);
            if (e.data.data.rid === notificationChannelId) {
              this.getNotifications();
            }
          }
        }
      },
      { passive: true }
    );
  },

  computed: {
    activeChats() {
      // filter out direct messages and empty channels
      let activeChats = this.channels.filter(
        (elem) =>
          elem.t !== "d" &&
          elem.msgs !== 0 &&
          elem._id !== notificationChannelId
      );
      // sort by date without instance to improve perf
      let sortedChats = activeChats.sort(
        (a, b) => Date.parse(b.lastMessage.ts) - Date.parse(a.lastMessage.ts)
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
