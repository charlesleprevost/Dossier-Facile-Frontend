<template>
  <div class="fr-container">
    <h2 class="fr-h2 text-center fr-mt-7w fr-mb-5w">
      {{ $t("joingroup.title") }}
    </h2>
    <InitPassword @on-init-password="onInitPassword" />
    <ConfirmModal v-if="isLoggedIn" @valid="logout()" @cancel="redirect()">
      <span>{{ $t("joingroup.already-logged") }}</span>
    </ConfirmModal>
  </div>
</template>

<script lang="ts">
import { User } from "df-shared/src/models/User";
import { Component, Vue } from "vue-property-decorator";
import InitPassword from "df-shared/src/Authentification/InitPassword.vue";
import { mapGetters } from "vuex";
import ConfirmModal from "df-shared/src/components/ConfirmModal.vue";

@Component({
  components: {
    InitPassword,
    ConfirmModal
  },
  computed: {
    ...mapGetters({
      isLoggedIn: "isLoggedIn"
    })
  }
})
export default class JoinCouple extends Vue {
  isLoggedIn!: boolean;

  onInitPassword(user: User) {
    user.token = this.$route.params.token;
    this.$store.dispatch("createPasswordGroup", user).then(
      () => {
        this.$toasted.show(this.$i18n.t("joingroup.password-update").toString(), {
          type: "success",
          duration: 7000
        });
        this.$router.push({ name: "TenantName" });
      },
      (error: any) => {
        if (
          error.response.data.message.includes(
            "password recovery token or is expired"
          )
        ) {
          this.$toasted.show(this.$i18n.t("joingroup.token-expired").toString(), {
            type: "error",
            duration: 7000
          });
        } else {
          this.$toasted.show(this.$i18n.t("joingroup.error").toString(), {
            type: "error",
            duration: 7000
          });
        }
      }
    );
  }

  async logout() {
    await this.$store.dispatch("logout", false);
  }

  redirect() {
    this.$router.push({ name: "Account" });
  }
}
</script>

