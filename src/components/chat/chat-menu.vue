<template>
  <div class="h-flex h100p">
    <div class="pa-3 al-c">
      <a href="/" class="mr-auto">
        <img
          src="https://dashboard.4everland.org/img/svg/logo-m.svg"
          style="height: 24px"
        />
      </a>
    </div>
    <q-scroll-area
      class="flex-1"
      :thumb-style="{
        right: '2px',
        width: '3px',
        opacity: 0.35,
      }"
    >
      <div v-for="i in 30" :key="i">{{ i }}</div>
    </q-scroll-area>
    <div class="bg-setting al-c pa-3">
      <jazz-icon :hash="userInfo.uid" :size="24" />
      <span class="ml-2">{{ userInfo.uname }}</span>
    </div>
  </div>
</template>

<script>
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
    }),
  },
  created() {
    if (this.token && !this.userInfo.uid) {
      this.getUserInfo();
    }
  },
  methods: {
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