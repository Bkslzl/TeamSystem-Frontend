<template>
  <div>
    <user-card-list :user-list="userList" :loading="loading"/>
    <van-empty v-if="!userList || userList.length < 1" description="搜索结果为空" />

    <van-pagination
      v-model="currentPage"
      :total-items="total"
      :items-per-page="pageSize"
      :show-page-size="5"
      @change="onPageChange"
    />
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';
import myAxios from '../plugins/myAxios';
import { Toast } from 'vant';
import qs from 'qs';
import UserCardList from '../components/UserCardList.vue';
import { TAG_OPTIONS } from "../constants/tags";

const route = useRoute();
const { searchType, searchValue } = route.query;

const loading = ref(true);

const userList = ref([]);
const currentPage = ref(1);
const pageSize = ref(10);
const total = ref(0);

const mapTagsToText = (tags: number[]): string[] => {
  return tags.map((tagValue) => {
    const tag = TAG_OPTIONS.find((option) => option.value === tagValue);
    return tag ? tag.text : "未知标签";
  });
};

const loadData = async () => {
  let apiUrl = '';
  let params = {};

  if (searchType === 'name') {
    apiUrl = '/user/search/name';
    console.log('这里是名称结果页面');
    params = {
      username: searchValue,
      pageNum: currentPage.value,
      pageSize: pageSize.value,
    };
  } else if (searchType === 'tags') {
    apiUrl = '/user/search/tags';
    params = {
      tagNameList: searchValue,
      pageNum: currentPage.value,
      pageSize: pageSize.value,
    };
  }

  const userListData = await myAxios
    .get(apiUrl, {
      params,
      paramsSerializer: (params) => {
        return qs.stringify(params, { indices: false });
      },
    })
    .then(function (response) {
      console.log(`${apiUrl} succeed`, response);
      return response?.data;
    })
    .catch(function (error) {
      console.error(`${apiUrl} error`, error);
      Toast.fail('请求失败');
    });

  if (userListData?.records) {
    userListData.records.forEach((user) => {
      if (user.tags) {
        user.tags = mapTagsToText(JSON.parse(user.tags));
      }
    });
    userList.value = userListData.records;
    total.value = userListData.total;
  }
  loading.value = false;
};

const onPageChange = (page) => {
  currentPage.value = page;
  loadData();
};

onMounted(() => {
  loadData();
});
</script>

<style scoped>
.van-pagination {
  margin: 16px 0;
  text-align: center;
}
</style>