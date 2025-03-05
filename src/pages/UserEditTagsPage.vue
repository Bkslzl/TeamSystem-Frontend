<template>
    <van-form @submit="onSubmit">
      <div class="tag-buttons">
        <van-button
          v-for="tag in tagOptions"
          :key="tag.value"
          :type="selectedTags.includes(tag.value) ? 'primary' : 'default'"
          @click="toggleTag(tag.value)"
        >
          {{ tag.text }}
        </van-button>
      </div>
  
      <div style="margin: 16px">
        <van-button round block type="primary" native-type="submit">
          提交
        </van-button>
      </div>
    </van-form>
  </template>
  
  <script setup lang="ts">
  import { useRoute, useRouter } from "vue-router";
  import { ref, onMounted } from "vue";
  import { Toast } from "vant";
  import { getCurrentUser } from "../services/user";
  import myAxios from "../plugins/myAxios";
  import { TAG_OPTIONS } from "../constants/tags";
  
  const user = ref();
  const route = useRoute();
  const router = useRouter();
  
  const selectedTags = ref<number[]>([]);
  const tagOptions = ref(TAG_OPTIONS);
  
  onMounted(async () => {
    user.value = await getCurrentUser();
  
    if (user.value?.tags) {
      try {
        const tagsArray = JSON.parse(user.value.tags);
        selectedTags.value = tagsArray.map((tag: string) => parseInt(tag, 10));
        console.log("解析后的标签数组:", selectedTags.value);
      } catch (error) {
        console.error("解析 tags 失败:", error);
      }
    }
  });
  
  const toggleTag = (tagId: number) => {
    const index = selectedTags.value.indexOf(tagId);
    if (index > -1) {
      selectedTags.value.splice(index, 1);
    } else {
      selectedTags.value.push(tagId);
    }
  };
  
  const onSubmit = async () => {
    const currentUser = user.value;
    if (!currentUser) {
      Toast.fail("用户未登录");
      return;
    }
  
    const tagsString = JSON.stringify(selectedTags.value);
  
    const res = await myAxios.post("/user/update", {
      id: currentUser.id,
      tags: tagsString,
    });
  
    if (res.code === 0 && res.data > 0) {
      Toast.success("修改成功");
      router.back();
    } else {
      Toast.fail("修改失败");
    }
  };
  </script>
  
  <style scoped>
  .tag-buttons {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin: 10px 0;
  }
  </style>