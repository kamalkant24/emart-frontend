<template>
<div class="shop-page">
    <!-- Hero Section -->
    <section class="shop-hero">
        <div class="shop-hero-content">
            <h1 class="shop-hero-title">Discover the Best Deals</h1>
            <p class="shop-hero-subtitle">Explore a wide range of products at unbeatable prices.</p>
            <router-link to="/categories" class="shop-hero-btn">Browse Categories</router-link>
        </div>
    </section>

    <!-- Filters Section -->
    <section class="filters-section py-4" v-if="products.length  >= 2">
        <div class="container">
            <div class="row">
                <h4>Filter by Price</h4>
                <div class="col-lg-3 price-ranger mt-4">
                    <Slider v-model="priceRange" :min="minPrice" :max="maxPrice" />
                    <!-- <button @click="getSelectedRange">Get Selected Range</button> -->
                </div>
                <div class="col-lg-6 text-center text-lg-end">
                    <button class="btn btn-success" @click="getSelectedRange">Get Selected Range</button>
                </div>
            </div>
        </div>
    </section>
    <div v-if="loading" class="text-center">
        <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">Loading...</span>
        </div>
    </div>
    <!-- Products Grid Section -->
    <section class="products-grid py-5" >
        <div class="container" v-if="products.length > 0">
            <div class="row row-cols-1 row-cols-md-2 row-cols-lg-4 g-4">
                <div class="col" v-for="product in products" :key="product.id">
                    <div class="card product-card shadow-sm">
                        <!-- Product Image -->
                        <img :src="getImageUrl(product.image)" @click="detailProduct(product.slug, product.id)" style="cursor: pointer;" class="card-img-top product-image" alt="Product Image">
                        <div class="card-body product-body">
                            <h5 class="product-name">{{ product.name }}</h5>
                            <!-- <p class="product-description">{{ product.description }}</p> -->
                            <p class="product-description">
                                {{ isExpanded[product.id] ? product.description : product.description.slice(0, 100) + '...' }}
                                <button v-if="product.description.length > 100" class="see-more-btn" @click="toggleDescription(product.id)">
                                    {{ isExpanded[product.id] ? 'See Less' : 'See More' }}
                                </button>
                            </p>

                            <!-- Price Section -->
                            <div class="d-flex justify-content-between align-items-center mt-2">
                                <p class="price">₹{{ parseInt(product.discount_price) }}</p>
                                <p class="original-price text-muted">₹{{ parseInt(product.price) }}</p>
                            </div>

                            <!-- Buttons -->
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
        <div v-else-if="!loading && products.length === 0" class="flex flex-col items-center justify-center py-20">
            <svg class="w-16 h-16 text-gray-400 mb-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h18M9 3v15m6-15v15m-9 4h12a2 2 0 002-2V6H3v12a2 2 0 002 2z" />
            </svg>
            <h3 class="text-2xl font-semibold text-gray-600">No Products Found</h3>
            <p class="text-gray-500 mt-2">Try adjusting your filters or check back later.</p>
        </div>
    </section>
</div>
</template>

<script setup>
import {
    ref,
    onMounted,
    computed,
    reactive
} from 'vue';
import {
    getImageUrl
} from '@/utils/Helper.js';
import apiClient from '@/service/Index';
import showToast from "@/plugins/toast.js";
import {
    useRouter
} from 'vue-router';
import {
    useStore
} from 'vuex';
import Slider from "@vueform/slider";
// export default {
//     name: "ShopPage",
//     components: { Slider },
//     setup() {
const wishlist = reactive(new Set());
const products = ref([]);
const loading = ref(true);
const isExpanded = ref({});
const store = useStore();
const router = useRouter();
const maxPrice = ref(0);
const minPrice = ref(0);
const priceRange = ref([100, 400]);
const userData = computed(() => ({
    // role: store.getters['auth/getRole'],
    token: store.getters['auth/getToken'],
}));
const fetchProducts = async () => {
    try {
        const responseProduct = await apiClient.get('/products');

        if (responseProduct.data ?.success && responseProduct.data.products.length > 0) {
            products.value = responseProduct.data.products;
            minPrice.value = Number(responseProduct.data.minPrice);
            maxPrice.value = Number(responseProduct.data.maxPrice);
            priceRange.value = [minPrice.value, maxPrice.value];
            loading.value = false;
            // showToast("success", "Products found successfully.");
        } else {
            throw new Error('Failed to fetch products');
        }
    } catch (error) {
        if (error.response && error.response.status === 404) {
            console.error('Products not found:');
            // showToast("error", "Products not found.");
        }
        products.value = [];
        loading.value = false;
    }
};
const toggleDescription = (productId) => {
    isExpanded.value[productId] = !isExpanded.value[productId];
};
const handleAddToCart = async (id) => {
    if (!userData.value.token) {
        localStorage.setItem('redirectPath', window.location.pathname);
        // alert('Please log in to add items to the cart.');
        showToast("info", "Please log in to add items to the cart.");
        return router.push({
            name: 'Login'
        });
    }
    try {
        const responseAddToCart = await apiClient.post('/add-to-cart', {
            productId: id
        });
        if (responseAddToCart ?.data) {
            const userItemCount = responseAddToCart.data.count;
            store.dispatch('userCart/userCartItem', userItemCount);
            if (responseAddToCart.data.message == 'Product added to the cart successfully.') {
                showToast("success", responseAddToCart.data.message);
                return router.push({
                    name: 'Cart'
                });
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
const detailProduct = async (slug, id) => {
    try {
        const responseProduct = await apiClient.get(`/product-details/${slug}/${id}`);
        if (responseProduct.data && responseProduct.data.success) {
            const product = responseProduct.data.product;
            router.push({
                name: 'productDeatils',
                params: {
                    productSlug: product.slug
                }
            });
        } else {
            console.error('Product not found or API error:', responseProduct.data.message);
            showToast("error", "Product not found.");
        }
    } catch (error) {
        console.error('Error fetching product:', error);
        showToast("error", "Error fetching product.");
    }
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
        const response = await apiClient.get(`/products-price-range/${minPr}/${maxPr}`);
        if (response.data ?.success && Array.isArray(response.data.products)) {
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
    fetchProducts();
});
//      return {
//         products,
//         getImageUrl,
//         loading,
//         toggleDescription,
//         isExpanded,
//         handleAddToCart,
//         viewDetails,
//         maxPrice,
//         minPrice,
//         priceRange,
//         getSelectedRange
//      }
//     }
// };
</script>

<style scoped>
/* Hero Section */
@import "@vueform/slider/themes/default.css";
@import '@/assets/website/home.css';
@import '@/assets/website/shop.css';
.text-white {
  color: rgb(68, 223, 76) !important;
}

.text-muted {
  color: gray !important;
}


/* hero section css */
.shop-hero {
    position: relative;
    width: 100%;
    height: 30vh; /* Adjusted for a balanced layout */
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    color: white;
    padding: 20px;
    background: linear-gradient(135deg, #ff7e5f, #feb47b, #ffcc33); /* Warm, vibrant gradient */

}

.shop-hero::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5); /* Dark overlay for readability */
}

.shop-hero-content {
    position: relative;
    z-index: 1;
    background: rgba(255, 255, 255, 0.1); /* Light transparent background */
    padding: 30px;
    border-radius: 12px;
    backdrop-filter: blur(5px); /* Glassmorphism effect */
}

.shop-hero-title {
    font-size: 2.8rem;
    font-weight: bold;
    text-transform: uppercase;
    margin-bottom: 12px;
}

.shop-hero-subtitle {
    font-size: 1.4rem;
    margin-bottom: 20px;
}

.shop-hero-btn {
    background: #ffcc33;
    color: white;
    padding: 12px 24px;
    border-radius: 8px;
    text-decoration: none;
    font-size: 1.2rem;
    font-weight: bold;
    transition: 0.3s ease-in-out;
    box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.2);
}

.shop-hero-btn:hover {
    background: #ff9900;
    transform: scale(1.05);
}

/* ✅ Responsive Design */
@media (max-width: 768px) {
    .shop-hero {
        height: 40vh;
        padding: 10px;
    }

    .shop-hero-title {
        font-size: 2.2rem;
    }

    .shop-hero-subtitle {
        font-size: 1.1rem;
    }

    .shop-hero-btn {
        font-size: 1rem;
        padding: 10px 20px;
    }
}

</style>
