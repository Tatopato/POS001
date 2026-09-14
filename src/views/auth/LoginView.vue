<script setup>
import { ref } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useAuthStore } from "../../stores/auth";

const username = ref("");
const password = ref("");
const showPassword = ref(false);
const error = ref("");
const loading = ref(false);

const auth = useAuthStore();
const router = useRouter();
const route = useRoute();

async function submit() {
  error.value = "";
  loading.value = true;

  try {
    await auth.login(username.value, password.value);

    // Web POS อนุญาตเฉพาะ ADMIN และ OWNER
    if (auth.role === "ADMIN") {
      router.push("/admin");
      return;
    }

    if (auth.role === "OWNER") {
      router.push(route.query.redirect || "/owner");
      return;
    }

    // EMPLOYEE ไม่สามารถใช้ Web POS
    auth.logout();

    error.value =
      "บัญชีพนักงานไม่สามารถเข้าสู่ Web POS ได้ กรุณาเข้าใน แอปพลิเคชันposแทน";

  } catch (e) {
    error.value =
      e.response?.data?.message ||
      e.message ||
      "Login failed";
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <form
    class="w-full max-w-md rounded-xl bg-white p-8 shadow"
    @submit.prevent="submit"
  >
    <h1 class="text-2xl font-bold">
      POS Login
    </h1>

    <p class="mt-1 mb-6 text-sm text-gray-500">
      Admin / Store Owner
    </p>

    <div
      v-if="error"
      class="mb-4 rounded bg-red-50 p-3 text-red-700"
    >
      {{ error }}
    </div>

    <input
      v-model="username"
      class="mb-3 w-full rounded-lg border px-3 py-2"
      placeholder="Username"
      autocomplete="username"
    />

    <div class="relative mb-5">
      <input
        v-model="password"
        :type="showPassword ? 'text' : 'password'"
        class="w-full rounded-lg border px-3 py-2 pr-10"
        placeholder="Password"
        autocomplete="current-password"
      />
      <button
        type="button"
        class="absolute right-3 top-1/2 -translate-y-1/2 cursor-pointer text-gray-500"
        :aria-label="showPassword ? 'Hide password' : 'Show password'"
        @click="showPassword = !showPassword"
      >
        <svg
          v-if="showPassword"
          class="h-5 w-5"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          aria-hidden="true"
        >
          <path d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12Z" />
          <circle cx="12" cy="12" r="3" />
        </svg>
        <svg
          v-else
          class="h-5 w-5"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          aria-hidden="true"
        >
          <path d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12Z" />
          <circle cx="12" cy="12" r="3" />
          <path d="m3 3 18 18" />
        </svg>
      </button>
    </div>

    <button
      :disabled="loading"
      class="w-full cursor-pointer rounded-lg bg-blue-600 py-2 text-white disabled:cursor-not-allowed disabled:opacity-50"
    >
      {{ loading ? "Signing in..." : "Sign in" }}
    </button>
  </form>
</template>