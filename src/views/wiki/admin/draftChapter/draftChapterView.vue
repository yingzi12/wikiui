storyChapterEdit.vue<template>
  <!--        标题-->
  <div>
    <el-menu
        :default-active="1"
        mode="horizontal"
        style="margin:0px;pardding:0px"
    >
      <el-menu-item index="1">{{draftChapter.sname}} -{{draftChapter.pname}}-章节目录</el-menu-item>
    </el-menu>
  </div>
  <!--        表格-->
  <div>
    <h1>所属分卷 --  {{draftChapter.pname}}</h1>

      <div>
        <h3>{{ draftChapter.title }}</h3>
         <div v-html="draftChapter.contentZip"></div>

      </div>
      <div>
        <el-button @click="handleReturn">返回</el-button>
      </div>
  </div>
</template>

<script lang="ts" setup>
import { inject, reactive, ref, toRefs} from 'vue'
import { getDraftChapter } from "@/api/admin/draftChapter";


import {useRoute, useRouter} from "vue-router";
import {ElMessage, FormInstance} from "element-plus";

// 接收url里的参数
const route = useRoute();
const router = useRouter()

const sid = ref(route.query.sid);
const dscid = ref(null);
dscid.value = route.query.dscid;
const scname = ref(route.query.scname);
scname.value = <string>route.query.scname;
const sname = ref(route.query.sname);

const draftChapter=ref({})

function handleInfo(){
  getDraftChapter(sid.value,dscid.value).then(response => {
      draftChapter.value=response.data
  });
}
function handleReturn(){
  router.back()
}
handleInfo()
</script>

<style scoped>


.center {
  display: flex;
  justify-content: center;
  align-items: center;
}


</style>
