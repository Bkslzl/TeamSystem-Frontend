<template>
  <template v-if="user">
    <van-cell title="昵称" is-link to="/user/edit" :value="user.username"  @click="toEdit('username', '昵称', user.username)"/>
    <van-cell title="账号" :value="user.userAccount"/>
    <van-cell title="头像" is-link :value="user.avatarUrl" @click="toEdit('avatarUrl', '头像', user.avatarUrl)" >
      <img style="height: 48px" :src="user.avatarUrl"/>
    </van-cell>
    <van-cell title="性别" is-link :value="user.gender" @click="toEdit('gender', '性别', user.gender)"/>
    <van-cell title="电话" is-link to="/user/edit" :value="user.phone" @click="toEdit('phone', '电话', user.phone)"/>
    <van-cell title="邮箱" is-link to="/user/edit" :value="user.email" @click="toEdit('email', '邮箱', user.email)"/>
    <van-cell title="标签" is-link to="/user/editTags" :value="user.tag" @click="toEditTags('tag', '标签', user.tag)"/>
    <van-cell title="注册时间" :value="formatTime(user.createTime)"/>
    
  </template>
</template>

<script setup lang="ts">
import {useRouter} from "vue-router";
import {onMounted, ref} from "vue";
import myAxios from "../plugins/myAxios";
import {Toast} from "vant";
import {getCurrentUser} from "../services/user";
import dayjs from "dayjs";
const user = ref();

onMounted(async () => {
  user.value = await getCurrentUser();
})

const router = useRouter();

const toEdit = (editKey: string, editName: string, currentValue: string) => {
  router.push({
    path: '/user/edit',
    query: {
      editKey,
      editName,
      currentValue,
    }
  })
}

const toEditTags = (editKey: string, editName: string, currentValue: string) => {
  router.push({
    path: '/user/editTags',
    query: {
      editKey,
      editName,
      currentValue,
    }
  })
}

const formatTime = (time: string | Date) => {
  return dayjs(time).format("YYYY-MM-DD HH:mm"); // 格式化为 xxxx-xx-xx xx:xx
};

</script>

<style scoped>
</style>