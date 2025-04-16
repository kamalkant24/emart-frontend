<template>
<div class="deal-page">
    <!-- Hero Section -->
    <section class="hero-section text-center py-5">
        <div class="container">
            <h1 class="display-4 fw-bold text-white">Exclusive Deals & Discounts</h1>
            <p class="lead text-white mb-4">Shop now to grab the best offers and limited-time deals on top products!</p>
            <router-link to="/shop" class="btn btn-deal btn-lg">Shop Now</router-link>
            <!-- <button class="btn btn-deal btn-lg">Shop Now</button> -->
        </div>
    </section>
    <div class="loader">
        <div v-if="loading" class="text-center">
            <div class="spinner-border text-primary" role="status">
                <span class="visually-hidden">Loading...</span>
            </div>
        </div>
    </div>
    <!-- Deals Grid Section -->
    <section class="deals-grid py-5" v-if="deals.length > 0">
        <div class="container">
            <div v-for="deal in deals" :key="deal.id">
                <h2 class="text-center mb-3 mt-5">{{ deal.name }}</h2>
                <p class="text-center text-danger" v-if="deal.end_date">
                    <strong>End of Sale:</strong> {{ countdownMessage(deal.end_date) }}
                </p>
                <p class="text-center">{{ deal.description }}</p>
                <div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
                    <div class="col" v-for="product in deal.products" :key="product.id">
                        <div class="card deal-card shadow-lg">
                            <img :src="getImageUrl(product.image)" class="card-img-top" alt="Deal Image" />
                            <span v-if="deal.discount_type === 'percentage'" class="badge bg-success position-absolute top-0 end-0 m-3">
                                {{ calculateDiscountPercentage(product.price, deal) }}% Off
                            </span>
                            <span v-if="deal.discount_type === 'fixed'" class="badge bg-success position-absolute top-0 end-0 m-3">
                                ₹{{ Number(deal.discount_value).toFixed(0) }} Off
                            </span>
                            <div class="card-body">
                                <h5 class="card-title"></h5>
                                <p class="card-text">{{ product.description }}</p>

                                <div class="d-flex justify-content-between align-items-center">
                                    <p class="price text-success fw-bold">₹{{ calculateDiscountedPrice(product.price, deal) }}</p>
                                    <p class="price text-danger" style="text-decoration: line-through;">₹{{ parseInt(product.price) }}</p>
                                    <button class="btn btn-primarys btn-sm" @click="handleAddToCart(product.id)">Add to Cart</button>
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
        </div>
    </section>
    <div class="container text-center mt-5" v-else-if="!loading">
        <div class="row">
            <div class="col-12 col-md-8 offset-md-2">
                <div class="card p-4 shadow-lg border-0 rounded">
                    <h4 class="display-4 text-danger mb-3">
                        Oops! No Deals Found
                    </h4>
                    <p class="lead mb-4">
                        It looks like the deals you're searching for aren't available right now. But don't worry, our team is always working on bringing you the best offers. Stay tuned!
                    </p>
                    <div class="cta-buttons">
                        <a href="/" class="btn btn-primary btn-lg mb-3 mb-md-0 mr-md-3">Go Back Home</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
</template>

<script setup>
import apiClient from '@/service/Index';
import {
    onMounted,
    ref,
    computed,
    reactive
} from 'vue';
import {calculateDiscountPercentage} from '@/utils/Helper';
import {getImageUrl} from '@/utils/Helper.js';
import {calculateDiscountedPrice} from '@/utils/Helper';
import {useRouter} from 'vue-router';
import {useStore} from 'vuex';
import showToast from "@/plugins/toast.js";
const wishlist = reactive(new Set());
const deals = ref([]);
const currentTime = ref(new Date());
const loading = ref(true);
const store = useStore();
const router = useRouter();
const userData = computed(() => ({
    token: store.getters['auth/getToken'],
}));
const fetchdeals = async () => {
    try {
        const response = await apiClient.get('/current-deals');
        if (response.data ?.success) {
            deals.value = response.data.deals;
            loading.value = false;
        }
    } catch (error) {
        console.log('deals not fetch', error);
        loading.value = true;
    }
};

const updateCurrentTime = () => {
    currentTime.value = new Date();
};
// const countdownMessage = (deal) => {
//     const dealEndDate = new Date(deal);
//     const timeLeft = dealEndDate - currentTime.value;
//     if (timeLeft > 0) {
//         const hours = Math.floor((timeLeft / (1000 * 60 * 60)) % 24);
//         const minutes = Math.floor((timeLeft / (1000 * 60)) % 60);
//         const seconds = Math.floor((timeLeft / 1000) % 60);
//         return `${hours} hours ${minutes} minutes ${seconds} seconds`;
//     } else {
//         return 'Sale Ended!';
//     }
// };

const countdownMessage = (deal) => {
    const dealEndDate = new Date(deal);
    const timeLeft = dealEndDate - currentTime.value;

    if (timeLeft > 0) {
        const days = Math.floor(timeLeft / (1000 * 60 * 60 * 24));
        const hours = Math.floor((timeLeft / (1000 * 60 * 60)) % 24);
        const minutes = Math.floor((timeLeft / (1000 * 60)) % 60);
        const seconds = Math.floor((timeLeft / 1000) % 60);

        return `${days > 0 ? `${days} day${days > 1 ? 's' : ''} ` : ''}${hours} hour${hours !== 1 ? 's' : ''} ${minutes} minute${minutes !== 1 ? 's' : ''} ${seconds} second${seconds !== 1 ? 's' : ''}`;
    } else {
        return 'Sale Ended!';
    }
};

const handleAddToCart = async (id) => {
    if (!userData.value.token) {
        localStorage.setItem('redirectPath', window.location.pathname);
        alert('Please log in to add items to the cart.');
        return router.push({
            name: 'Login'
        });
    }
    try {
        const responseAddToCart = await apiClient.post('/add-to-cart', {
            productId: id
        });
        if (responseAddToCart ?.data) {
            store.dispatch('userCart/userCartItem', responseAddToCart.data.count);
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
        console.error('Error adding to cart:', error);
        showToast("error", "Failed to add item to cart. Please try again.");
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
    fetchdeals();
    setInterval(updateCurrentTime, 1000);
});

</script>
<style scoped>
/* Hero Section */
.hero-section {
    background: linear-gradient(135deg, #ff7e5f, #feb47b, #ffcc33); /* Warm, vibrant gradient */

    /* background-color: #007bff; */
    color: white;
    padding-top: 80px;
    padding-bottom: 80px;

}

.hero-section h1 {
    font-size: 3rem;
    font-weight: 700;
}

.hero-section p {
    font-size: 1.2rem;
}

.card.p-4.shadow-lg.border-0.rounded {
    margin-bottom: 45px;
}

.hero-section .btn-deal {
    background: #ffcc33;
    /* background-color: #ff5733; */
    color: white;
    padding: 0.75rem 1.5rem;
    font-weight: 600;
    border-radius: 25px;
    transition: background-color 0.3s, border-color 0.3s;
}

.hero-section .btn-deal:hover {
    background-color: #ff9900;
}

/* Deals Grid Section */
.deals-grid {
    background-color: #f8f9fa;
}

.card.deal-card {
    border: none;
    border-radius: 15px;
    transition: transform 0.3s ease, box-shadow 0.3s ease;
    display: flex;
    flex-direction: column;
    /* Ensures that the card layout adjusts well for different content */
    height: 100%;
    /* Ensure cards have equal height */
}

.card.deal-card:hover {
    transform: translateY(-10px);
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
}

.card-img-top {
    border-radius: 15px 15px 0 0;
    object-fit: cover;
    height: 200px;
}

.card-body {
    padding: 1.5rem;
    background-color: #fff;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    /* Ensures card content is evenly spaced */
}

.card-title {
    font-size: 1.5rem;
    font-weight: 600;
    color: #333;
    margin-bottom: 1rem;
    overflow: hidden;
    /* Prevents overflow if text is too long */
    text-overflow: ellipsis;
    /* Adds ellipsis for long text */
    white-space: nowrap;
    /* Prevents wrapping */
}

.card-text {
    color: #777;
    font-size: 1rem;
    margin-bottom: 1rem;
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 10;
    /* Limits description to 3 lines */
    -webkit-box-orient: vertical;
}

.price {
    font-size: 1.25rem;
    font-weight: 700;
    color: #28a745;
}

.btn-primarys {
    background-color: #28a745;
    /* Green button color */
    border-color: #28a745;
    padding: 0.75rem 1.5rem;
    font-weight: 600;
    border-radius: 35px;
    transition: background-color 0.3s;
    margin-top: auto;
    /* Pushes the button to the bottom */
}

.btn.btn-primarys {
    color: #FFFFFF;
}

.btn-primarys:hover {
    background-color: #218838;
    /* Darker green shade on hover */
    border-color: #1e7e34;
}

.deals-grid h2 {
    font-size: 2.5rem;
    font-weight: 700;
    color: #444;
    margin-bottom: 40px;
    text-align: center;
    /* Centers the heading */
}

/* Responsive Design */
@media (max-width: 768px) {
    .hero-section h1 {
        font-size: 2.5rem;
    }

    .hero-section p {
        font-size: 1rem;
    }

    .card-title {
        font-size: 1.2rem;
        white-space: normal;
        /* Allow text to wrap */
    }

    .card-text {
        font-size: 0.9rem;
    }

    .price {
        font-size: 1.1rem;
    }

    .btn-primary {
        padding: 0.5rem 1rem;
    }
}

/* deal section */
.card.p-4.shadow-lg.border-0.rounded[data-v-364a363c] {
    margin-bottom: 45px;
    padding-bottom: 107px !important;
}

.py-5 {
    padding-top: 3rem !important;
    padding-bottom: 2rem !important;
}
.text-white {
  color: rgb(45, 48, 45) !important;
}

.text-muted {
  color: gray !important;
}
.btn-outline-secondary {
  color: #6c757d;
  border-color: #6c757d;
}
.btn-outline-secondary:hover {
  background-color: #6c757d;
  color: #fff;
}
.btn-primarys, .btn-outline-secondary {
  border-radius: 35px;
}
</style>
