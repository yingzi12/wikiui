storyChapterEdit.vue<template>
  <!--        标题-->
  <div>
    <el-menu
        :default-active="1"
        mode="horizontal"
        style="margin:0px;pardding:0px"
    >
      <el-menu-item index="1">{{sname}} -{{form.pname}}-章节目录</el-menu-item>
    </el-menu>
  </div>
  <!--        表格-->
  <div>
    <h1>所属分卷 --  {{form.pname}}</h1>
    <el-form
        ref="ruleFormRef"
        :model="form"
        :rules="rules"
        label-width="120px">
      <el-form-item label="章节名称" prop="title">
        <el-input v-model="form.title" />
      </el-form-item>

      <div>
        <h3>章节内容</h3>
        <ckeditor    :editorDisabled="true"  :editor="editor" v-model="form.contentZip" :config="editorConfig"></ckeditor>

      </div>
      <div>
        <el-button @click="handleReturn">返回</el-button>
        <el-button type="primary" @click="handleReelAdd(ruleFormRef)">确定</el-button>
      </div>
    </el-form>
  </div>
</template>

<script lang="ts" setup>
import { inject, reactive, ref, toRefs} from 'vue'
import { updateDraftChapter,getDraftChapter } from "@/api/admin/draftChapter";
import  Editor  from 'ckeditor5-custom-build-free-all/build/ckeditor';
const editor = Editor
const baseUrl = inject("$baseUrl")
const uploadImgUrl = ref(baseUrl + "/common/uploadImage"); // 上传的图片服务器地址
const editorConfig ={
  language:"zh-cn",
  simpleUpload: {
    // The URL the images are uploaded to.
    uploadUrl: uploadImgUrl.value,
  },
}
import {useRoute, useRouter} from "vue-router";
import {ElMessage, FormInstance} from "element-plus";

// 接收url里的参数
const route = useRoute();
const router = useRouter()

const sid = ref(route.query.sid);
const dscid = ref(null);
dscid.value = route.query.dscid;
const scname = ref('');
scname.value = <string>route.query.scname;

const sname = ref(route.query.sname);
//console.log("世界id="+wid.value);


class Story {
  id: number
  name: string
  types: string
  intro: string
  createTime:string
}

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,

    title: undefined,
    sid:sid.value,
    level:1
  },
  rules: {
    title: [{ required: true, message: "分类/目录名称不能为空", trigger: "blur" }, { min: 2, max: 20, message: "用户名称长度必须介于 2 和 20 之间", trigger: "blur" }],
  }
});
const ruleFormRef = ref<FormInstance>()
const { queryParams, form, rules } = toRefs(data);

const handleReelAdd = async (formEl: FormInstance | undefined) => {
  if (!formEl) return
  if (dscid.value == undefined || dscid.value == null || dscid.value == '') {
    ElMessage.error("非法操作")
    return
  }
  form.value.sid=sid.value
  await formEl.validate((valid, fields) => {
    if (valid) {
      updateDraftChapter(form.value).then(response => {
        router.push("/admin/draftChapter?sid="+sid.value+"&sname="+sname.value);
      })
    } else {
      //console.log('error submit!', fields)
    }
  })
}
function handleInfo(){
  getDraftChapter(sid.value,dscid.value).then(response => {
    form.value=response.data
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
