<script setup>
import { computed, onMounted, provide, watch } from 'vue';
import { useRoute } from 'vue-router';
import { useModal } from '@/composables/useModal';
import { useI18n } from '@/composables/useI18n';
import { useDomain } from '@/composables/useDomain';
import { useUserSkin } from '@/composables/useUserSkin';
import { useApp } from '@/composables/useApp';
import { useWeb3 } from '@/composables/useWeb3';
import { useNotifications } from '@/composables/useNotifications';
import aliases from '@/../snapshot-spaces/spaces/aliases.json';

const { domain } = useDomain();
const { loadLocale } = useI18n();
const route = useRoute();
const { modalOpen } = useModal();
const { userSkin } = useUserSkin();
const { init, explore, app } = useApp();
const { web3 } = useWeb3();
const { notify } = useNotifications();

provide('web3', web3);
provide('notify', notify);

const space = computed(() => {
  const key = aliases[domain] || domain || route.params.key;
  return explore.value.spaces?.[key];
});

const skin = computed(() => {
  if (domain && space.value?.skin) {
    let skinClass = space.value.skin;
    if (userSkin.value === 'dark-mode')
      skinClass += ` ${space.value.skin}-dark-mode`;
    return skinClass;
  }
  return userSkin.value;
});

onMounted(async () => {
  await loadLocale();
  init();
});

watch(modalOpen, val => {
  const el = document.body;
  el.classList[val ? 'add' : 'remove']('overflow-hidden');
});
</script>

<template>
  <div :class="skin" id="app" class="overflow-hidden pb-4 font-serif text-base">
    <UiLoading v-if="app.loading || !app.init" class="overlay big" />
    <div v-else>
      <Scroller />
      <div :class="{ 'sm:ml-[68px]': !domain }">
        <Topnav />
        <router-view :key="$route.path" class="flex-auto" />
      </div>
    </div>
    <div id="modal" />
    <Notifications />
  </div>
</template>
