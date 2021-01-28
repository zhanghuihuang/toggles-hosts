<template>
  <div>
    <el-container style="height: 500px">
      <el-aside width="300px">
        <el-row style="margin-top:10px;padding-left: 10px">
          <el-button size="mini" type="primary" round @click="openDialog">添加</el-button>
        </el-row>
        <el-row v-for="host in hostList" v-bind:key="host.id" style="margin-top:10px;padding-left: 10px">
          <el-col :span="16">
            <label style="margin-right: 5px">{{ host.name }}</label>
            <el-link type="primary" v-show="host.id != 1 && host.id != currentHostId" @click="delHost(host.id)"><i
                class="el-icon-delete"></i></el-link>
          </el-col>
          <el-col :span="8">
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
            :rows="20"
            placeholder="请输入Hosts文件内容"
            v-model="currentHostContent"
            :disabled="currentTextAreaState"
            @change="changeHostContent">
        </el-input>
      </el-main>
    </el-container>
    <el-dialog title="添加Host配置" :visible.sync="dialogFormVisible">
      <el-form :model="form" :rules="rules">
        <el-form-item label="配置名称" label-width="120px" prop="name">
          <el-input v-model="form.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addHost">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
const fs = window.require('fs')
const defaultHostId = 1
const toggleHostConfigPath = "D:\\toggleHost\\toggleHost.json"
const hostPath = "C:\\Windows\\System32\\drivers\\etc\\hosts"
const defaultHostPath = "D:\\toggleHost\\defaultHost"

export default {
  name: "ToggleHost",
  data() {
    let validateName = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入配置名称'));
      } else {
        for (let host of this.hostList) {
          if (host.name === value) {
            callback(new Error('输入的配置名称已存在'));
            break;
          }
        }
      }
    };
    return {
      currentHostId: 0,
      currentHostContent: "",
      currentTextAreaState: true,
      hostList: [],
      dialogFormVisible: false,
      form: {
        name: ''
      },
      rules: {
        name: [
          {required: true, message: '请输入配置名称', trigger: 'blur'},
          {min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur'},
          {validator: validateName, trigger: 'blur'}
        ]
      }
    };
  },
  created() {
    fs.readFile(defaultHostPath, "utf-8", (err, data) => {
      if (err) {
        console.log("读取默认host文件错误", err)
      } else {
        this.hostList = [
          {
            id: defaultHostId,
            name: "默认Host配置",
            content: data,
            switchFlag: false
          }
        ]
      }
    })
  },
  mounted() {
    fs.readFile(toggleHostConfigPath, "utf-8", (err, data) => {
      if (err) {
        console.log("读取配置文件错误", err)
      } else {
        data = JSON.parse(data)
        console.log("读取json配置", typeof data, data)
        data.hostList[0] = this.hostList[0]
        console.log("设置默认host", data)
        Object.assign(this, data)
        for (let host of this.hostList) {
          if (host.id == this.currentHostId) {
            this.currentHostContent = host.content
            host.switchFlag = true
            this.currentTextAreaState = host.id == defaultHostId
            this.writeHostFile()
          }
        }
      }
    })
  },
  methods: {
    openDialog() {
      this.dialogFormVisible = true
    },
    addHost() {
      //添加一个host配置
      let max = 1;
      let defaultHost = null;
      for (let host of this.hostList) {
        max = Math.max(max, host.id)
        if (host.id == defaultHostId) {
          defaultHost = host
        }
      }
      let host = {
        id: max + 1,
        name: this.form.name,
        content: defaultHost.content,
        switchFlag: false
      }
      this.hostList.push(host)
      this.dialogFormVisible = false
      this.form.name = ''
      this.writeToggleHostConfig()
    },
    delHost(id) {
      if (id === this.currentHostId) {
        this.$message({
          showClose: true,
          message: '开启的host配置不能删除',
          type: 'warning'
        });
      }
      let newHostList = []
      for (let host of this.hostList) {
        if (host.id != id || host.id == defaultHostId) {
          newHostList.push(host)
        }
      }
      this.hostList = newHostList;
      this.writeToggleHostConfig()
    },
    changeHostContent() {
      for (let host of this.hostList) {
        if (host.id == this.currentHostId) {
          host.content = this.currentHostContent
          break;
        }
      }
    },
    handleChange(newValue, id) {
      if (newValue) {
        //如果开,则关闭所有其他的host
        for (let host of this.hostList) {
          if (host.id != id) {
            host.switchFlag = false
          } else {
            this.currentHostContent = host.content
            this.currentTextAreaState = host.id == defaultHostId
          }
        }
      } else {
        //如果关,则设置默认为当前host
        for (let host of this.hostList) {
          if (host.id == defaultHostId) {
            host.switchFlag = true
            this.currentHostContent = host.content
            this.currentTextAreaState = true
          }
        }
      }
      console.log(this.currentTextAreaState)
      this.writeToggleHostConfig()
      this.writeHostFile()
    },
    writeToggleHostConfig() {
      let currentData = {
        currentHostId: this.currentHostId,
        currentHostContent: this.currentHostContent,
        currentTextAreaState: this.currentTextAreaState,
        hostList: this.hostList
      }
      fs.writeFile(toggleHostConfigPath, JSON.stringify(currentData), (err) => {
        if (err) {
          console.log("写入配置文件错误", err)
        }
      })
    },
    writeHostFile() {
      fs.writeFile(hostPath, this.currentHostContent, (err) => {
        if (err) {
          console.log("写入配置文件错误", err)
        }
      })
    }
  }
};
</script>

<style>
.el-textarea.is-disabled .el-textarea__inner {
  background-color: white;
  color: black;
}

.el-aside {
  background-color: #d3dce6;
  color: #333;
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
