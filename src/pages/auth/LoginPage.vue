<template>
  <div class="login-container d-flex justify-content-center align-items-center">
    <div class="login-box d-flex">
      <!-- Left Side: Login Form -->
      <div class="login-form p-4">
        <h3 class="text-primary">OUR SHOP</h3>
        <h4 class="mt-3">Sign In</h4>
        <p>
          Need an Account? 
          <a href="#" class="text-primary">Get Started!</a>
        </p>

        <Form @submit="handleLogin" :validation-schema="schema" v-slot="{ errors }">
          <BaseInput name="email" label="E-mail" type="email" placeholder="Type your account email..." :error="errors.email" />
          <BaseInput name="password" label="Password" type="password" placeholder="Type Password..." :error="errors.password" />

          <div class="d-flex justify-content-between align-items-center mb-3">
            <BaseInput name="rememberMe" label="Remember Me" type="checkbox" />
            <a href="#" class="text-primary">Forgot Password?</a>
          </div>

          <button type="submit" class="btn btn-primary w-100">Sign In</button>
        </Form>
      </div>

      <!-- Right Side: Sign-Up Section -->
      <div class="signup-section d-flex flex-column justify-content-center align-items-center text-center p-4">
        <h4 class="mt-3">New Here?</h4>
        <p>Create an account and start your shopping journey with us!</p>
        <button class="btn btn-outline-primary" @click="redirectToSignUp">Sign Up</button>
      </div>
    </div>
  </div>
</template>

<script>
import apiClient from "@/service/Index";
import { useStore } from "vuex";
import { useRouter } from "vue-router";
import { Form } from "vee-validate";
import showToast from "@/plugins/toast.js";
import BaseInput from "@/components/inputs/BaseInput.vue";
import { loginSchema } from "@/validations/authValidation";

export default {
  name: "LoginPage",
  components: { BaseInput, Form },
  setup() {
    const router = useRouter();
    const store = useStore();
    const handleLogin = async (values) => {
      try {
        const response = await apiClient.post("/login", values);
        // const encodedCredentials = btoa(`${values.email}:${values.password}`);

        // const response = await apiClient.post(
        //   "/login",
        //   {}, // Send empty body if API expects credentials in headers
        //   {
        //     headers: {
        //       Authorization: `Basic ${encodedCredentials}`,
        //       "Content-Type": "application/json",
        //     },
        //   }
        // );
        if (response.data?.token && response.data?.role) {
          showToast("success", "Login successful!");
          store.dispatch("auth/login", {
            token: response.data.token,
            role: response.data.role,
            userId: response.data.userId,
          });
          const role = response.data.role;
          if(role === 'user'){
            await userCartItem();
          }
          if (role === "user") {
            const redirectPath = localStorage.getItem("redirectPath") || "/";
            localStorage.removeItem("redirectPath");
            router.push(redirectPath);
          } else if (role === "admin") {
            router.push("/admin-dashboard");
          }
          // router.push(response.data.role === "admin" ? "/admin-dashboard" : "/");
        } else {
          showToast("error", "Invalid login details");
        }
      } catch (error) {
        if (error.response && error.response.status === 403) {
          showToast("error", error.response.data.message);
        } else {
          showToast("error", "Login failed");
        }
        console.error("Login error:", error);
      }
    };
    const userCartItem = async () => {
      
      try {
        const { data } = await apiClient.get('/count-cart-item');

        if (data?.success) {
          const userItemCount =  data?.count;
          store.dispatch("userCart/userCartItem", userItemCount);
        } else {
          console.warn("User cart is empty.");
        }
      } catch (error) {
        console.error("Service issue:", error);
      }
    };

    const redirectToSignUp = () => {
      router.push("/sign-up");
    };

    return { handleLogin, schema: loginSchema, redirectToSignUp };
  },
};
</script>

<style scoped>
/* Base Styles */
.login-container {
  height: 77vh;
  background: #f8f9fa;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px; /* Ensures proper spacing on smaller screens */
}

/* Login Box */
.login-box {
  background: #fff;
  display: flex;
  width: 800px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

/* Form Section */
.login-form {
  flex: 1;
  padding: 40px;
}

h4 {
  font-size: 1.5rem;
  font-weight: 600;
}

a {
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

.btn-primary {
  background-color: #28a745;
  border-color: #28a745;
  border: none;
  padding: 12px;
  font-weight: 600;
}

.btn-primary:hover {
  background-color: #28a745;
  border-color: #28a745;
}
.btn.btn-primary {
    color: #FFFFFF;
}
/* Right Side: Sign-Up Section */
.signup-section {
  flex: 1;
  background: #eaf2ff;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  text-align: center;
  padding: 20px;
}

.signup-section h4 {
  font-size: 1.5rem;
  font-weight: 600;
}

.signup-section p {
  font-size: 1rem;
  color: #555;
  margin: 10px 0;
}

.btn-outline-primary {
  border: 2px solid #007bff;
  background: transparent;
  color: #007bff;
  padding: 10px 15px;
  font-weight: 600;
  border-radius: 5px;
  transition: 0.3s ease-in-out;
}

.btn-outline-primary:hover {
  background: #007bff;
  color: #fff;
}

/* 📌 Responsive Design Adjustments */
@media (max-width: 992px) {
  .login-box {
    width: 90%;
    flex-direction: column;
  }
  
  .login-form {
    padding: 30px;
  }

  .signup-section {
    padding: 30px;
  }
}

@media (max-width: 768px) {
  .login-container {
    height: auto;
    padding: 40px 20px;
  }

  .login-box {
    width: 100%;
    flex-direction: column;
    text-align: center;
  }

  .login-form {
    padding: 20px;
  }

  .signup-section {
    padding: 20px;
  }

  h4 {
    font-size: 1.25rem;
  }

  .btn-primary {
    padding: 10px;
  }

  .btn-outline-primary {
    padding: 8px;
    font-size: 14px;
  }
}

@media (max-width: 480px) {
  .login-box {
    width: 100%;
    border-radius: 5px;
    box-shadow: none;
  }

  .login-form {
    padding: 15px;
  }

  .signup-section {
    padding: 15px;
  }

  h4 {
    font-size: 1.1rem;
  }

  .btn-primary {
    font-size: 14px;
    padding: 8px;
  }

  .btn-outline-primary {
    font-size: 13px;
    padding: 7px;
  }
}
</style>
