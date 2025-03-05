<template>
  <div class="tag-search-page">
    <van-search
      v-model="username"
      show-action
      placeholder="请输入用户名"
      @search="doSearch"
      @cancel="onCancel"
    />

    <van-divider content-position="left">已选标签</van-divider>
    <van-grid :column-num="4" :gutter="20" style="padding: 0 16px">
      <van-grid-item v-for="tag in activeIds" :key="tag">
        <van-tag closeable size="medium" type="primary" @close="doClose(tag)">
          {{ getTagText(tag) }}
        </van-tag>
      </van-grid-item>
    </van-grid>

    <van-tree-select
      v-model:active-id="activeIds"
      v-model:main-active-index="activeIndex"
      :items="filteredTagList"
    />

    <div style="padding: 16px">
      <van-button block type="primary" @click="doSearch">搜索</van-button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useRouter } from 'vue-router';
import { TAG_OPTIONS } from '../constants/tags';
import myAxios from '../plugins/myAxios';
import qs from 'qs';
import { Toast } from 'vant';

const router = useRouter();
const username = ref('');

const originTagList = [
  {
    text: '请选择标签',
    children: TAG_OPTIONS.map(tag => ({ text: tag.text, id: tag.value })),
  },
];

const filteredTagList = computed(() => {
  return originTagList.map(parentTag => {
    const tempChildren = [...parentTag.children];
    const tempParentTag = { ...parentTag };
    tempParentTag.children = tempChildren.filter(item =>
      item.text.includes(username.value)
    );
    return tempParentTag;
  });
});

const onCancel = () => {
  username.value = '';
};

const activeIds = ref<number[]>([]);
const activeIndex = ref(0);

const doClose = (tag: number) => {
  activeIds.value = activeIds.value.filter(item => item !== tag);
};

const getTagText = (tagValue: number) => {
  const tag = TAG_OPTIONS.find(option => option.value === tagValue);
  return tag ? tag.text : '未知标签';
};

const doSearch = async () => {
  if (username.value) {
    const response = await myAxios.get('/user/search/name', {
      params: {
        username: username.value,
        pageNum: 1,
        pageSize: 10,
      },
      paramsSerializer: (params) => {
        return qs.stringify(params, { indices: false });
      },
    });
    router.push({
      path: '/user/list',
      query: {
        searchType: 'name',
        searchValue: username.value,
      },
    });
  } else if (activeIds.value.length > 0) {
    router.push({
      path: '/user/list',
      query: {
        searchType: 'tags',
        searchValue: activeIds.value.join(','),
      },
    });
  } else {
    Toast.fail('请输入用户名或选择标签');
  }
};
</script>

<style scoped>
.tag-search-page {
  padding: 16px;
}
</style>