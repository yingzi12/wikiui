<template>
    <StoryHead :head-type="6" second-type="1" :sid="sid" :sname="sname"></StoryHead>

  <!--        统计-->
        <div style="background-color:#b0c4de;margin: auto;padding: 10px">
          <el-row>
            <el-col  :span="20">
              <el-select v-model="queryParams.types" placeholder="类型" clearable >
                <el-option
                    v-for="item in discussTypes"
                    :key="item.id"
                    :label="item.name"
                    :value="item.id"
                />
              </el-select>
              <el-select placeholder="处理状态" v-model="queryParams.status" clearable >
                <el-option
                    v-for="item in discussStatus"
                    :key="item.id"
                    :label="item.name"
                    :value="item.id"
                />
              </el-select>
              <el-input v-model="queryParams.title" placeholder="请输入主题" class="input-with-select" style="width: 250px"/>
              <el-button :icon="Search" circle @click="getList(1)"/>
            </el-col >
            <el-col :span="4"  style="text-align: right;">
              <div style="text-align: right; font-size: 12px" class="toolbar">
              </div>
            </el-col>
          </el-row>
        </div>
        <!--        表格-->
        <div>
          <el-scrollbar>
            <el-table :data="discussList">
              <el-table-column label="序号" width="80" >
                <template #default="scope">
                  {{scope.$index+1}}
                </template>
              </el-table-column>
              <el-table-column prop="wname" label="世界" width="100" :show-overflow-tooltip="true" />
              <el-table-column prop="etitle" label="元素" width="100" :show-overflow-tooltip="true" />
              <el-table-column prop="title" label="主题"   :show-overflow-tooltip="true"/>
              <el-table-column  label="类型"  :show-overflow-tooltip="true" >
                <template #default="scope">
                  <span>{{discussTypesMap.get(scope.row.types)}}</span>
                </template>
              </el-table-column>
              <el-table-column prop="comment" label="讨论内容"  :show-overflow-tooltip="true" />
              <el-table-column prop="createTime" label="创建时间"  :show-overflow-tooltip="true" />
              <el-table-column  label="状态" >
                <template #default="scope">
                  <span>{{discussStatusMap.get(scope.row.status)}}</span>
                </template>
              </el-table-column>
              <el-table-column prop="reply" label="处理说明" :show-overflow-tooltip="true"/>
              <el-table-column fixed="right" label="操作" width="150">
                <template #default="scope">
                  <el-button link type="primary" size="small" @click="handleSee(scope.row)">查看</el-button>
                  <el-button link v-if="scope.row.types != 1" type="primary" size="small" @click="handleOpen(scope.row)">处理</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-scrollbar>
        </div>
        <!--        分页-->
        <div style="float:right; position:relative; ">
          <el-pagination

              :total="total"
              layout="total, prev, pager, next"

              v-model:page="queryParams.pageNum"
              :page-size=20
              @current-change="getList"/>
        </div>

      <!--      审核弹出框-->
      <el-dialog v-model="dialogFormVisible" title="管理讨论">
        <el-form :model="form"
                 :rules="rules"
                 ref="ruleFormRef"
        >
          <el-form-item label="处理" :label-width="formLabelWidth" prop="status">
            <el-select v-model="form.status" placeholder="请选择处理意见">
              <el-option label="同意" value="2" />
              <el-option label="关闭" value="3" />
            </el-select>
          </el-form-item>
          <el-form-item label="说明" :label-width="formLabelWidth"  prop="reply">
            <el-input v-model="form.reply" autocomplete="off"  />
          </el-form-item>
        </el-form>
        <template #footer>
         <span class="dialog-footer">
           <el-button @click="dialogFormVisible = false">关闭</el-button>
           <el-button type="primary" @click="onSudmit(ruleFormRef)">确认</el-button>
        </span>
        </template>
      </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref, toRefs} from 'vue'
import {useRoute, useRouter} from "vue-router";
import { Search } from '@element-plus/icons-vue'
import { listDiscussAdmin,updateStatusAdmin } from "@/api/admin/discuss";
import {ElMessage, FormInstance, FormRules} from "element-plus";
import { discussTypesMap,discussStatusMap,discussTypes,discussStatus } from "@/utils/constant";

import StoryHead from "./storyHead.vue";

// 接收url里的参数
const route = useRoute()
const router = useRouter()

const sname = ref(route.query.sname);
const sid = ref(route.query.sid);
const wid = ref(route.query.wid);


//分页
const dateRange = ref([]);
//分类选项
const dataStree = ref([])
const loading = ref(true);
const discussList = ref([]);
const total = ref(0);
const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,

    auditStatus:0,
    title: undefined,
    types: undefined,
    status:1,
    source:2,
    sid:sid.value,
    wid:wid.value,
  },
  rules: {
    reply: [{required: true, message: '请输入回复', trigger: 'blur'},
      {min: 10, max: 200, message: '回复长度在 10 到 200 之间', trigger: 'blur'},
    ],
    status: [{required: true, message: '请选择处理类型', trigger: 'blur'}]
  }
});
const { queryParams, form, rules } = toRefs(data);

/** 查询世界列表 */
function getList(page: number) {
  window.scrollTo(0, 0); // 滚动到顶部
  queryParams.value.pageNum=page;

  listDiscussAdmin(queryParams.value).then(response => {
    loading.value = false;
    discussList.value = response.data;
    total.value = response.total;
  });
}

//弹出框
const dialogFormVisible = ref(false)
const ruleFormRef = ref<FormInstance>()
const formLabelWidth = '140px'
function handleSee(row){
  router.push("/discuss/index?wid="+row.wid+"&sid="+row.sid+"&did="+row.id+"&source=2");
}
function handleOpen(row){
  dialogFormVisible.value=true
  form.value.id=row.id
  form.value.wid=wid.value
  form.value.status=undefined
  form.value.reply=''
}

const onSudmit = async (formEl: FormInstance | undefined) => {
  if (!formEl) return
  await formEl.validate((valid, fields) => {
    if (valid) {
      form.value.source=2
      form.value.sid=sid.value
      form.value.wid=wid.value
      //console.log('submit!')
      updateStatusAdmin(form.value).then(response => {
        dialogFormVisible.value=false
        ElMessage.success("处理成功")
        getList(queryParams.value.pageNum);

      })
    } else {
      //console.log('error submit!', fields)
    }
  })
}
getList(1)
</script>

<style scoped>

.center {
  display: flex;
  justify-content: center;
  align-items: center;
}


</style>
