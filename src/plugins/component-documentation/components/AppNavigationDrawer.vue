<template>
  <VNavigationDrawer
    v-model="appStore.isAppNavDrawerOpen"
    :rail="appStore.isAppRailOpen"
    app
    floating
    clipped
    expand-on-hover
    class="bg-background"
  >
    <v-card
      elevation="0"
      :rounded="false"
      color="background"
    >
      <ComponentNavigation />
    </v-card>
    <!--    <template #append>
      <VList bg-color="background">
        <v-list-item
          prepend-icon="mdi-logout"
          title="Logout"
          :to="{ name: 'login' }"
        />
      </VList>
    </template>-->
  </VNavigationDrawer>
</template>

<script setup lang="ts">
import { useUserStore } from "@/stores/user";
import { useAppStore } from "@/stores/app";
import ComponentNavigation from "./ComponentNavigation.vue";

const userStore = useUserStore();
const appStore = useAppStore();
const drawer = ref(true);
const rail = ref(true);
</script>

<style scoped lang="scss">
//.app-navigation__header {
//  background-color: #fff !important;
//  color: #000 !important;
//}
</style>
