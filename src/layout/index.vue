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

    </el-main>
  </el-container>
</template>

<script>

export default {
  name: "Layout",
  data() {
    return {
      currentHostId: "1",
      hostList: [
        {
          id: 1,
          name: "默认",
          content: "",
          switchFlag: true
        },
        {
          id: 2,
          name: "物流商品",
          content: "",
          switchFlag: false
        }
      ]
    };
  },
  methods: {
    handleChange(newValue, id) {
      if (newValue) {
        //如果开,则关闭所有其他的host
      } else {
        //如果关,判断当前
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
