<style scoped>
.back-character-edit-wrap {
  height: 100%;
  padding: 15px;
  overflow: auto;
}
.breadcrumb {
  margin-bottom: 60px;
}

.back-character-form {
  width: 600px;
  margin: 0 auto;
}
</style>

<template>
  <section class="back-character-edit-wrap">
    <el-breadcrumb separator="/" class="breadcrumb">
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>后台管理</el-breadcrumb-item>
      <el-breadcrumb-item :to="{path: 'backcharacterlist'}">角色列表</el-breadcrumb-item>
      <el-breadcrumb-item>编辑</el-breadcrumb-item>
    </el-breadcrumb>

    <div class="back-character-form">
      <el-form label-position="top" label-width="80px" :model="formCharacter" :rules="rules" ref="formCharacter">
        <el-form-item label="角色名称" prop="name">
          <el-input v-model="formCharacter.name" placeholder="请输入角色名称"></el-input>
        </el-form-item>
        <el-form-item label="显示名称" prop="display_name">
          <el-input v-model="formCharacter.display_name" placeholder="请输入显示名称"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="description">
          <el-input v-model="formCharacter.description" placeholder="请输入角色描述"></el-input>
        </el-form-item>
        <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">全选</el-checkbox>
        <div style="margin: 15px 0;"></div>
        <el-checkbox-group v-model="formCharacter.pres" @change="handleCheckedItemChange">
          <el-checkbox v-for="authority in authorities" :label="authority.id" :key="authority.id" style="margin: 10px;">{{authority.display_name}}</el-checkbox>
        </el-checkbox-group>
        <el-form-item>
          <el-button type="primary" @click="submitForm('formCharacter')" style="width: 100%;margin-top: 10px;">确定</el-button>
        </el-form-item>
      </el-form>
    </div>
        
  </section>
</template>

<script>
export default {
  data() {
    return {
      authorities: [],
      isIndeterminate: true,
      checkAll: true,
      authoritiesArr: [],

      rules: {
        name: [{ required: true, message: "权限角色名称不能为空" }],
        display_name: [{ required: true, message: "权限角色显示名称不能为空" }]
      },

      formCharacter: {
        name: "",
        display_name: "",
        description: "",
        pres: []
      }
    };
  },
  created() {
    if (sessionStorage.authorities) {
      this.authorities = JSON.parse(sessionStorage.authorities);
    } else {
      this.$api.getPermission(res => {
        this.authorities = res.data.data;
        sessionStorage.authorities = JSON.stringify(res.data.data);
      });
    }

    //循环填充 id， 为全选做准备
    let arr = [];
    for (let i = 0; i < this.authorities.length; i++) {
      arr.push(this.authorities[i].id);
    }
    this.authoritiesArr = arr;
    console.log(this.authoritiesArr);

    //当从编辑进去时
    // const character = this.$route.params.character;
    // if (character) {
    //   this.formCharacter = character;
    //   this.formCharacter.pres = character.perms.reduce((arr, value) => {
    //     return arr.concat(value.id);
    //   }, []);
    // }
  },
  methods: {
    /*
    * 全选
    */
    handleCheckAllChange(event) {
      this.formCharacter.pres = event.target.checked ? this.authoritiesArr : [];
      this.isIndeterminate = false;
    },

    /*
    * 监听多选
    */
    handleCheckedItemChange(value) {
      let checkedCount = value.length;
      this.checkAll = checkedCount === this.authorities.length;
      this.isIndeterminate =
        checkedCount > 0 && checkedCount < this.authorities.length;
    },

    /*
    * 表单提交
    */
    submitForm(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.$api.addRole(this.formCharacter, res => {
            this.$message({
              type: "success",
              message: "保存成功",
              showClose: true
            });
            this.$router.push("/backcharacterlist");
          });
        } else {
          this.$message({
            type: "warning",
            message: "信息填写不正确",
            showClose: true
          });
          return false;
        }
      });
    }
  }
};
</script>
