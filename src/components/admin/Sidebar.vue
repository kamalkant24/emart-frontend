<template>
    <div class="sidebar w-64 h-screen bg-gray-200 p-4 fixed">
      <ul class="space-y-2">
        <li>
          <router-link to="/admin-dashboard" class="block px-4 py-2 text-gray-800 hover:bg-blue-500 hover:text-white rounded">
            Dashboard
          </router-link>
        </li>
  
        <li>
          <button @click="toggleDropdown('category')" class="w-full flex justify-between px-4 py-2 text-gray-800 hover:bg-blue-500 hover:text-white rounded">
            <span>Category Section</span>
            <span :class="{ 'rotate-180': openDropdown === 'category' }">▼</span>
          </button>
          <ul v-show="openDropdown === 'category'" class="pl-4 space-y-1">
            <li><router-link to="/admin-dashboard/category" class="block px-4 py-2 text-gray-700 hover:bg-gray-300 rounded">Categories</router-link></li>
            <li><router-link to="/admin-dashboard/sub-category" class="block px-4 py-2 text-gray-700 hover:bg-gray-300 rounded">Sub Category</router-link></li>
          </ul>
        </li>
  
        <li>
          <button @click="toggleDropdown('product')" class="w-full flex justify-between px-4 py-2 text-gray-800 hover:bg-blue-500 hover:text-white rounded">
            <span>Products Section</span>
            <span :class="{ 'rotate-180': openDropdown === 'product' }">▼</span>
          </button>
          <ul v-show="openDropdown === 'product'" class="pl-4 space-y-1">
            <li><router-link to="/admin-dashboard/add-product" class="block px-4 py-2 text-gray-700 hover:bg-gray-300 rounded">Add Product</router-link></li>
          </ul>
        </li>
  
        <li>
          <button @click="toggleDropdown('deal')" class="w-full flex justify-between px-4 py-2 text-gray-800 hover:bg-blue-500 hover:text-white rounded">
            <span>Deal Section</span>
            <span :class="{ 'rotate-180': openDropdown === 'deal' }">▼</span>
          </button>
          <ul v-show="openDropdown === 'deal'" class="pl-4 space-y-1">
            <li><router-link to="/admin-dashboard/add/deal" class="block px-4 py-2 text-gray-700 hover:bg-gray-300 rounded">Add Deal</router-link></li>
          </ul>
        </li>
  
        <li>
          <button @click="toggleDropdown('coupon')" class="w-full flex justify-between px-4 py-2 text-gray-800 hover:bg-blue-500 hover:text-white rounded">
            <span>Coupon Section</span>
            <span :class="{ 'rotate-180': openDropdown === 'coupon' }">▼</span>
          </button>
          <ul v-show="openDropdown === 'coupon'" class="pl-4 space-y-1">
            <li><router-link to="/admin-dashboard/add/coupon" class="block px-4 py-2 text-gray-700 hover:bg-gray-300 rounded">Add Coupon</router-link></li>
          </ul>
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  import { ref } from "vue";
  
  export default {
    name: "Sidebar",
    setup() {
      const openDropdown = ref(null);
  
      const toggleDropdown = (section) => {
        openDropdown.value = openDropdown.value === section ? null : section;
      };
  
      return { openDropdown, toggleDropdown };
    },
  };
  </script>
  
  <style scoped>
  .rotate-180 {
    transform: rotate(180deg);
  }
  </style>
  