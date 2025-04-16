<template>
  <nav class="bg-white shadow-sm px-4 py-3 flex justify-between items-center">
    <input type="text" class="w-72 p-2 border rounded-lg focus:outline-none focus:ring focus:border-blue-500" placeholder="Search" />

    <div class="flex items-center space-x-6">
      <button class="px-4 py-2 bg-gray-100 rounded-lg hover:bg-gray-200">Message</button>
      <button class="px-4 py-2 bg-gray-100 rounded-lg hover:bg-gray-200">Notification</button>

      <div class="relative">
        <button @click="toggleDropdown" class="px-4 py-2 bg-gray-100 rounded-lg flex items-center space-x-2 hover:bg-gray-200">
          <span>John Doe</span>
          <span :class="{ 'rotate-180': isDropdownOpen }">▼</span>
        </button>
        
        <ul v-show="isDropdownOpen" @click.stop class="absolute right-0 mt-2 w-48 bg-white shadow-lg rounded-lg py-2 border z-50">
          <li><router-link to="/admin-dashboard/profile" class="block px-4 py-2 hover:bg-gray-200">Profile</router-link></li>
          <li><router-link to="/admin-dashboard/change-password" class="block px-4 py-2 hover:bg-gray-200">Change Password</router-link></li>
          <li v-if="token"><a href="#" @click="handleLogout" class="block px-4 py-2 text-red-500 hover:bg-gray-200">Log Out</a></li>
        </ul>
      </div>
    </div>
  </nav>
</template>

<script>
import { ref, computed } from "vue";
import { useStore } from "vuex";
import { useRouter } from "vue-router";
import apiClient from "@/service/Index";
import showToast from "@/plugins/toast";

export default {
  name: "Navbar",
  setup() {
    const store = useStore();
    const router = useRouter();
    const isDropdownOpen = ref(false);

    const toggleDropdown = () => {
      isDropdownOpen.value = !isDropdownOpen.value;
    };

    const handleLogout = async () => {
      try {
        const response = await apiClient.post("/logout");
        if (response.data.status === 200) {
          store.dispatch("auth/logout");
          router.push("/");
          showToast("success", "Logout successfully");
        }
      } catch (error) {
        console.error("Something went wrong", error);
      }
    };

    const token = computed(() => store.getters["auth/getToken"]);

    return {
      isDropdownOpen,
      toggleDropdown,
      handleLogout,
      token,
    };
  },
};
</script>

<style scoped>
.rotate-180 {
  transform: rotate(180deg);
}
</style>
