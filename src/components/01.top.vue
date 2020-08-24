<template>
  <div class="top-container">
    <div class="left-box">
      <div class="icon-wrapper">
        <span class="el-tag el-tag--light" plain>R</span>
        <span class="el-tag el-tag--success el-tag--light">F</span>
        <span class="el-tag el-tag--info el-tag--light">的</span>
        <span class="el-tag el-tag--warning el-tag--light">音</span>
        <span class="el-tag el-tag--danger el-tag--light">乐</span>
    
      </div>
      <div class="history-wrapper">
         <span class="iconfont icon-arrow-lift" @click="back"></span>
        <span class="iconfont icon-arrow-right" @click="forward"></span>
      </div>
    </div>
    <div class="right-box">
      <div class="el-input el-input--small el-input--prefix">
        <!--搜索-->
        <input
          type="text"
          autocomplete="off"
          placeholder="搜索"
          class="el-input__inner"
          @keyup.enter="toResult"
          v-model="inputValue"
        />
        <span class="el-input__prefix">
          <i class="el-input__icon el-icon-search"></i>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'top',
    data() {
      return {
        //搜索内容
        inputValue: ''
      }
    },
    methods:{
      toResult(){
        //非空判断
        if(this.inputValue=='')
        {
          //提示
          this.$message.warning('请输入内容')
        }else{
          //搜索
          this.$router.push('/result?q='+this.inputValue)
          	this.$router.go(0)

        }
      },
      back() {
      this.$router.back()
    },
    forward() {
      this.$router.forward()
    }
    }
  }
</script>

<style scoped></style>
