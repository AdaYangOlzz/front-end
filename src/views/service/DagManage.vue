<template>
    <div class="outline">
        <div>
          <h3>现有任务流水线</h3>
        </div>

        <!-- <el-table :data="dagList" border style="width: 100%">
            <el-table-column prop="dag_name" label="name" width="180" />
            <el-table-column prop="dag" label="dag" width="360" />
            
        </el-table> -->

        <el-table :data="dagList" style="width: 100%">
            <el-table-column label="dag_name" width="180">
            <template #default="scope">
                <div style="display: flex; align-items: center">
                <!-- <el-icon><timer /></el-icon> -->
                <span style="margin-left: 10px">{{ scope.row.dag_name }}</span>
                </div>
            </template>
            </el-table-column>
            <el-table-column label="dag" width="540">
            <template #default="scope">
                <div>{{ scope.row.dag }}</div>
            </template>
            </el-table-column>
            <el-table-column label="Operations">
            <template #default="scope">
                <el-button size="small" @click="handleEdit(scope.$index, scope.row)"
                >Edit</el-button
                >
                <el-button
                size="small"
                type="danger"
                @click="handleDelete(scope.$index, scope.row)"
                >Delete</el-button
                >
            </template>
            </el-table-column>
        </el-table>

        <br>
        <div>
            <el-input v-model="editInput" :disabled="editDisabled" placeholder="Please input" />
        </div>

        <br>

        <div>
          <el-button type="primary" round @click="handleEditSubmit">提交更改</el-button>
          <!-- <el-button round>修改</el-button>
          <el-button round>删除</el-button> -->
        </div>
        <br>
<br>
        <div>
          <h3>新增任务流水线</h3>
        </div>

        <div>
                <div class="new-dag-font-style">dag_name: </div>
                <el-input v-model="newInputName" placeholder="name" />
                <br>
                <br>
                <div class="new-dag-font-style">dag: </div>

                <ul style="list-style-type: none" class="svc-container">
                  <li
                    v-for="(service, index) in services"
                    :key="index"
                    class="svc-item"
                  >
                    <el-button
                      :label="service"
                      @click="putSvcIntoList(service)"
                      >{{ service }}</el-button>
                    <!-- <el-divider /> -->
                  </li>
                </ul>

                <el-input v-model="newInputDag" placeholder="[]" />


        </div>
        <br/>

        <div>
          <el-button type="primary" round @click="handleNewSubmit">提交新增</el-button>
          <!-- <el-button round>修改</el-button>
          <el-button round>删除</el-button> -->
        </div>



    </div>
</template>

<script>
import { ElTable, ElTableColumn, ElTooltip, ElTag, ElInput, ElButton} from 'element-plus';
export default {
    components: {
        ElTable,
        ElTableColumn,
        ElTooltip,
        ElTag,
        ElInput,
        ElButton
    },
    data(){
        return{
            services: [],
            editInput: '',
            newInputName: '',
            newInputDag: '',
            editDisabled: true,
            editingIndex: -1,
            editingRow: null,
            dagList:[
                // {
                //     "dag_name":"headup",
                //     "dag":["face_detection","face_alignment"]
                // },
                // {
                //     "dag_name":"traffic",
                //     "dag":["car_detection","plate_recognition"]
                // },
                // {
                //     "dag_name":"ixpe",
                //     "dag":["ixpe_preprocess","ixpe_sr_and_pc","ixpe_edge_observe"]
                // }
                
            ],
        };
    },
    methods: {
        handleEdit(index, row) {
            this.editDisabled = false;
            this.editingIndex = index;
            this.editingRow = row;
            this.editInput = JSON.stringify(row.dag);
        },
        handleDelete(index, row) {
            this.dagList.splice(index, 1);
            this.updateDagList();
        },
        handleEditSubmit() {
            // 更新数据列表中对应索引的行
            // this.editingRow.dag = JSON.parse(this.editInput);
            // 更新dagList字段
            this.dagList[this.editingIndex].dag = JSON.parse(this.editInput);
            this.updateDagList();

            // 重置编辑状态
            this.editingIndex = -1;
            this.editInput = '';
            this.editDisabled = true;
        },
        handleNewSubmit(){
            this.dagList.push({
                "dag_name":this.newInputName,
                "dag":JSON.parse(this.newInputDag)
            });
            this.updateDagList();
            this.newInputName = '';
            this.newInputDag = '';
        },
        getDagList(){
            fetch('/serv/get-dag-workflows-api') 
            .then(response => response.json())
            .then(data => {
                console.log(data);
                // 成功获取数据后，更新dagList字段
                this.dagList = data;
            })
            .catch(error => {
                console.error('Error fetching data:', error);
            });
        },
        fetchData(){
          this.getDagList();
        },
        updateDagList() {
          fetch('/serv/update-dag-workflows-api', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(this.dagList)
          })
          .then(response => response.json())
          .then(data => {
            console.log('Data sent successfully:', data);
          })
          .catch(error => {
            console.error('Error sending data:', error);
          });
        },
        async getServiceList() {
          const response = await fetch("/serv/get_service_list");
          const data = await response.json();
          // const data = ["face_detection","face_alignment","car_detection","helmet_detection","ixpe_preprocess","ixpe_sr_and_pc","ixpe_edge_observe"]
          this.services = data;
          // console.log(this.services);
      },
      putSvcIntoList(service){
        service = "\"" + service + "\""
        if(this.newInputDag == ''){
          this.newInputDag = '[' + service + ']'
        }else{
          service = ',' + service;
          const lastBracketIndex = this.newInputDag.lastIndexOf(']');
          if (lastBracketIndex !== -1) {
            this.newInputDag = this.newInputDag.slice(0, lastBracketIndex) + service + this.newInputDag.slice(lastBracketIndex);
          } else {
            this.newInputDag += service;
          }   
      }
    }
  },
  mounted() {
          // 初次加载数据
          this.fetchData();

          // 每隔一段时间获取一次数据
          setInterval(() => {
            this.fetchData();
          }, 5000); 

          this.getServiceList();
          setInterval(this.getServiceList, 5000);
        },
};
</script>

<style scoped>
body {
  font-family: Arial, sans-serif;
  background-color: #f9f9f9;
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

form {
  max-width: 600px;
  padding: 20px;
  background-color: #fff;
  border-radius: 8px;
  /* box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1); */
}

h3 {
  font-size: 24px;
  color: #333;
  margin-bottom: 20px;
}

input[type="text"],
input[type="file"] {
  width: calc(100% - 20px);
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
}

input[type="file"] {
  cursor: pointer;
}

.el-button {
  font-size: 16px;
  margin-right: 10px;
}

.el-button:first-child {
  margin-left: 0;
  
  
}

.outline{
    /* max-width: 600px; */
    padding: 20px;
    background-color: #fff;
    border-radius: 8px;
    /* box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1); */
  }

.new-dag-font-style{
    font-size: 16px;
    margin-bottom: 20px;
    font-weight: bold;
}

.svc-container {
    display: flex;
    flex-wrap: wrap;
    padding: 5px; /* 可根据需要调整 */
    list-style-type: none;
}

.svc-item {
    margin: 2px; /* 可根据需要调整 */
    padding: 2px; /* 可根据需要调整 */
    border-radius: 10px; /* 圆角矩形 */
}

.el-button {
    font-size: 16px;
    margin-right: 10px;
}

.el-button:first-child {
  margin-left: 0;
    
}
</style>


