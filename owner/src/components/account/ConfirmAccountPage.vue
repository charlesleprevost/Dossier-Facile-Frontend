<script setup lang="ts">
import { useI18n } from 'vue-i18n';
import { useRoute, useRouter } from 'vue-router';
import { ref } from 'vue';
import AuthService from '../auth/AuthService';

const route = useRoute();
const router = useRouter();
const { t } = useI18n();

const token = route.params.token.toString();
const showError = ref(false);
const errorMessage = ref('');

AuthService.confirmAccount(token).then(
  () => {
    router.push({ name: 'Dashboard' });
  },
  (error: any) => {
    errorMessage.value = error;
    showError.value = true;
  },
);
</script>

<template>
  <div class="fr-container">
    <div v-if="!showError">{{ t('loading') }}</div>
    <div v-if="showError">{{ t('error-occured') }}</div>
  </div>
</template>

<i18n>
{
  "en": {
    "loading": "Loading",
    "error-occured": "An error occured"
  },
  "fr": {
    "loading": "Veuillez patienter",
    "error-occured": "Une erreur est survenue"
  }
}

</i18n>
