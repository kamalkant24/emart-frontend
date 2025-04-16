<template>
  <div class="wrapper">
    <Sidebar />
    <div class="main-content-admin">
      <Navbar />
      <div class="container-fluid mt-4">
        <slot></slot> 
      </div>
      <!-- <Footer /> -->
    </div>
  </div>
</template>

<script setup>
import Sidebar from "@/components/admin/Sidebar.vue";
import Navbar from "@/components/admin/Navbar.vue";
// import Footer from "@/components/admin/Footer.vue";
</script>

<style scoped>
.wrapper {
  display: flex;
}
.main-content-admin {
  margin-left: 250px;
  width: 100%;
}
</style>
