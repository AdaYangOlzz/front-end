<template>
    <div class="content">
        <!-- 视频任务配置new  -->
        <el-card shadow="hover" style="margin-left: 20px;margin-right: 20px;">
            <div slot="header" style="font-size: 20px; font-weight: bold; margin-bottom: 20px;">视频任务配置</div>
            
            <!-- 显示当前页的内容 -->
            <div style="display: flex; flex-direction: column; align-items: center;">
              <!-- 上面的 div 包含 v-for 循环的内容 -->
              <div>
                <div v-for="(values, job_id, index) in currentPageItems" class="available-node-result"
                  v-on:click="selectItem(job_id)"
                  v-bind:class="{ 'selected': selected === job_id }"
                >
                  <div class="centered-div">
                    <ul style="list-style-type: none;">
                        <li style="font-size: 16px; font-weight: bold;">{{values.selectedIp}}</li>
                        <li class="subli">摄像头ID: {{ values.selectedVideoId }}</li>
                        <li class="subli">描述: {{ values.type }}</li>
                        <li class="subli">优化模式: {{ values.mode == 'latency'? '时延优先':'精度优先' }}</li>
                        <li class="subli">
                            {{ values.mode === 'latency' ? '时延约束' : '精度约束' }}: {{ values.mode === 'latency' ? values.delay_constraint : values.acc_constraint }}
                            <!-- {{ values.delay_constraint }} - {{ values.acc_constraint }} -->
                        </li>
                    </ul>
                  </div>
                </div>
              </div>

              <!-- 下面的 div 包含分页控件 -->
              <div class="pagination">
                <el-button @click="prevPage" :disabled="currentPage === 1">上一页</el-button>
                <el-button @click="nextPage" :disabled="currentPage === pageCount">下一页</el-button>
              </div>
            </div>

        </el-card>

        <el-card shadow="hover" style="margin: 20px;">
          <div ref="checkboxContainer">
            <el-checkbox @change="showPic()" :disabled="!selected" v-model="checkedPic">原始视频流</el-checkbox>
            <!-- <el-checkbox v-for="(item, idx) in keyList" :key="idx" :label="item" @change="toggleChart(item)" :disabled="!selected">{{ item }}</el-checkbox> -->
            <el-checkbox v-for="(item, idx) in modifiedKeyList" :key="idx" :label="item" @change="toggleChart(mapBack(item))" :disabled="!selected" v-model="checkboxes[idx]">{{ item }}</el-checkbox>
  
          </div>

          <!-- 画布容器 -->
          <div>
            <div v-show="showOriginal" style="width: 500px; height: 250px; margin-top: 20px; float: left;">
              <img :src="videoUrl + selected" style="width: 100%; height: 90%;" />
            </div>
            
            <div v-for="(item, idx) in keyList" :key="idx" style="float: left;margin-left: 20px;">
              <div v-show="showChart(item)" :id="item" style="width: 500px; height: 250px; margin-top: 20px;"></div>
            </div>
          </div>

        </el-card>



    </div>
</template>
<script>
import { ElLoading, ElMessage } from "element-plus";
import * as echarts from "echarts";
export default{
    data(){
        return{
            // url
            resultUrl: "/dag/query/get_agg_info/",
            resourceUrl: "/serv/get_cluster_info",
            videoUrl: "video/user/video/",

            cluster_info:null,
            selectedIp:null,
            // 资源
            resource_display:[],
            // 可查询任务
            submit_jobs:[],
            // 选取的查询任务
            submit_job:null,
            // 查询结果
            result:null,
            appended_result:null,
            runtime:null,
            plan:null,

            // 结果细节
            delay: null,
            obj_n: null,
            obj_size: null,
            obj_stable: null,

            // 选择的查询任务
            selected: null,

            node_type_list: [],
            // 前端获取已经提交的任务
            job_info_dict:{},

            // 绘制结果折线图
            chart:null,

            // 定时器
            timer:null,

            // 分页控件
            itemsPerPage: 3, // 每页显示的数量
            currentPage: 1, // 当前页数

            // checkbox相关
            keyList:[],
            checkedKey:null,
            showOriginal:false,
            selectedCharts:[],
            showOverview:false,

            checkedPic:false,
            checkboxes:[],

            // 一段时间折线图
            maxCount:10,
            delay_list:{},
            objn_list:{},
            objsize_list:{},
            objstable_list:{},

            
        };
    },
    methods: {
        // 绘制饼图
        // drawTest(){
        //   var chart = echarts.init(document.getElementById("chart2"));
        //   const appended_result = this.appended_result;

        //   const data = this.appended_result[this.appended_result.length - 1];
        //   var keyList = Object.keys(data["count_result"]);
        //   var valueList = Object.values(data["count_result"]);

        //   var seriesData = [];
        //   for(var i = 0;i < keyList.length;i ++){
        //     seriesData.push({
        //       value: valueList[i],
        //       name: keyList[i],
        //     });
        //   }
        //   var option = {
        //     series: [
        //       {
        //         type: 'pie',
        //         data: seriesData,
        //       }
        //     ]
        //   };
        //   chart.setOption(option);
        // },
        mapTOChinese(value){
          if(value === "up"){
            return "抬头人数";
          }else if(value === "total"){
            return "总人数";
          }else if(value === "car"){
            return "汽车";
          }else if(value === "truck"){
            return "卡车";
          }else if(value === "train"){
            return "火车";
          }else if(value === "bus"){
            return "公交车";
          }else if(value === "motorcycle"){
            return "摩托车"
          }else if(value === "person"){
            return "行人";
          }else{
            return value;
          }
        },
        mapBack(value){
          if(value === "抬头人数"){
            return "up";
          }else if(value === "总人数"){
            return "total";
          }else if(value === "汽车"){
            return "car";
          }else if(value === "卡车"){
            return "truck";
          }else if(value === "火车"){
            return "train";
          }else if(value === "公交车"){
            return "bus";
          }else if(value === "摩托车"){
            return "motorcycle";
          }else if(value === "行人"){
            return "person";
          }
          else{
            return value;
          }
        },
        showPic(){
          if(!this.showOriginal){
            this.showOriginal = true;
          }else{
            this.showOriginal = !this.showOriginal;
          }
          console.log(this.showOriginal);
        },

        toggleChart(itemKey){
          // 已经被选择则清除
          // console.log(this.selectedCharts);
          if (this.selectedCharts.includes(itemKey)) {
            this.selectedCharts = this.selectedCharts.filter(item => item !== itemKey);
            // console.log("尝试清除");
            echarts.dispose(document.getElementById(itemKey));
          } else {
            this.selectedCharts.push(itemKey);
            this.showSelectedResult(itemKey);
          }
          console.log(this.selectedCharts);

          // 没被选择则加入到list中
        },
        showChart(itemKey){
          return this.selectedCharts.includes(itemKey);
        },

        showSelectedResult(itemKey){
          // console.log(itemKey);
          // this.updateResultUrl();
          var chart;
          if (echarts.getInstanceByDom(document.getElementById(itemKey)) === undefined) {
            chart = echarts.init(document.getElementById(itemKey));
          } else {
            chart = echarts.getInstanceByDom(document.getElementById(itemKey));
            // chart.clear();
          }
          // var chart = echarts.init(document.getElementById(itemKey));
          
          var xAsixName = "n_loop";

          const data = this.appended_result.map((item) => ({
            x:item.n_loop,
            y:item.count_result[itemKey],
          }))
          // console.log(data);
          const option = {
            xAxis: {
              type: 'category',
              data: data.map((item) => item.x), // 使用映射后的横坐标数据
              name: "帧数"
            },
            yAxis:{
              type:'value',
              name:'检测到的数量'
            },
            series:[
              {
                data: data.map((item) => item.y),
                type:'line',
                label:{
                  show:true,
                  position:'bottom',
                  textStyle:{
                    fontSize:12
                  }
                }
              }
            ],
            title:{
              show:true,
              text: this.mapTOChinese(itemKey),
            }
            // legend:{
            //   data:[key],
            // }
            
          };
          chart.setOption(option);
        },

        // 绘制结果折线图
        drawResult(){
          this.updateResultUrl();
          this.showOverview = !this.showOverview;
          // console.log(this.showOverview);
          if(this.showOverview){
            const chart = echarts.init(document.getElementById('result'));

            const yKey = Object.keys(this.appended_result[0])[0]; // 获取第一个键名作为纵坐标的键名

            const data = this.appended_result.map((item) => ({
              x: item.n_loop,
              y: item[yKey], // 使用纵坐标的键名来获取对应的值
            }));

            const appended_data = this.appended_result;

            // 获取追加结果中key非n_loop的数据
            var xAsixName = "n_loop";
            var seriesData = {};
            var resKeyList = [];
            // console.log("appended_data length: " + appended_data.length);
            for (var i = 0; i < appended_data.length; i++) {
              var keyList = Object.keys(appended_data[i]["count_result"]);
              for (var j = 0; j < keyList.length; j++) {
                // 以抬头检测为例,将up total这些key都加入到resKeyList中
                if (resKeyList.indexOf(keyList[j]) == -1) {
                  resKeyList.push(keyList[j]);
                }
              }
              // resKeyList = resKeyList.concat(keyList);
            }
            // resKeyList = [new Set(resKeyList)]
            // console.log("resKeyList: " + resKeyList);
            // var resKeyList = Object.keys(appended_data[0])
            for (var i = 0; i < resKeyList.length; i++) {
              var key = resKeyList[i];
              if (key !== xAsixName) {
                seriesData[key] = [];
              }
            }
            // console.log("init seriesData keys: " + Object.keys(seriesData));
            // 生成各key的数据序列
            for (var i = 0; i < appended_data.length; i++) {
              var frameInfo = appended_data[i]["count_result"];
              for (var j in frameInfo) {
                var key = j;
                var value = frameInfo[j];
                if (key !== xAsixName) {
                  seriesData[key].push(value);
                }
              }
            }
            // 生成series列表和与其对应的legend列表
            var seriesList = [];
            var legendList = [];
            // console.log("seriesData entries: " + Object.entries(seriesData));
            for (var ent of Object.entries(seriesData)) {
              // console.log("ent[0]=" + ent[0]);
              // console.log("ent[1]=" + ent[1]);
              legendList.push(ent[0]);
              seriesList.push({
                name: ent[0],
                type: "line",
                // type: "bar",
                // stack: "stack",
                label: {
                  show: true,
                  position: "top",
                  color: "black",
                  fontSize: 12,
                  formatter: function (d) {
                    return d.data;
                  },
                },
                data: ent[1],
              });
            }

            const option = {
              grid: {
                bottom: "10%",
                right: "15%",
                top: "30%",
                left: "15%",
              },
              xAxis: {
                type: "category",
                data: data.map((item) => item.x), // 使用映射后的横坐标数据
                name: "帧数",
              },
              yAxis: {
                type: "value",
                name: "检测到的数量",
              },
              legend: {
                data: legendList,
              },
              series: seriesList,
              // series: [
              //   {
              //     type: "line",
              //     data: data.map((item) => item.y), // 使用映射后的纵坐标数据
              //   },
              // ],
            };
            // console.log("尝试画图...");
            chart.setOption(option,true);
          }else{
            // console.log('尝试清除...');
            echarts.dispose(document.getElementById('result'));
          }
          

        },
        // 绘制圆环
        drawCircle(canvas,content_filled,percentage,color) {
            // const canvas = this.$refs.circleCanvas;
            const context = canvas.getContext('2d');

            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;
            const radius = Math.min(centerX, centerY) - 20;
            const lineWidth = 10;
            // const bandwidth = "50 Mbps"; // 网络带宽文本

            context.clearRect(0, 0, canvas.width, canvas.height);

            // 绘制圆环的背景
            context.beginPath();
            context.arc(centerX, centerY, radius, 0, Math.PI * 2);
            context.lineWidth = lineWidth;
            context.strokeStyle = '#ccc'; // 圆环的颜色
            context.stroke();

            // 绘制网络带宽文本
            context.font = '20px Arial';
            context.fillStyle = '#333'; // 文本颜色
            context.textAlign = 'center';
            context.textBaseline = 'middle';
            context.fillText(content_filled, centerX, centerY);

            // 绘制圆环的前景
            // const percentage = 1; // 设置圆环的百分比
            const endAngle = (Math.PI * 2 * percentage) - (Math.PI / 2);
            context.beginPath();
            context.arc(centerX, centerY, radius, -Math.PI / 2, endAngle);
            context.lineWidth = lineWidth;
            context.strokeStyle = color; // 前景颜色
            context.stroke();
        },
        prevPage() {
          if (this.currentPage > 1) {
            this.currentPage--;
          }
        },
        nextPage() {
          if (this.currentPage < this.pageCount) {
            this.currentPage++;
          }
        },
        
        // 点击选择查询任务
        selectItem(job_id){
        //   console.log(job_id);
          this.selected = job_id;
          this.submit_job = job_id;
          this.selectedCharts = [];
          this.showOriginal = false;
          this.checkedPic = false;
          this.clearCheckboxes();
          this.updateResultUrl();
          // this.fetchAndProcessData();
        },
        updateKeyList(){
          // 获取appended_result中的key值
          const data = this.appended_result;
          // console.log(data);
          const resKeyList = [];
          for(var i = 0;i < data.length;i ++){
            var keylist =  Object.keys(data[i]['count_result']);
            for(var j = 0;j < keylist.length;j ++){
              if(resKeyList.indexOf(keylist[j]) == -1){
                resKeyList.push(keylist[j]);
              }
            }
          }
          const isSame = (arr1, arr2) => {
            if (arr1.length !== arr2.length) {
              return false;
            }
            
            return arr1.every(item => arr2.includes(item));
          };

          // 如果keyList有变化则清空selectedCharts和checkboxes的选中状态
          const keyListsAreSame = isSame(this.keyList, resKeyList);
          if(!keyListsAreSame){
            this.keyList = resKeyList;
            console.log(this.keyList);
            this.clearCheckboxes();
            this.selectedCharts = [];
          }
        },
        // 清除复选框勾选状态
        
        clearCheckboxes() {
          // 清除所有复选框的勾选状态
          this.checkboxes = this.checkboxes.map(() => false);
        },
        
        // 查询结果
        updateResultUrl() {
          // console.log(this.submit_job);
          if(this.submit_job !== null && this.submit_job !== undefined){
            const url = this.resultUrl + this.submit_job;

            // console.log(url);
            const loading = ElLoading.service({
              lock: true,
              text: "Loading",
              background: "rgba(0, 0, 0, 0.7)",
            });
            fetch(url)
              .then((response) => response.json())
              .then((data) => {
                loading.close();
                // console.log(data);
                // this.runtime = this.result["latest_result"]["runtime"];
                // this.plan = this.result["latest_result"]["plan"];         
                // console.log(this.runtime);
  
                this.result = data;
                // console.log("new data:" + data);
                loading.close();
                this.appended_result = data['appended_result'];
                // console.log(this.appended_result);
                const result = this.appended_result;
                const len = this.appended_result.length;
                // console.log(len);
                this.runtime = result[len - 1]['ext_runtime'];
                this.plan = result[len - 1]['ext_plan'];
  
                // 应用情境
                const job_id = this.submit_job;
                if (this.runtime && this.runtime["delay"]) {
                  this.delay = this.runtime["delay"].toFixed(2);
                  
                  if(!this.delay_list[this.submit_job]){
                    this.delay_list[job_id] = [];
                  }
  
                  this.delay_list[job_id].push(this.delay);
  
                  // 超过个数就移除
                  if(this.delay_list[job_id].length > this.maxCount){
                    this.delay_list[job_id].shift();
                  }
                }
  
                if (this.runtime && this.runtime["obj_n"]) {
                  this.obj_n = Math.floor(this.runtime["obj_n"]);
  
                  if(!this.objn_list[this.submit_job]){
                    this.objn_list[job_id] = [];
                  }
  
                  this.objn_list[job_id].push(this.obj_n);
  
                  if(this.objn_list[job_id].length > this.maxCount){
                    this.objn_list[job_id].shift();
                  }
                } 
  
                if (this.runtime && this.runtime["obj_size"]) {
                  this.obj_size = this.runtime["obj_size"].toFixed(2);
  
                  if(!this.objsize_list[this.submit_job]){
                    this.objsize_list[job_id] = [];
                  }
  
                  this.objsize_list[job_id].push(this.obj_size);
  
                  if(this.objsize_list[job_id].length > this.maxCount){
                    this.objsize_list[job_id].shift();
                  }
                }
  
                if (this.runtime && this.runtime["obj_stable"]) {
                  this.obj_stable = this.runtime["obj_stable"];
  
                  if(!this.objstable_list[this.submit_job]){
                    this.objstable_list[job_id] = [];
                  }
  
                  this.objstable_list[job_id].push(this.obj_stable);
  
                  if(this.objstable_list[job_id].length > this.maxCount){
                    this.objstable_list[job_id].shift();
                  }
                }
                
                this.updateKeyList();
  
              })
              .catch((error) => {
                console.log(error);
                loading.close();
                ElMessage({
                  showClose: true,
                  message: "结果尚未生成,请稍后",
                  type: "error",
                  duration: 1500,
                });
                this.result = null;
              });
          }
          
        },



        // 更新系统资源状态
        updateResourceUrl() {
          const url = this.resourceUrl;
          fetch(url)
            .then((resp) => resp.json())
            .then((data) => {
              // console.log(data);
              this.cluster_info = data;
            })
            .catch((err) => {
              console.log(err);
            });
        },
        
    },
    computed: {
        pageCount() {
          return Math.ceil(Object.keys(this.job_info_dict).length / this.itemsPerPage);
        },
        currentPageItems() {
          const start = (this.currentPage - 1) * this.itemsPerPage;
          const end = start + this.itemsPerPage;
          return Object.keys(this.job_info_dict).slice(start, end).reduce((obj, key) => {
            obj[key] = this.job_info_dict[key];
            return obj;
          }, {});
        },
        modifiedKeyList(){
          // console.log(this.keyList);
          if(!this.keyList){
            return null;
          }
          return Object.fromEntries(
            Object.entries(this.keyList).map(([key,value]) =>{
              if(value === "up"){
                return [key,"抬头人数"];
              }else if(value === "total"){
                return [key,"总人数"];
              } else if(value === "car"){
                return [key,"汽车"];
              } else if(value === "truck"){
                return [key,"卡车"];
              } else if(value === "train"){
                return [key,"火车"];
              } else if(value === "bus"){
                return [key,"公交车"];
              } else if(value === "motorcycle"){
                return [key,"摩托车"];
              } else if(value === "person"){
                return [key,"行人"];
              } else{
                return [key,value];
              }
            })
          );
        },
        modifiedRuntime() {
            if (!this.runtime) {
                return null;
            }
            return Object.fromEntries(
                Object.entries(this.runtime).map(([key, value]) => {
                if (key === "delay") {
                    return ["时延", value.toFixed(2)];
                } else if (key === "obj_n") {
                    return ["目标数量", value];
                } else if (key === "obj_size") {
                    return ["目标大小", value.toFixed(2)];
                } else if (key === "obj_stable") {
                    return ["场景稳定性", value];
                } else {
                    return [key, value];
                }
                })
            );
        },
        // 改写中文
        modifiedInfo() {
            if (!this.job_info_dict) {
                return null;
            }
            let modified_info = Object.fromEntries(
                Object.entries(this.job_info_dict).map(([key, value]) => {
                let mode = value.mode === "latency" ? "时延优先" : "精度优先";
                return [
                    key,
                    {
                    job_id: value.job_id,
                    selectedIp:value.selectedIp,
                    摄像头ID: value.selectedVideoId,
                    描述: value.type,
                    优化模式: mode,
                    时延约束: value.delay_constraint,
                    精度约束: value.acc_constraint
                    }
                ];
                })
            );
            return modified_info;
        },
        modifiedPlan() {
            if (!this.plan) {
                return null;
            }
            return Object.fromEntries(
                Object.entries(this.plan).map(([key, value]) => {
                if (key === "flow_mapping") {
                    return ["云边协同配置", value];
                } else if (key === "video_conf") {
                    return ["视频配置", value];
                } else {
                    return [key, value];
                }
                })
            );
        },
    },
    mounted(){
        
        // 获取可查询的任务相关信息 存储在submit_jobs和job_info_dict中
        const delay_list = sessionStorage.getItem("delay_list");
        if (delay_list) {
            this.delay_list = JSON.parse(delay_list);
            // console.log(this.submit_jobs);
        }
        const job_info = sessionStorage.getItem("job_info_dict");
        if(job_info){
            this.job_info_dict = JSON.parse(job_info);
            console.log(this.job_info_dict);
        }

        const submitJobs = sessionStorage.getItem("submit_jobs");
        if (submitJobs) {
          this.submit_jobs = JSON.parse(submitJobs);
        }
        // REMOVE: 静态填充
        this.resource_display = ["cpu_ratio","mem_ratio","gpu_ratio","net_ratio(MBps)"];
        
        this.appended_result = [
        {
            "count_result": {
                "total": 19,
                "up": 16
            },
            "delay": 0.22060694013323104,
            "execute_flag": true,
            "frame_id": 774,
            "n_loop": 63,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.27425,
                    "latency": 1.1754405498504639,
                    "pid": 11065
                }
            ]
        },
        {
            "count_result": {
                "total": 18,
                "up": 15
            },
            "delay": 0.20396453993661062,
            "execute_flag": true,
            "frame_id": 780,
            "n_loop": 64,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.27399999999999997,
                    "latency": 1.0766024589538574,
                    "pid": 11065
                }
            ]
        },
        {
            "count_result": {
                "total": 16,
                "up": 12
            },
            "delay": 0.18409783499581472,
            "execute_flag": true,
            "frame_id": 786,
            "n_loop": 65,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.27949999999999997,
                    "latency": 0.9475843906402588,
                    "pid": 11065
                }
            ]
        },
        {
            "count_result": {
                "total": 19,
                "up": 16
            },
            "delay": 0.21081665584019255,
            "execute_flag": true,
            "frame_id": 792,
            "n_loop": 66,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.27725,
                    "latency": 1.1265530586242676,
                    "pid": 11065
                }
            ]
        },
        {
            "count_result": {
                "total": 17,
                "up": 16
            },
            "delay": 0.21186903544834684,
            "execute_flag": true,
            "frame_id": 798,
            "n_loop": 67,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.26975,
                    "latency": 1.1213617324829102,
                    "pid": 11065
                }
            ]
        },
        {
            "count_result": {
                "total": 19,
                "up": 14
            },
            "delay": 0.21648645401000977,
            "execute_flag": true,
            "frame_id": 804,
            "n_loop": 68,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.273,
                    "latency": 1.1257836818695068,
                    "pid": 11065
                }
            ]
        },
        {
            "count_result": {
                "total": 18,
                "up": 16
            },
            "delay": 0.21124557086399626,
            "execute_flag": true,
            "frame_id": 810,
            "n_loop": 69,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.276,
                    "latency": 1.0686159133911133,
                    "pid": 11065
                }
            ]
        },
        {
            "count_result": {
                "total": 20,
                "up": 15
            },
            "delay": 0.21829724311828613,
            "execute_flag": true,
            "frame_id": 816,
            "n_loop": 70,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.271,
                    "latency": 1.180079460144043,
                    "pid": 11065
                }
            ]
        },
        {
            "count_result": {
                "total": 20,
                "up": 15
            },
            "delay": 0.21993769918169295,
            "execute_flag": true,
            "frame_id": 822,
            "n_loop": 71,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.27575,
                    "latency": 1.1873185634613037,
                    "pid": 11065
                }
            ]
        },
        {
            "count_result": {
                "total": 19,
                "up": 15
            },
            "delay": 0.2089878831590925,
            "execute_flag": true,
            "frame_id": 828,
            "n_loop": 72,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.27949999999999997,
                    "latency": 1.1177542209625244,
                    "pid": 11065
                }
            ]
        },
        {
            "count_result": {
                "total": 18,
                "up": 14
            },
            "delay": 0.20454134259905135,
            "execute_flag": true,
            "frame_id": 834,
            "n_loop": 73,
            "proc_resource_info_list": [
                {
                    "cpu_util_limit": 1,
                    "cpu_util_use": 0.27425,
                    "latency": 1.0840375423431396,
                    "pid": 11065
                }
            ]
        }
    ];
      this.updateKeyList();

      // this.initChart();
      this.timer = setInterval(() => {
        this.updateResultUrl();
        this.updateResourceUrl();
        // TODO:updateKeyList后会导致选中的复选框内容可能变化
        // 解决:如果keyList发生变化则清空已选内容和复选框勾选状态;不变则继续更新原来的折线图
        for(var i = 0;i < this.selectedCharts.length;i ++){
          this.showSelectedResult(this.selectedCharts[i]);
        }
      }, 6000);
      
      
    },
    beforeUnmount() {
      clearInterval(this.timer);
    },
}
</script>
<style>
.content {
    margin-top: 20px;
}
.card-container {
  display: flex;
  margin: 20px;
}
.image-container{
    width: 100%;
    height: 80%;
    display: flex;
    justify-content: center;
    align-items: center;
}
.card {
  height: 600px;
  margin-right: 10px; /* 添加适当的间距 */
}
.info-header {
  text-align: center;
  color: #2f74ff;
  font-weight: 1000;
  line-height: 40px;
  border: 2px solid rgb(77, 77, 77);
  margin: 5px;
  border-radius: 5px;
}
.carousel-item-content {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  height: 100%; /* 设置高度以确保内容垂直居中 */
}

.horizontal-scroll-container {

  display: flex;
  overflow-x: auto;
  white-space: nowrap;

  max-width: 80%; /* 设置最大宽度为80% */
  margin: 0 auto; /* 水平居中 */
}

.available-node-result{
  display: inline-block;
  margin: 20px;
  /* background-color: cadetblue; */
  width: 320px;
  border: 1px solid #5c5858;
  border-radius: 10px;
  height: 180px;
}
.centered-div {
  margin-top: 10px;
  display: flex;
  justify-content: center; /* 在水平方向上居中对齐 */
  
}
.selected{
  /* background-color: rgb(96, 158, 254);
  color:white; */
  background-color: #deebf7;
}
.canvas-container {
    display: flex;
    justify-content: space-between;
}

.inner-div {
    display: flex;
    flex-direction: column;
    align-items: center;
}

canvas {
    margin-top: 10px; /* 为了将span内容留出一些空间 */
}
.custom-select {
  position: relative;
  display: inline-block;
  width: 200px; /* 自定义宽度 */
  background-color: #f5f5f5; /* 自定义背景颜色 */
  border: 1px solid #ccc; /* 自定义边框样式 */
  border-radius: 4px; /* 自定义圆角 */
  overflow: hidden; /* 隐藏溢出内容 */
}

select {
  width: 100%;
  padding: 10px; /* 自定义内边距 */
  background-color: transparent; /* 透明背景色 */
  border: none; /* 移除默认边框 */
  outline: none; /* 移除选中边框 */
  appearance: none; /* 隐藏默认下拉箭头 */
  cursor: pointer; /* 鼠标样式 */
}
.custom-arrow {
  position: absolute;
  top: 50%;
  right: 10px;
  transform: translateY(-50%);
  font-size: 16px; /* 自定义箭头图标大小 */
}
.subli {
  font-size: 18px;
  margin-top: 10px;
  padding: 0px;
}
</style>

