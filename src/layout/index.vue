<template>
  <el-container style="height: 500px">
    <el-aside width="250px">
      <el-row v-for="host in hostList" v-bind:key="host.id" style="margin-top:10px">
        <el-col :span="12">
          <label>{{ host.name }}</label>
        </el-col>
        <el-col :span="12">
          <el-switch
              v-model="host.switchFlag"
              active-color="#13ce66"
              inactive-color="#ff4949"
              active-text="开"
              inactive-text="关"
              @change="handleChange(host.switchFlag,host.id)">
          </el-switch>
        </el-col>
      </el-row>
    </el-aside>
    <el-main>
      <el-input
          type="textarea"
          :rows="100"
          placeholder="请输入Hosts文件内容"
          v-model="currentHostContent">
      </el-input>
    </el-main>
  </el-container>
</template>

<script>
const defaultHostId = "1"
const fs = require("fs")
const path = require('path')

export default {
  name: "Layout",
  data() {
    return {
      currentHostId: defaultHostId,
      currentHostContent: "",
      hostList: [
        {
          id: 1,
          name: "默认",
          content: "默认",
          switchFlag: true
        },
        {
          id: 2,
          name: "物流商品",
          content: "物流商品",
          switchFlag: false
        },
        {
          id: 3,
          name: "同城APP",
          content: "同城APP",
          switchFlag: false
        }
      ]
    };
  },
  mounted() {
    for (let host of this.hostList) {
      if (host.id == this.currentHostId) {
        this.currentHostContent = host.content
        let basedir = path.resolve("./")
        console.log(basedir)
        var data = fs.readFileSync('D:\\username.name');
        console.log("同步读取: " + data.toString());
      }
    }
  },
  methods: {
    handleChange(newValue, id) {
      if (newValue) {
        //如果开,则关闭所有其他的host
        for (let host of this.hostList) {
          if (host.id != id) {
            host.switchFlag = false
          } else {
            this.currentHostContent = host.content
          }
        }
      } else {
        //如果关,则设置默认为当前host
        for (let host of this.hostList) {
          if (host.id == defaultHostId) {
            host.switchFlag = true
            this.currentHostContent = host.content
          }
        }
      }
    }
  }
};
</script>

<style>

.el-aside {
  background-color: #d3dce6;
  color: #333;
  text-align: center;
}

.el-main {
  background-color: #e9eef3;
  color: #333;
  text-align: center;
}

body > .el-container {
  margin-bottom: 40px;
}
</style>
