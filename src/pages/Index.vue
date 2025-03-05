<template>
  <van-cell center title="显示最匹配的队友">
    <template #right-icon>
      <van-switch v-model="isMatchMode" size="24" />
    </template>
  </van-cell>
  <user-card-list :user-list="userList" :loading="loading"/>
  <van-empty v-if="!userList || userList.length < 1" description="数据为空"/>

  <van-pagination
    v-model="currentPage"
    :total-items="totalPages * 6"
    :items-per-page="6"
    :show-page-size="5"
    @change="loadData"
  />
</template>

<script setup lang="ts">
import { ref, watchEffect } from 'vue';
import myAxios from "../plugins/myAxios";
import { Toast, Pagination } from "vant";
import UserCardList from "../components/UserCardList.vue";
import { UserType } from "../models/user";
import { TAG_OPTIONS } from "../constants/tags";

const isMatchMode = ref<boolean>(false);
const userList = ref([]);
const loading = ref(true);
const currentPage = ref(1);
const totalPages = ref(0);

const mapTagsToText = (tags: number[]): string[] => {
  return tags.map((tagValue) => {
    const tag = TAG_OPTIONS.find((option) => option.value === tagValue);
    return tag ? tag.text : "未知标签";
  });
};

const loadData = async () => {
  let userListData;
  loading.value = true;
  if (isMatchMode.value) {
    const num = 6;
    userListData = await myAxios.get('/user/match', {
      params: {
        num,
      },
    })
      .then(function (response) {
        console.log('/user/match succeed', response);
        return response?.data;
      })
      .catch(function (error) {
        console.error('/user/match error', error);
        Toast.fail('请求失败');
      });
  } else {
    userListData = await myAxios.get('/user/recommend', {
      params: {
        pageSize: 6,
        pageNum: currentPage.value,
      },
    })
      .then(function (response) {
        console.log('/user/recommend succeed', response);
        totalPages.value = Math.ceil(response?.data?.total / 6);
        return response?.data?.records;
      })
      .catch(function (error) {
        console.error('/user/recommend error', error);
        Toast.fail('请求失败');
      });
  }
  if (userListData) {
    userListData.forEach((user: UserType) => {
      if (user.tags) {
        user.tags = mapTagsToText(JSON.parse(user.tags));
      }
    });
    userList.value = userListData;
  }
  loading.value = false;
};

watchEffect(() => {
  loadData();
});
</script>

<style scoped>
</style>