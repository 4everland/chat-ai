<template>
  <div class="pa-6"></div>
  <div class="al-c f-center">
    <q-card class="full-width m-auto" style="max-width: 600px">
      <q-card-section class="ta-c py-9">
        <q-btn color="primary" rounded style="width: 220px" @click="onLogin"
          >Sign In</q-btn
        >
        <div class="py-6">OR</div>
        <q-btn
          color="primary"
          outline
          rounded
          style="width: 220px"
          @click="$bus.emit('show-import')"
          >Import A 4EVER AI Key</q-btn
        >
        <div class="mt-8 fz-15">To start your Al Journey!</div>
      </q-card-section>
    </q-card>
  </div>
</template>

<script>
export default {
  data() {
    return {};
  },
  methods: {
    onLogin() {
      this.$router.replace("/login");
    },
  },
};
</script>