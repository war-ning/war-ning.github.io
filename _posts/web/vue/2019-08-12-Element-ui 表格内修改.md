---
layout: articles
title: Element-ui 表格行内修改
tags:  Vue
author: Warning
key:    web-vue-head-03
aside:
  toc: true
sidebar:
nav: Web
category: [web, vue]
---

今天项目有个需求,是想要在表格内实现行内修改,然后进行提交
找雪姐帮搭了个小页面
扔上面备用吧...

<!--more-->

> 其实很简单就是在行内加了个变量用v-if去显示修改的标签而已
> 这里注意,因为是用的template标签,是不能使用v-show的
> 只能用性能略低的v-if

#### 小拓展:v-if和v-show的区别

- 因为v-if每次在显示和隐藏的时候,会销毁和重新建立DOM,所以在频繁切换的时候,会比较浪费性能
- v-show只是在渲染的时候,加了个display:none 的css样式,所以频换切换的时候并不损失性能.但是初始渲染的时候会略慢


```html
<template>
  <dialogTemplate :dialogInfo="dialoginfo" v-if="addShow" @closeClick="closeDialog">
    <div slot="content" class="content-box" style="width:100%;height:100%;overflow:hidden;">
      <!-- 添加 -->
      <el-form
        inline
        :model="form"
        ref="form"
        class="demo-form-inline">
        <el-form-item label="监区编号：" required>
          <el-input v-model="form.num" placeholder="输入内容"></el-input>
        </el-form-item>
        <el-form-item label="监区名称：" required>
          <el-input v-model="form.name" placeholder="输入内容"></el-input>
        </el-form-item>
        <el-form-item label="监区长" required>
          <el-select v-model="form.value" placeholder="请选择">
            <el-option label="区长1" value="shanghai"></el-option>
            <el-option label="区长2" value="beijing"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm">添加</el-button>
          <el-button type="primary" @click="reset">重置</el-button>
        </el-form-item>
      </el-form>
      <!-- 表格 -->
      <el-table
        :data="tableData"
        border
        height="300"
        style="width: 100%">

        <el-table-column
          prop="num"
          label="监区编号"
          width="150">
        </el-table-column>
        <el-table-column
          label="监区名称">
          <template slot-scope="scope">
            <template v-if="!scope.row.isEdit">{{ scope.row.name }}</template>
            <template v-else>
              <el-input v-model="scope.row.name" placeholder="输入内容"></el-input>
            </template>
          </template>
        </el-table-column>
        <el-table-column
          label="监区长">
          <template slot-scope="scope">
            <template v-if="!scope.row.isEdit">{{ scope.row.value }}</template>
            <template v-else>
              <el-select v-model="scope.row.value" placeholder="请选择">
                <el-option label="区长1" value="shanghai"></el-option>
                <el-option label="区长2" value="beijing"></el-option>
              </el-select>
            </template>
          </template>
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="100">
          <template slot-scope="scope">
            <template v-if="!scope.row.isEdit">
              <el-button @click="handleOpt(scope.row, true)" type="text" size="small">编辑</el-button>
              <el-popconfirm
                title="确定删除吗？"
                @confirm='handleDelete(scope.row)'
              >
                <el-button type="text" size="small" slot="reference">删除</el-button>
              </el-popconfirm>
            </template>
            <template v-if="scope.row.isEdit">
              <el-button @click="handleSubmit(scope.row)" type="text" size="small">确认</el-button>
              <el-button @click="handleOpt(scope.row, false)" type="text" size="small">取消</el-button>
            </template>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div slot="btnBottom">
      <el-button class="btn-2 btn" type="success" @click="closeDialog">关闭</el-button>
    </div>
  </dialogTemplate>
</template>

<script>
import dialogTemplate from "@/components/dialogTemplate";
import mdcontent from "@/views/common/content";

export default {
  components: {
    dialogTemplate,
    mdcontent
  },
  data() {
    return {
      dialoginfo: {
        title: "添加监区",
        isFooter: true,
        width: "80%",
        height: "550px",
        ismiddle: true,
        marginTop: "0%",
        iscloseX: false
      },
      addShow: false,
      // add form
      form: {
        num: '',
        name: '',
        value: ''
      },
      tableData: []
    }
  },
  methods: {
    // 关闭弹窗
    closeDialog() {
      this.addShow = false;
    },
    // 打开弹窗
    open() {
      this.addShow = true;
      this.tableData = [{
        num: '01',
        name: '王小虎',
        value: 'shanghai',
        isEdit: false
      }, {
        num: '02',
        name: '王小虎',
        value: 'shanghai',
        isEdit: false
      }, {
        num: '03',
        name: '王小虎',
        value: '上海',
        isEdit: false
      }, {
        num: '04',
        name: '王小虎',
        value: 'beijing',
        isEdit: false
      }, {
        num: '05-05-01',
        name: '王小虎',
        value: '上海',
        isEdit: false
      }]
    },
    // 重置
    reset() {
      this.form = {
        num: '',
        name: '',
        value: ''
      }
    },
    // 添加
    submitForm() {
      if (!this.form.num) {
        this.$message.console.error('请输入监区编号');
        return;
      }
      if (!this.form.name) {
        this.$message.console.error('请输入监区名称');
        return;
      }
      if (!this.form.value) {
        this.$message.console.error('请选择监区长');
        return;
      }
      console.log('form---', this.form)
    },
    // 编辑/取消
    handleOpt(row, val) {
      console.log('opt--', row)
      row.isEdit = val;
    },
    // 删除
    handleDelete(row) {
      console.log('del---', row)
    },
    // 确认
    handleSubmit(row) {
      console.log('确认--', row)
    }
  }
}
</script>

```
