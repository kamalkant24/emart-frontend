<template>
  <div id="app">
    <HeaderSection v-if="role !== 'admin'" />
    <SidebarAdmin v-if="role === 'admin'" />
    <div :class="{ 'main-content': role !== 'admin' }">
      <router-view></router-view>
    </div>
    <FooterSection v-if="role !== 'admin'" />
  </div>
</template>

<script>
import { computed } from 'vue';
import { useStore } from 'vuex';
import FooterSection from './components/website/Footer.vue';
import HeaderSection from './components/website/Header.vue';
import SidebarAdmin from '@/components/admin/Sidebar.vue';

export default {
  name: 'App',
  components: {
    HeaderSection,
    FooterSection,
    SidebarAdmin,
  },
  setup() {
    const store = useStore();
    const role = computed(() => store.getters['auth/getRole']);
    return {
      role,
    };
  },
};
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}
#app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.main-content {
  flex-grow: 1;
  padding-top: 70px;
  
}

footer {
  background-color: #333;
  color: white;
  padding: 20px;
  text-align: center;
}
</style>
