<template>
  <div id="teamPage">
    <van-search v-model="searchText" placeholder="搜索队伍" @search="onSearch" />
    <van-tabs v-model:active="active" @change="onTabChange">
      <van-tab title="公开" name="public" />
      <van-tab title="加密" name="private" />
    </van-tabs>
    <div style="margin-bottom: 16px" />
    <van-button class="add-button" type="primary" icon="plus" @click="toAddTeam" />
    <team-card-list :teamList="teamList" @join-team="joinTeam" />
    <van-empty v-if="teamList?.length < 1" description="数据为空"/>
  </div>
</template>

<script setup lang="ts">
import { useRouter } from "vue-router";
import TeamCardList from "../components/TeamCardList.vue";
import { onMounted, ref } from "vue";
import myAxios from "../plugins/myAxios";
import { Toast } from "vant";

const active = ref('public');
const router = useRouter();
const searchText = ref('');
const teamList = ref([]);

const onTabChange = (name) => {
  listTeam(searchText.value, name === 'public' ? 0 : 2);
};

const toAddTeam = () => {
  router.push({
    path: "/team/add",
  });
};

const listTeam = async (val = '', status = 0) => {
  const res = await myAxios.get("/team/list", {
    params: {
      searchText: val,
      pageNum: 1,
      status,
    },
  });
  if (res?.code === 0) {
    teamList.value = res.data;
  } else {
    Toast.fail('加载队伍失败，请刷新重试');
  }
};

onMounted(() => {
  listTeam();
});

const onSearch = (val) => {
  listTeam(val);
};

const joinTeam = async (teamId: number) => {
  const res = await myAxios.post("/team/join", {
    teamId,
    userId: currentUser.value.id,
  });
  if (res?.code === 0) {
    Toast.success("加入成功");
    listTeam(searchText.value, active.value === 'public' ? 0 : 2);
  } else {
    Toast.fail(res.message || "加入失败");
  }
};
</script>

<style scoped>
#teamPage {
}
</style>