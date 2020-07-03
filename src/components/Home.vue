<template>
  <div class="wrap">
    <div class="acupointLeft">
      <!-- <div class="acupoint-txt" v-show="textarea"></div> -->
      <el-input
        class="acupoint-txt"
        v-show="textarea"
        type="textarea"
        :autosize="{ minRows: 2, maxRows: 20}"
        v-model="textarea"
      ></el-input>
      <div v-if="videoShow">
        <el-button plain class="btn" @click="bindAddtableRow" size="mini">添加多音字</el-button>
        <el-button plain class="btn" @click="bindremovetableRow" size="mini">移除多音字</el-button>
        <el-table :data="tableData" border class="table-wrap">
          <el-table-column prop="oldVal" label="旧值">
            <template scope="scope">
              <div class="input-box">
                <el-input
                  size="small"
                  @blur="handleInputBlur(scope.$index,scope.row)"
                  v-model="scope.row.oldVal"
                ></el-input>
              </div>
            </template>
          </el-table-column>
          <el-table-column prop="newVal" label="新值">
            <template scope="scope">
              <div class="input-box">
                <el-input
                  size="small"
                  @blur="handleInputBlur(scope.$index,scope.row)"
                  v-model="scope.row.newVal"
                ></el-input>
              </div>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="audioBox" v-if="videoShow">
        <el-button type="primary" @click="bindChangeAudio">音频生成</el-button>
        <audio controls class="audio">
          <source :src="mp3" ref="audioname" type="audio/ogg" />
          <source :src="mp3" ref="audioname" type="audio/mpeg" />
        </audio>
      </div>
    </div>
    <div class="acupointRight">
      <video controls ref="videoname" class="video" v-if="videoShow">
        <source :src="video" type="video/mp4" />
        <source :src="video" type="video/ogg" />
      </video>
      <el-select
        v-model="acupoint"
        placeholder="请选择"
        @change="bindGetacupointVal"
        clearable
        filterable
        class="select-box"
      >
        <el-option-group v-for="group in acupointData" :key="group.label" :label="group.label">
          <el-option
            v-for="item in group.options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-option-group>
      </el-select>
      <el-select
        class="select-box"
        v-model="double"
        clearable
        placeholder="请选择"
        @change="bindGetdoubleVal"
      >
        <el-option
          v-for="item in doubleData"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        ></el-option>
      </el-select>
      <el-button type="primary" class="select-box" @click="bindStart">开始</el-button>
      <div class="box-card box-flex" v-show="textData.length>0">
        <div
          v-for="item in textData"
          :key="item.text"
          class="text"
        >{{item.txt + `(${item.seconds})`}}</div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: "home",
  data() {
    return {
      tableData: [
        {
          oldVal: "",
          newVal: ""
        }
      ],
      isShowed: false,
      textarea: "",
      acupointData: [],
      doubleData: [
        {
          value: "1",
          label: "1"
        },
        {
          value: "1.1",
          label: "1.1"
        },
        {
          value: "1.2",
          label: "1.2"
        },
        {
          value: "1.3",
          label: "1.3"
        }
      ],
      acupoint: "",
      double: "1.3",
      mp3: "",
      seconds: Number,
      video: "",
      textData: [],
      videoShow: false,
      changeData: []
    };
  },
  beforeMount() {
    this.getacupointDataList();
  },
  watch: {
    video() {
      this.$nextTick(() => {
        this.videoShow = true;
      });
    }
  },
  methods: {
    getacupointDataList() {
      this.$ajax({
        url: "http://172.16.93.105:2345/init",
        method: "get"
      }).then(res => {
        console.log(res.data);
        if (res.data) {
          this.acupointData = res.data.options;
        }
      });
    },
    getaInfoDataList(acupoint, double) {
      let that = this;
      this.$ajax({
        url: "http://172.16.93.105:2345/acuGet",
        method: "get",
        params: {
          acu: acupoint,
          speed: double
        }
      }).then(res => {
        if (res.data.code == 0) {
          this.videoShow = false;
          this.mp3 = res.data.mp3;
          this.seconds = res.data.seconds;
          this.textarea = res.data.text;
          this.video = res.data.video;
          this.textData.push({ txt: this.acupoint, seconds: this.seconds });
        }
      });
    },
    bindGetacupointVal(acupoint) {
      this.acupoint = acupoint;
    },
    bindGetdoubleVal(double) {
      this.double = double;
    },
    bindStart() {
      if (this.acupoint !== "" && this.double !== "") {
        this.getaInfoDataList(this.acupoint, this.double);
      } else {
        this.$message.error("没选择你提交啥子");
      }
    },
    bindAddtableRow() {
      this.tableData.push({
        oldVal: "",
        newVal: ""
      });
    },
    bindremovetableRow() {
      this.tableData.shift({
        oldVal: "",
        newVal: ""
      });
    },
    handleInputBlur(index, data) {
      this.changeData[index] = { oldval: data.oldVal, newval: data.newVal };
      console.log(this.changeData);
    },
    bindChangeAudio() {
      this.$confirm("是否提交音频", "提交确认", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "info"
      })
        .then(() => {
          this.$ajax
            .post("http://172.16.93.105:2345/audioCreate", {
              acu: this.acupoint,
              text: this.textarea,
              change: this.changeData
            })
            .then(res => {
              console.log(res);
              if (res.data.code === 0) {
                this.$message({
                  type: "success",
                  message: res.data.msg
                });
              }
            })
            .catch(err => {
              this.$message({
                type: "error",
                message: err
              });
            });
        })
        .catch(err => {
          this.$message({
            type: "error",
            message: "操作已取消"
          });
        });
    }
  }
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.wrap {
  width: 1200px;
  margin: 0 auto;
  display: -webkit-flex;
  flex-flow: row nowrap;
  justify-content: center;
  padding: 20px 0;
}
.acupointLeft {
  display: -webkit-flex;
  flex-direction: column;
}
.acupoint-txt {
  margin-top: 7px;
  margin-bottom: 20px;
  font-size: 20px;
}
.acupointRight {
  max-width: 300px;
  margin-left: 30px;
  display: -webkit-flex;
  flex-direction: column;
}
.select-box {
  margin-top: 20px;
}
.acupoint-video {
  border: 1px solid red;
}
.box-flex {
  margin: 20px 0;
  padding: 8px;
  display: -webkit-flex;
  flex-flow: row wrap;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  border: 1px solid #ebeef5;
}
.text {
  margin: 10px;
}
.video {
  max-height: 180px;
}
.audioBox {
  margin-top: 20px;
}
.audio {
  margin-left: 20px;
}
.audioBox {
  display: -webkit-flex;
  flex-flow: ropw nowrap;
  align-items: center;
  justify-content: space-between;
}
.btn{
  margin-bottom: 20px;
}
</style>
