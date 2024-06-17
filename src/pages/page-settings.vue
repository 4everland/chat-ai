<template>
  <div>
    <div class="mt-9 pa-4 d-flex flex-center">
      <div class="w100p" style="max-width: 500px">
        <div class="al-c">
          <jazz-icon v-if="userInfo.uid" :hash="userInfo.uid" :size="40" />
          <img v-else src="/img/chat/avatar.svg" width="40" />
          <span class="ml-3 fw-b fz-16">{{ userInfo.uname || "Visitor" }}</span>
        </div>
        <div class="row q-col-gutter-md mt-3">
          <div class="col-6" v-for="it in links" :key="it.label">
            <a
              target="_blank"
              :href="$getHomeUrl(it.href)"
              class="bg-left bdrs-5 pa-3 al-c hover-1"
            >
              <b class="fz-16">{{ it.label }}</b>
              <img class="ml-auto" src="/img/chat/link.svg" width="24" />
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
export default {
  data() {
    return {};
  },
  computed: {
    ...mapState({
      userInfo: (s) => s.userInfo,
      apiKey: (s) => s.apiKey,
    }),
    links() {
      return [
        {
          label: "Deposit Land",
          href: "/billing",
        },
        {
          label: "Activities",
          href: "/ai-rpc/key/autoGen/" + this.apiKey,
        },
        {
          label: "Models",
          href: "/ai-rpc?tab=Models",
        },
        {
          label: "AI RPC",
          href: "https://docs.4everland.org/ai/ai-rpc",
        },
      ];
    },
  },
};
</script>