<template>
    <StoryHead :head-type="3" second-type="4" :sid="sid" :sname="sname" :reel-name="reelName"></StoryHead>
    <div style="background-color:#b0c4de;margin: auto;padding: 10px">
        <el-row>
            <el-col :span="20">
                <h1>所属分卷 --  {{reelName}}</h1>
            </el-col>
            <el-col :span="4" style="text-align: right;">
                <div style="text-align: right; font-size: 12px" class="toolbar">
                    <el-button text @click="goBack()">返回</el-button>
                </div>
            </el-col>
        </el-row>
    </div>
  <!--        表格-->
  <div style="margin: auto;padding: 10px">
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
import { getChapter } from "@/api/admin/chapter";
import { addDraftChapter } from "@/api/admin/draftChapter";
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
import StoryHead from "./storyHead.vue";

// 接收url里的参数
const route = useRoute();
const router = useRouter()

const sid = ref(route.query.sid);
const scid = ref(route.query.scid);
const scname = ref(route.query.scname);
const reelName = ref(route.query.scname);
const sname = ref(route.query.sname);


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
    pid:scid.value,
    level:1
  },
  rules: {
    title: [{ required: true, message: "章节名称不能为空", trigger: "blur" }, { min: 2, max: 20, message: "章节名称长度必须介于 2 和 20 之间", trigger: "blur" }],
  }
});
const ruleFormRef = ref<FormInstance>()
const { queryParams, form, rules } = toRefs(data);
const chapter=ref({})

const handleReelAdd = async (formEl: FormInstance | undefined) => {
  if (!formEl) return

  if (scid.value == undefined || scid.value == null || scid.value == '') {
    ElMessage.error("非法操作")
    return
  }
  form.value.sid=chapter.value.sid
  form.value.pid=chapter.value.pid
  form.value.sname=chapter.value.sname
  form.value.pname=chapter.value.pname
  form.value.isNew=2
  await formEl.validate((valid, fields) => {
    if (valid) {
      addDraftChapter(form.value).then(response => {
        // console.log("添加成功:"+JSON.stringify(response))
        router.push("/admin/draftChapter?sid="+chapter.value.sid+"&sname="+chapter.value.value+"&scid="+chapter.value.pid+"&scname="+chapter.value.pname);

      })
    } else {
      //console.log('error submit!', fields)
    }
  })
}
function handleInfo(){
  getChapter(sid.value,scid.value).then(response => {
      chapter.value=response.data
    form.value.id=chapter.value.id
    form.value.sid=chapter.value.sid
    form.value.wid=chapter.value.wid
    form.value.title=chapter.value.title
      form.value.contentZip=chapter.value.contentZip
    reelName.value=chapter.value.pname
  });
}
function handleReturn(){
  router.back()
}
function goBack() {
    // 使用 router.go(-1) 返回上一页
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
