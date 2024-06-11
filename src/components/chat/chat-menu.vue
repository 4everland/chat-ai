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
      class="flex-1"
      :thumb-style="{
        right: '2px',
        width: '3px',
        opacity: 0.35,
      }"
    >
      <div class="pa-3">
        <div class="mb-1" v-for="(it, i) in chatMenus" :key="it.id">
          <q-btn
            class="w100p"
            :class="{
              'bg-btn-on': i == menuIdx,
            }"
            flat
            @click="onMenu(i)"
          >
            <div class="w100p ta-l">
              <span>{{ it.title || "New Conversation" }}</span>
            </div>
          </q-btn>
        </div>
      </div>
    </q-scroll-area>
    <div class="bg-setting al-c pa-3">
      <jazz-icon :hash="userInfo.uid" :size="24" />
      <span class="ml-2">{{ userInfo.uname }}</span>
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
    }),
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
    onMenu(i) {
      this.$setStore({
        menuIdx: i,
      });
      this.$bus.emit("chat-focus");
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
    onLogin() {
      this.$router.replace("/login");
    },
    onLogout() {
      this.$setStore({
        loginData: {},
        userInfo: {},
        apiKey: "",
        keyList: [],
        importKey: null,
      });
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