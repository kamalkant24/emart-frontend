<template>
<AdminLayout>
    <div class="row">
        <div class="col-md-3">
            <div class="shadow-sm p-3 total-sale">
                <h5 class="total-u">Total users</h5>
                <p class="number">{{ totalUsers || 0 }}</p>
            </div>
        </div>
        <div class="col-md-3">
            <div class="shadow-sm p-3 total-sale">
                <h5 class="total-u">Today registered</h5>
                <p class="number">{{ todayUsers || 0 }}</p>
            </div>
        </div>
        <div class="col-md-3">
            <div class="shadow-sm p-3 total-sale">
                <h5  class="total-u">Today Sale</h5>
                <p  class="number">$1234</p>
            </div>
        </div>
        <div class="col-md-3">
            <div class="shadow-sm p-3 total-sale">
                <h5  class="total-u">Total Revenue</h5>
                <p  class="number">$1234</p>
            </div>
        </div>
    </div>

    <div class="row mt-4">
        <div class="col-md-6">
            <div class="shadow-sm p-3">
                <h5>Worldwide Sales</h5>
            </div>
        </div>
        <div class="col-md-6">
            <div class="shadow-sm p-3">
                <h5>Sales & Revenue</h5>
            </div>
        </div>
    </div>

    <div class="row mt-4">
        <div class="col-md-12">
            <div class="shadow-sm p-3">
                <h5>Recent Sales</h5>
                <table class="table">
                    <thead>
                        <tr>
                            <th>Date</th>
                            <th>Invoice</th>
                            <th>Customer</th>
                            <th>Amount</th>
                            <th>Status</th>
                            <th>Action</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td>01 Jan 2045</td>
                            <td>INV-0123</td>
                            <td>Jhon Doe</td>
                            <td>$123</td>
                            <td><span class="badge bg-success">Paid</span></td>
                            <td><button class="btn btn-success btn-sm">Detail</button></td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</AdminLayout>
</template>

  
<script setup>
import AdminLayout from "@/layouts/AdminLayout.vue";
import apiClient from "@/service/Index";
import {ref, onMounted } from "vue";
const totalUsers= ref(0);
const todayUsers = ref(0);
const users = async () => {
    try {
        const response = await apiClient.get('/users');

        if(response?.data){
            totalUsers.value = response.data.totalCount;
            todayUsers.value = response.data?.todayCount;

        }
    } catch (error) {
        console.log('server error',error);
    }
}

onMounted(() => {
    users();
})
</script>
<style scoped>
.total-sale{
    background-color: #4cce5f;
    border-radius: 10px;
    padding:12px!important;
    color: white;
    width: 250px;
}
.total-u,.number {
    font-size: 1.25rem;
    color: white;
}
</style>
