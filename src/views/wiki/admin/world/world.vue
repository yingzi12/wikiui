<template>
        <div>
          <el-menu
              :default-active="1"
              mode="horizontal"
              style="margin:0px;pardding:0px"
          >
            <el-menu-item index="1"><span style="font-size: 20px;font-weight:bold;">世界管理</span></el-menu-item>
          </el-menu>
        </div>
        <!--        多选-->
        <div style="padding: 10px">
          <el-space wrap>
            <el-button text @click="findType(null)">全部</el-button>
            <div v-for="types in worldTypes" :key="i">
              <el-button text @click="findType(types.id)">{{types.name }} </el-button>
            </div>
          </el-space>
        </div>
        <!--        统计-->
        <div style="background-color:#b0c4de;margin: auto;padding: 10px">
          <el-row>
            <el-col  :span="20">
                <el-input v-model="queryParams.name" placeholder="请输入世界名称" class="input-with-select" style="width: 250px"/>
                <el-button :icon="Search" circle @click="getList(1)"/>
            </el-col >
            <el-col :span="4"  style="text-align: right;">
              <div style="text-align: right; font-size: 12px" class="toolbar">
                <el-button text @click="handleAdd">创建世界</el-button>
              </div>
            </el-col>
          </el-row>
        </div>
        <!--        表格-->
        <div>
          <el-scrollbar>
            <el-table v-loading="loading" :data="worldList" @selection-change="handleSelectionChange">
              <el-table-column label="序号" width="80" >
                <template #default="scope">
                  {{scope.$index+1+(queryParams.pageNum-1)*20}}
                </template>
              </el-table-column>
              <el-table-column label="名称" align="center" key="name" prop="name"  :show-overflow-tooltip="true">
                <template #default="scope">
                    <el-icon   color="#F56C6C" v-if=" scope.row.countAuditElement>0 ||  scope.row.countAuditStory>0 ||  scope.row.countAuditDiscuss >0 "><WarningFilled /></el-icon>
                  <router-link  :to="{path:'/admin/worldInfo', query: {wid:scope.row.id,wname:scope.row.name}}"><el-tag v-if="scope.row.source=='原创'">原创</el-tag>{{scope.row.name}}</router-link>
                </template>
              </el-table-column>
              <el-table-column type="expand">
                <template #default="props">
                  <el-descriptions title="详细" style="margin: 20px">

                    <el-descriptions-item label="是否私有">{{ props.row.isPrive }}</el-descriptions-item>
                    <el-descriptions-item label="经验">{{ props.row.vitality }}</el-descriptions-item>
                    <el-descriptions-item label="得分">{{ world.scores }}</el-descriptions-item>
                    <el-descriptions-item label="元素数">{{ props.row.countElement }}</el-descriptions-item>
                    <el-descriptions-item label="居民数">{{ props.row.countResident }}</el-descriptions-item>
                    <el-descriptions-item label="评论数">{{ props.row.countComment }}</el-descriptions-item>
                    <el-descriptions-item label="点赞数">{{ props.row.countLike }}</el-descriptions-item>
                    <el-descriptions-item label="关注数">{{ props.row.countFllow }}</el-descriptions-item>
                    <el-descriptions-item label="故事数">{{ props.row.countStory }}</el-descriptions-item>
                    <el-descriptions-item label="编辑数">{{ props.row.countEdit }}</el-descriptions-item>
                    <el-descriptions-item label="创建人">{{ props.row.createName }}</el-descriptions-item>
                    <el-descriptions-item label="创建时间">{{ props.row.createTime }}</el-descriptions-item>
                    <el-descriptions-item label="更新人">{{ props.row.updateName }}</el-descriptions-item>
                    <el-descriptions-item label="更新时间">{{ props.row.updateTime }}</el-descriptions-item>
                  </el-descriptions>
                </template>
              </el-table-column>
              <el-table-column label="等级" align="center" key="ranks" prop="ranks"   width="50"/>
              <el-table-column label="类型" align="center" :show-overflow-tooltip="true"  width="80" >
               <template #default="scope">
                <span>{{worldTypesMap.get(scope.row.types)}}</span>
                </template>
              </el-table-column>
              <el-table-column label="简介" align="center" key="intro" prop="intro"  :show-overflow-tooltip="true" />
              <el-table-column label="状态" align="center"  width="80" >
                <template #default="scope">
                  <span>{{worldStatusMap.get(scope.row.status)}}</span>
                </template>
              </el-table-column>
              <el-table-column label="操作" align="center" width="150" class-name="small-padding fixed-width">
                <template #default="scope">
                  <el-tooltip content="详情" placement="top" >
                    <el-button
                        type="text"
                        icon="View"
                        @click="handleSee(scope.row)"
                    ></el-button>
                  </el-tooltip>
                  <el-tooltip content="修改" placement="top" >
                    <el-button
                        type="text"
                        icon="Edit"
                        @click="handleUpdate(scope.row)"
                    ></el-button>
                  </el-tooltip>
                  <el-tooltip v-if="scope.row.status == 1" content="发布" placement="top" >
                    <el-button
                        type="text"
                        icon="Position"
                        @click="handleIssue(scope.row)"
                    ></el-button>
                  </el-tooltip>
                  <el-tooltip content="删除" placement="top" >
                    <el-button
                        type="text"
                        icon="Delete"
                        @click="handleDelete(scope.row)"
                    ></el-button>
                  </el-tooltip>
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
              @current-change="getList"
          />
        </div>
</template>

<script lang="ts" setup>
import { inject, reactive, ref, toRefs} from 'vue'
import { listMangeWorld as listWorld,delWorld,issue } from "@/api/admin/world";
import { useRouter} from "vue-router";
import {ElMessage, ElMessageBox} from "element-plus";
import {Search} from "@element-plus/icons-vue";
import { worldStatusMap,worldTypesMap } from "@/utils/constant";

const fits = ['世界', '粉丝', '关注']
const activeIndex = ref('1')
const baseUrl = inject("$baseUrl")

const router = useRouter()


class World {
  id: number
  name: string
  types: string
  intro: string
  createTime:string
}


const loading = ref(true);
const worldList = ref([]);
const total = ref(0);
const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,

    name: undefined,
    types: undefined,
  },
  rules: {
    // userName: [{ required: true, message: "用户名称不能为空", trigger: "blur" }, { min: 2, max: 20, message: "用户名称长度必须介于 2 和 20 之间", trigger: "blur" }],
  }
});
const worldTypes=reactive([{id:6,name:"科学"},{id:1,name:"武侠"},{id:2,name:"仙侠"},{id:3,name:"魔幻"},{id:4,name:"奇幻"},{id:5,name:"其他"}])
const { queryParams, form, rules } = toRefs(data);
const dateRange = ref([]);
const ids = ref([]);
const single = ref(true);
const multiple = ref(true);
const search = ref('')
function handleUpdate (row)  {
  router.push("/admin/worldEdit?wid="+row.id+"&wname="+row.name);
}
function handleAdd ()  {
  router.push("/world/add");
}

function handleDelete ( row){
  const worldId = row.id ;
  ElMessageBox.confirm('是否确认删除世界名称为"' + row.name + '"的数据？').then(function () {
    return delWorld(worldId);
  }).then(() => {
    getList(queryParams.value.pageNum);
    ElMessage.success("删除成功");
  }).catch(() => {});
}

function handleSee(row){
  router.push("/admin/worldInfo?wid="+row.id);
}
function handleIssue(row){

  issue(row.id).then(response => {
    getList(queryParams.value.pageNum);

  });
}
/** 选择条数  */
function handleSelectionChange(selection) {
  ids.value = selection.map(item => item.userId);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}
/**根据分类查询世界*/
function findType(typeId:number) {
  queryParams.value.types=typeId;
  listWorld(queryParams.value).then(response => {
    loading.value = false;
    worldList.value = response.data;
    total.value = response.total;
  });
}
/** 查询世界列表 */
function getList(page: number) {
  window.scrollTo(0, 0); // 滚动到顶部
  queryParams.value.pageNum=page;

  listWorld(queryParams.value).then(response => {
    loading.value = false;
    worldList.value = response.data;
    total.value = response.total;
  });
}
getList(queryParams.value.pageNum);

</script>

<style scoped>

.center {
  display: flex;
  justify-content: center;
  align-items: center;
}


</style>
