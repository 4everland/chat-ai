<style lang="scss">
.chat-input {
  textarea {
    max-height: 200px;
    &::-webkit-input-placeholder {
      color: #cbd5e1;
    }
  }
  .send-btn {
    transform: scale(0.86);
  }
  .q-field--outlined .q-field__control {
    padding-right: 6px;
    &::before,
    &::after {
      border: none !important;
    }
  }
  .q-field__append {
    margin-top: auto;
  }
}
</style>

<template>
  <div class="chat-input px-2 py-1 pt-0 d-flex al-end">
    <q-btn class="mr-2 bg-white mb-1" dense flat @click="onClearChat">
      <img src="/img/ic-clear.svg" width="24" class="px-2p" />
    </q-btn>
    <q-input
      ref="input"
      class="flex-1 s-mb-2 bg-left bdrs-8"
      outlined
      dense
      v-model="inputVal"
      autogrow
      placeholder="Chat or prompt"
      @keyup.enter="onEnter"
      @focus="isFoucs = true"
      @blur="isFoucs = false"
    >
      <template #append>
        <q-btn
          class="send-btn"
          round
          :color="trimVal ? 'primary' : 'info'"
          dense
          :disable="!trimVal"
          @click="onEnter"
        >
          <img
            src="/img/send.svg"
            width="20"
            class="pos-r"
            style="left: -1px"
          />
        </q-btn>
      </template>
    </q-input>
  </div>
</template>

<script>
import { mapState } from "vuex";

export default {
  data() {
    return {
      isFoucs: false,
      inputVal: "",
      composing: false,
    };
  },
  computed: {
    ...mapState({
      apiKey: (s) => s.apiKey,
      chatLogs: (s) => s.chatLogs,
      chatMenus: (s) => s.chatMenus,
      menuIdx: (s) => s.menuIdx,
    }),
    trimVal() {
      return this.inputVal.trim();
    },
  },
  watch: {
    apiKey() {
      if (this.needSend) {
        this.onEnter();
        this.needSend = false;
      }
    },
    menuIdx() {
      this.inputVal = "";
    },
  },
  mounted() {
    const input = this.$refs.input.nativeEl;
    input.addEventListener("compositionstart", () => {
      this.composing = true;
    });
    input.addEventListener("compositionend", () => {
      setTimeout(() => {
        this.composing = false;
      }, 100);
    });
    this.$bus.on("chat-focus", () => {
      this.$refs.input?.focus();
    });
  },
  methods: {
    async onClearChat() {
      // This will clean up your chat history. Unless you have exported the chat, this action is irreversible. Would you like to proceed?
      const len = this.chatLogs.length;
      this.$setStore({
        chatLogs: [],
      });
      this.$store.commit("updateChatMenu", {
        title: "",
      });
      // if (!len && this.chatMenus.length > 1) {
      //   await this.$sleep(100);
      // }
    },
    onEnter(e) {
      if (e.shiftKey) {
        return;
      }
      if (this.composing) return;
      if (!this.trimVal) {
        this.inputVal = "";
        return;
      }

      // console.log(this.trimVal);
      this.sendMsg();
    },
    sendMsg() {
      this.$bus.emit("send-msg", this.trimVal);
      this.inputVal = "";
    },
  },
};
</script>