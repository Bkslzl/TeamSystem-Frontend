<template>
  <van-skeleton title avatar :row="3" :loading="props.loading" v-for="user in props.userList" :key="user.id">
    <van-card
        :desc="user.profile"
        :title="`${user.username}`"
        :thumb="user.avatarUrl"
    >
      <template #tags>
        <van-tag plain type="danger" v-for="tag in user.tags" :key="tag" style="margin-right: 8px; margin-top: 8px">
          {{ tag }}
        </van-tag>
      </template>
      <template #footer>
        <van-button size="mini" @click="showEmail(user.email)">联系我</van-button>
      </template>
    </van-card>
  </van-skeleton>

  <van-popup v-model:show="showPopup" position="bottom" :style="{ height: '30%' }">
    <div style="padding: 20px; text-align: center;">
      <h3>联系邮箱</h3>
      <p>{{ currentEmail }}</p>
      <van-button type="primary" @click="showPopup = false">关闭</van-button>
    </div>
  </van-popup>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { UserType } from "../models/user";

interface UserCardListProps {
  loading: boolean;
  userList: UserType[];
}

const props = withDefaults(defineProps<UserCardListProps>(), {
  loading: true,
  userList: [] as UserType[],
});

const showPopup = ref(false);
const currentEmail = ref('');

const showEmail = (email: string) => {
  currentEmail.value = email;
  showPopup.value = true;
};
</script>

<style scoped>
</style>