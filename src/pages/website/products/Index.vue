<template>
<div class="shop-page">
    <!-- Hero Section -->
    <section class="relative bg-cover bg-center min-h-[250px] md:min-h-[250px] lg:min-h-[250px] flex items-center justify-center text-center text-white px-4">
        <div class="absolute inset-0 bg-black bg-opacity-50"></div>

        <div class="relative z-10 w-full max-w-4xl">
        <h1 class="text-3xl sm:text-4xl md:text-5xl lg:text-6xl font-bold mb-4">
            Explore Our Categories
        </h1>
        <p class="text-base sm:text-lg md:text-xl mb-6 max-w-2xl mx-auto">
            Discover a wide variety of categories to find exactly what you need.
        </p>
        <router-link to="/shop"
            class="bg-indigo-600 hover:bg-indigo-700 text-white px-5 sm:px-6 py-2 sm:py-3 rounded-lg shadow-lg transition duration-300 text-sm sm:text-base">
            Explore More
        </router-link>
        </div>
    </section>
    <!-- Filters Section -->
    <section class="filters-section py-4">
        <div class="container">
            <div class="row">
                <!-- <div class="col-lg-3">
                    <h4>Filter by Categories</h4>
                    <select class="form-select" v-model="selectedCategory">
                        <option value="">All Categories</option>
                        <option v-for="category in categories" :key="category.id" :value="category.id">
                            {{ category.name }}
                        </option>
                    </select>
                </div> -->
                <h4>Filter by Price</h4>
                <div class="col-lg-3 price-ranger">
                    <Slider v-model="priceRange" :min="minPrice" :max="maxPrice" />
                     <!-- <button @click="getSelectedRange">Get Selected Range</button> -->
                </div>
                <div class="col-lg-6 text-center text-lg-end">
                    <button class="btn btn-success" @click="getSelectedRange">Get Selected Range</button>
                </div>
            </div>
        </div>
    </section>
    <div class="loader">
        <div v-if="loading" class="text-center">
            <div class="spinner-border text-primary" role="status">
                <span class="visually-hidden">Loading...</span>
            </div>
        </div>
    </div>
    <section class="products-grid py-5" v-if="products?.length > 0">
        <div class="container">
            <div class="row row-cols-1 row-cols-md-2 row-cols-lg-4 g-4">
            <div class="col" v-for="product in products" :key="product.id">
                <div class="card product-card shadow-sm">
                <!-- <img :src="getImageUrl(product.image)" class="card-img-top product-image" alt="Product Image"> -->
                <img :src="getImageUrl(product.image)" @click="detailProduct(product.slug, product.id)" style="cursor: pointer;" class="card-img-top product-image" alt="Product Image">
                <div class="card-body product-body">
                    <h5 class="product-name">{{ product.name }}</h5>
                    <p class="product-description">
                      {{ isExpanded[product.id] ? product.description : product.description.slice(0, 100) + '...' }}
                      <button v-if="product.description.length > 100" class="see-more-btn" @click="toggleDescription(product.id)">
                        {{ isExpanded[product.id] ? 'See Less' : 'See More' }}
                      </button>
                    </p>                    
                    <div class="d-flex justify-content-between align-items-center mt-2">
                    <p class="price">₹{{ parseInt(product.discount_price) }}</p>
                    <p class="original-price text-muted">₹{{ parseInt(product.price) }}</p>
                    </div>
                    <div class="button-group d-flex justify-content-between mt-3">
                    <button class="btn btn-primarys btn-custom" @click="handleAddToCart(product.id)">
                        <i class="fas fa-shopping-cart"></i> Add to Cart
                    </button>
                    <!-- <button class="btn btn-outline-secondary btn-custom" @click="viewDetails(product.slug, product.id)">
                        <i class="fas fa-info-circle"></i> Details
                    </button> -->
                    <button class="btn btn-outline-secondary btn-custom" @click="addToWishlist(product.id)">
                        <i class="fas fa-heart" :class="{ 'text-white': wishlist.has(product.id), 'text-muted': !wishlist.has(product.id) }"></i>
                        Wishlist
                    </button>
                    </div>
                </div>
                </div>
            </div>
            </div>
        </div>
    </section>

<div class="container text-center text-danger" v-else>
  <h4 class="not-found">Product not found</h4>
</div>

</div>
</template>

<script setup>
import {ref,onMounted,computed,reactive } from 'vue';
import {useRoute,useRouter } from 'vue-router'
import apiClient from '@/service/Index';
import {getImageUrl} from '@/utils/Helper';
import { useStore } from 'vuex';
import Slider from "@vueform/slider";
import showToast from "@/plugins/toast.js";
const wishlist = reactive(new Set());
        const route = useRoute();
        const router = useRouter();
        const store = useStore();
        const id = route.params.subCatId;
        const products = ref([]);
        const loading = ref(true);
        const isExpanded = ref({});
        const maxPrice = ref(0);
        const minPrice = ref(0);
        const priceRange = ref([minPrice.value, maxPrice.value]);
        const userData =computed(() => ({
            role: store.getters['auth/getRole'],
            token:  store.getters['auth/getToken'],
        }));
        const fetchProducts = async () => {
            try {
                const response = await apiClient.get(`fetch-product/${id}`);
                if (response.data && response.data.success) {
                    products.value = response.data.products;
                    minPrice.value = Number(response.data.minPrice);
                    maxPrice.value = Number(response.data.maxPrice);
                    priceRange.value = [minPrice.value, maxPrice.value];
                    loading.value = false;
                }else {
                    products.value = [];
                    loading.value = false;
                }
            } catch (error) {
                console.log('product not fetch', error);
                 products.value = [];
                 loading.value = true;
            }
        }
        const handleAddToCart = async (id) => {
            if (!userData.value.token) {
                localStorage.setItem('redirectPath', window.location.pathname);
                alert('Please log in to add items to the cart.');
                showToast("info", "Please log in to add items to the cart.");
                return router.push({ name: 'Login' });
            }
            try {
                const responseAddToCart = await apiClient.post('/add-to-cart', { productId: id });
                if (responseAddToCart?.data) {
                    const userItemCount = responseAddToCart.data.count;
                    store.dispatch('userCart/userCartItem',userItemCount);
                    if(responseAddToCart.data.message == 'Product added to the cart successfully.'){
                        showToast("success", responseAddToCart.data.message);
                         return router.push({ name: 'Cart' });
                    }
                    if (responseAddToCart.data.message == 'Product is already in the cart.') {
                        showToast("info", responseAddToCart.data.message);
                    }
                }
            } catch (error) {
              showToast("error", "Failed to add item to cart. Please try again.");
              console.error('Error adding to cart:', error);
              // alert('Failed to add item to cart. Please try again.');
            }
        };
       const detailProduct = async (slug,id) => {
            try {
                const responseProduct = await apiClient.get(`/product-details/${slug}/${id}`);
                if (responseProduct.data && responseProduct.data.success) {
                    const product = responseProduct.data.product;
                    router.push({ name: 'productDeatils', params: { productSlug: product.slug } });
                } else {
                    console.error('Product not found or API error:', responseProduct.data.message);
                    alert('Product not found.');
                }
            } catch (error) {
                console.error('Error fetching product:', error);
                alert('An error occurred while fetching the product details.');
            }
        };
        const toggleDescription = (productId) => {
          isExpanded.value[productId] = !isExpanded.value[productId];
        };

        const getSelectedRange = async () => {
            if (!Array.isArray(priceRange.value) || priceRange.value.length !== 2) {
                console.error("Invalid price range selected.");
                return;
            }
            const [minPr, maxPr] = priceRange.value;
            if (isNaN(minPr) || isNaN(maxPr) || minPr > maxPr) {
                console.error("Invalid price range values.");
                return;
            }
            try {
                // const response = await apiClient.get(`/product-subcategory--range/${id}/${minPr}/${maxPr}`);
                const response = await apiClient.get(`/products-subcategory-price-range/${id}/${minPr}/${maxPr}`);
                if (response.data?.success && Array.isArray(response.data.products)) {
                    products.value = response.data.products;
                } else {
                    console.warn("No products found in this price range.");
                    products.value = [];
                }
            } catch (error) {
                console.error("Server error:", error.message || error);
                products.value = [];
            }
        };
        const addToWishlist = async (id) => {
            if (!userData.value.token) {
                localStorage.setItem('redirectPath', window.location.pathname);
                showToast("info", "Please log in to add items to the wishlist.");
                return router.push({
                    name: 'Login'
                });
            }
            try {
                const wishlistResponse = await apiClient.post('/user/add/wishlist', {
                    id: id
                });
                if (wishlistResponse ?.data.success) {
                    wishlist.add(id);
                    showToast("success", "Product added to wishlist successfully.");
                } else {
                    showToast("error", wishlistResponse ?.data.message || "Something went wrong while adding to the wishlist.");
                }
            } catch (error) {
                if (error.response) {
                    const {
                        status,
                        data
                    } = error.response;
                    if (status === 409) {
                        showToast("warning", data.message || "This product is already in your wishlist.");
                    } else if (status === 422) {
                        showToast("error", "Invalid data. Please try again.");
                        console.error("Validation errors:", data.errors);
                    } else {
                        showToast("error", data.message || "An unexpected error occurred. Please try again later.");
                    }
                } else {
                    showToast("error", "Unable to connect to the server. Please check your internet connection.");
                }
                console.error('Error while adding to wishlist:', error);
            }
        };

        onMounted(() => {
            fetchProducts()
        });
</script>

<style scoped>
@import '@/assets/website/home.css';
@import "@vueform/slider/themes/default.css";
/* Product Image */
.text-white {
  color: rgb(68, 223, 76) !important;
}

.text-muted {
  color: gray !important;
}
</style>
