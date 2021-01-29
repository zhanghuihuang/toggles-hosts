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
            <el-link type="primary" v-show="host.id != defaultHostConfig.id && host.id != currentHostConfig.id"
                     @click="delHost(host.id)"><i
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
            v-model="currentHostConfig.content"
            :disabled="currentHostConfig.id == defaultHostConfig.id"
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
      defaultHostConfig: {
        id: 1,
        name: "默认Host配置",
        content: "",
        switchFlag: false
      },
      currentHostConfig: {},
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
    try {
      let data = fs.readFileSync(defaultHostPath, 'utf-8');
      this.defaultHostConfig.content = data
    } catch (err) {
      console.log("读取默认host文件错误", err)
      this.defaultHostConfig.content = ""
    }
    console.log("created阶段默认的host配置", this.defaultHostConfig)
  },
  mounted() {
    let data
    try {
      data = fs.readFileSync(toggleHostConfigPath, 'utf-8');
      try {
        data = JSON.parse(data)
        console.log("读取json配置", data)
      } catch (parseErr) {
        console.log("解析配置文件错误", parseErr.message, data);
        console.error(parseErr);
        data = {}
      }
    } catch (err) {
      console.log("读取配置文件错误", err)
      data = {}
    }
    console.log("mounted阶段默认的host配置", this.defaultHostConfig)
    if (data.currentHostConfig) {
      data.currentHostConfig = Object.assign(data.currentHostConfig, {switchFlag: true})
    }
    if (!data.currentHostConfig || data.currentHostConfig.id == this.defaultHostConfig.id) {
      data.currentHostConfig = Object.assign({}, this.defaultHostConfig, {switchFlag: true})
    }
    if (!(data.hostList instanceof Array) || data.hostList.length == 0) {
      data.hostList = []
      data.hostList.push(Object.assign({}, this.defaultHostConfig))
      if (data.currentHostConfig.id != this.defaultHostConfig.id) {
        data.hostList.push(Object.assign({}, data.currentHostConfig))
      } else {
        data.hostList[0].switchFlag = true
      }
    } else {
      data.hostList[0] = Object.assign({}, this.defaultHostConfig, {switchFlag: data.currentHostConfig.id == this.defaultHostConfig.id})
    }
    console.log("处理后的host配置", data)
    Object.assign(this, data)
    this.writeToggleHostConfig()
    this.writeHostFile()
  },
  methods: {
    openDialog() {
      this.dialogFormVisible = true
    },
    addHost() {
      //添加一个host配置
      let max = 1;
      for (let host of this.hostList) {
        max = Math.max(max, host.id)
      }
      let host = {
        id: max + 1,
        name: this.form.name,
        content: this.defaultHostConfig.content,
        switchFlag: false
      }
      this.hostList.push(host)
      this.dialogFormVisible = false
      this.form.name = ''
      this.writeToggleHostConfig()
    },
    delHost(id) {
      let message = ''
      if (id == this.currentHostConfig.id) {
        message = "开启的host配置不能删除"
      }
      if (id == this.defaultHostConfig.id) {
        message = "默认的host配置不能删除"
      }
      if (message) {
        this.$message({
          showClose: true,
          message: message,
          type: 'warning'
        });
        return
      }
      let newHostList = []
      for (let host of this.hostList) {
        if (host.id != id) {
          newHostList.push(host)
        }
      }
      this.hostList = newHostList;
      this.writeToggleHostConfig()
    },
    changeHostContent() {
      for (let host of this.hostList) {
        if (host.id == this.currentHostConfig.id) {
          host.content = this.currentHostConfig.content
          console.log("改变内容后,当前host配置", this.currentHostConfig)
          this.writeToggleHostConfig()
          this.writeHostFile()
          break;
        }
      }
    },
    handleChange(newValue, id) {
      if (newValue) {
        //如果开,则关闭所有其他的host
        for (let host of this.hostList) {
          Object.assign(host, {switchFlag: host.id == id})
          if (host.id == id) {
            Object.assign(this.currentHostConfig, host)
          }
        }
      } else {
        //如果关,则设置默认为当前host
        for (let host of this.hostList) {
          Object.assign(host, {switchFlag: host.id == this.defaultHostConfig.id})
          if (host.id == this.defaultHostConfig.id) {
            Object.assign(this.currentHostConfig, host)
          }
        }
      }
      console.log("切换开关后,当前host配置", this.currentHostConfig)
      this.writeToggleHostConfig()
      this.writeHostFile()
    },
    writeToggleHostConfig() {
      let currentData = {
        currentHostConfig: this.currentHostConfig,
        hostList: this.hostList
      }
      fs.writeFile(toggleHostConfigPath, JSON.stringify(currentData), (err) => {
        if (err) {
          console.log("写入配置文件错误", err)
        }
      })
    },
    writeHostFile() {
      console.log("写入host文件", this.currentHostConfig)
      fs.writeFile(hostPath, this.currentHostConfig.content, (err) => {
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
