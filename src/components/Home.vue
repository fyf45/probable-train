<template>
  <div class="wrap">
    <div class="acupointLeft">
      <div class="acupoint-txt" v-show="textarea">{{textarea}}</div>
      <audio controls  class="audio" v-if="videoShow">
        <source :src="mp3" ref="audioname" type="audio/ogg" />
        <source :src="mp3" ref="audioname" type="audio/mpeg" />
      </audio>
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
      videoShow:false
    };
  },
  beforeMount() {
    this.getacupointDataList();
  },
  watch:{
    video(){
     this.$nextTick(()=>{
       this.videoShow = true;
     }) 
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
      this
        .$ajax({
          url: "http://172.16.93.105:2345/acuGet",
          method: "get",
          params: {
            acu: acupoint,
            speed: double
          }
        })
        .then(res => {
          console.log(res.data);
          if (res.data.code == 0) {
            console.log(res.data)
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
    }
  }
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.wrap {
  width: 1200px;
  height: 100vh;
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
  width: 500px;
  min-height: 314px;
  margin-bottom: 20px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  border: 1px solid #ebeef5;
  padding: 8px;
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
</style>
