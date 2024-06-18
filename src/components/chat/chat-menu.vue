<style lang="scss">
.chat-menu-list {
  .q-hoverable:hover > .q-focus-helper {
    background: #e9eff5;
    opacity: 0.6;
  }
}
</style>

<template>
  <div class="h-flex h100p">
    <div class="pa-3 al-c">
      <a href="/" class="mr-auto">
        <img
          src="https://dashboard.4everland.org/img/svg/logo-m.svg"
          style="height: 24px"
        />
      </a>
      <q-btn dense flat @click="addMenu">
        <img src="/img/edit.svg" width="22" />
      </q-btn>
    </div>
    <q-scroll-area
      class="flex-1 w100p"
      :thumb-style="{
        right: '2px',
        width: '3px',
        opacity: 0.35,
      }"
    >
      <div class="pa-3 chat-menu-list">
        <div class="mb-1" v-for="(it, i) in chatMenus" :key="it.id">
          <q-btn
            class="w100p"
            :class="{
              'bg-btn-on': path == '/' && i == menuIdx,
            }"
            flat
            @click="onMenu(i)"
          >
            <div class="w100p al-c">
              <span class="line-1">{{ it.title || "New Conversation" }}</span>
            </div>
          </q-btn>
        </div>
      </div>
    </q-scroll-area>
    <div class="pa-3">
      <q-btn v-if="!token" color="primary" class="w100p" @click="onLogin"
        >Sign in</q-btn
      >
      <q-btn v-if="!apiKey" color="info" class="w100p mt-3" @click="onImport"
        >Import a key</q-btn
      >
    </div>
    <div class="bg-btn-1 al-c pa-3" @click="$router.push('/settings')">
      <jazz-icon v-if="userInfo.uid" :hash="userInfo.uid" :size="24" />
      <img v-else src="/img/chat/avatar.svg" width="24" />
      <span class="ml-2">{{ userInfo.uname || "Visitor" }}</span>
      <img class="ml-auto" src="/img/chat/settings.svg" width="22" />
    </div>
  </div>
</template>

<script>
import md5 from "md5";
import { mapState } from "vuex";

export default {
  data() {
    return {
      loadingUser: false,
    };
  },
  computed: {
    ...mapState({
      token: (s) => s.loginData.token,
      userInfo: (s) => s.userInfo,
      chatMenus: (s) => s.chatMenus,
      menuIdx: (s) => s.menuIdx,
      apiKey: (s) => s.apiKey,
    }),
    path() {
      return this.$route.path;
    },
  },
  created() {
    if (this.token && !this.userInfo.uid) {
      this.getUserInfo();
    }
    if (!this.chatMenus.length) {
      this.addMenu();
    }
  },
  methods: {
    onLogin() {
      // this.$router.replace("/login");
      location.href = this.$getHomeUrl("/quick-login?type=chat");
    },
    onImport() {
      this.$bus.emit("show-import");
    },
    onMenu(i) {
      if (this.path != "/") {
        this.$router.push("/");
      }
      this.$setStore({
        menuIdx: i,
      });
      this.$bus.emit("chat-focus");
      this.$bus.emit("close-left");
    },
    addMenu() {
      const row = this.chatMenus[0];
      if (!row || row.title) {
        const rand = (Math.random() + "").substring(2, 6);
        const id = "chat-" + md5(Date.now() + rand).substring(0, 6);
        this.$setStore({
          chatMenus: [
            {
              id,
            },
            ...this.chatMenus,
          ],
        });
      }
      this.onMenu(0);
    },
    async getUserInfo() {
      try {
        this.loadingUser = true;
        const { data } = await this.$http.get("$auth/user");
        const { uid = "", username } = data;
        data.uname = (username || uid).cutStr(4, 4);
        this.$setStore({
          userInfo: data,
        });
      } catch (error) {
        console.log(error);
      }
      this.loadingUser = false;
    },
  },
};
</script>