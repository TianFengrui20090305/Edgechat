<script setup>
import { computed, onMounted, reactive, ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import api from '../api.js';
import store from '../store.js';

const route = useRoute();
const router = useRouter();
const loading = ref(false);
const validating = ref(false);
const error = ref('');
const invite = ref(null);

const form = reactive({
  username: '',
  displayName: '',
  password: '',
  confirmPassword: ''
});

const token = computed(() => String(route.params.token || '').trim());

async function loadInvite() {
  validating.value = true;
  error.value = '';
  try {
    const payload = await api.getRegisterInvite(token.value);
    invite.value = payload.invite;
    if (payload.site) {
      store.setSite(payload.site);
    }
  } catch (currentError) {
    error.value = currentError.message;
  } finally {
    validating.value = false;
  }
}

async function submit() {
  if (form.password !== form.confirmPassword) {
    error.value = '两次输入的密码不一致';
    return;
  }

  loading.value = true;
  error.value = '';
  try {
    await api.registerWithInvite(token.value, form);
    router.push({ name: 'login', query: { registered: '1' } });
  } catch (currentError) {
    error.value = currentError.message;
  } finally {
    loading.value = false;
  }
}

onMounted(loadInvite);
</script>

<template>
  <div class="page-shell login-shell">
    <div class="page-card login-card">
      <p class="tag">邀请注册</p>
      <h1 class="title">{{ store.site.siteName }}</h1>
      <p class="subtitle">
        通过管理员生成的一次性注册链接创建自己的账号。
      </p>

      <p v-if="validating" class="muted">正在验证注册链接...</p>
      <p v-else-if="invite?.note" class="muted">邀请说明：{{ invite.note }}</p>
      <p v-if="error" class="error-text">{{ error }}</p>

      <form v-if="invite && !error" @submit.prevent="submit">
        <label class="field">
          <span>用户名</span>
          <input v-model.trim="form.username" autocomplete="username" />
        </label>
        <label class="field">
          <span>显示名称</span>
          <input v-model.trim="form.displayName" autocomplete="nickname" />
        </label>
        <label class="field">
          <span>密码</span>
          <input v-model="form.password" type="password" autocomplete="new-password" />
        </label>
        <label class="field">
          <span>确认密码</span>
          <input v-model="form.confirmPassword" type="password" autocomplete="new-password" />
        </label>

        <button class="button" :disabled="loading" type="submit">
          {{ loading ? '注册中...' : '完成注册' }}
        </button>
      </form>
    </div>
  </div>
</template>
