<template>
  <div class="redactUser">
    <div class="rollback" @click="$router.go(-1)">
      <i class="el-icon-arrow-left" />{{ $t("redactUser.fanhui") }}
    </div>
    <div class="uploading">
      <el-upload
        ref="upload"
        :action="$baseUrl + '/v1/upload/file'"
        list-type="picture-card"
        :auto-upload="true"
        :on-success="uploadSuccess"
      >
        <el-button
          round
          style="
            width: 115px;
            height: 37px;
            border: 1px solid #0066ed;
            border-radius: 18px;
          "
        >
          {{ $t("redactUser.xuanze") }}
        </el-button>
        <div slot="file" slot-scope="{ file }">
          <img class="el-upload-list__item-thumbnail" :src="file.url" alt="" />
        </div>
      </el-upload>
      <div class="preview" />
      <div class="uploading_a">
        {{ $t("redactUser.recommend") }}
      </div>
    </div>

    <!-- 标题简介 -->
    <div class="redactUser-a">
      <p
        style="
          color: #09090a;
          font-size: 30px;
          font-weight: bold;
          margin-bottom: 20px;
        "
      >
        {{ $t("redactUser.Edit") }}
      </p>
      <p
        style="
          width: 504px;
          height: 47px;
          font-size: 17px;
          font-family: Source Han Sans CN;
          font-weight: 400;
          color: #84858b;
        "
      >
        {{ $t("redactUser.EditDescription") }}
      </p>
    </div>
    <!-- 编辑表单 -->
    <el-form
      ref="formLabelAlign"
      label-position="top"
      :model="formLabelAlign"
      :rules="rules"
    >
      <el-form-item :label="$t('redactUser.name')" prop="username">
        <el-input
          v-model="formLabelAlign.username"
          class="el-input-a"
          :placeholder="$t('redactUser.PleaseName')"
        />
      </el-form-item>
      <el-form-item :label="$t('redactUser.URL')">
        <el-input
          v-model="formLabelAlign.short_url"
          width:400px
          class="el-input-a"
          :placeholder="$t('redactUser.PleaseURL')"
        >
          <template slot="prepend">lionnft.com/</template>
        </el-input>
      </el-form-item>
      <el-form-item :label="$t('redactUser.Bio')">
        <el-input
          v-model="formLabelAlign.desc"
          class="el-input-a"
          :placeholder="$t('redactUser.PleaseBio')"
        />
      </el-form-item>
      <el-button
        style="
          width: 180px;
          height: 45px;
          background: #2081e2;
          border-radius: 22px;
          margin-top: 20px;
        "
        type="primary"
        @click="postUserEdit(formLabelAlign)"
      >
        {{ $t("redactUser.Update") }}
      </el-button>
    </el-form>
  </div>
</template>

<script>
import $http from "../../utils/request";

export default {
  name: "RedactUser",
  props: {},
  data() {
    return {
      rules: {
        username: [
          { required: true, message: "名称不可为空", trigger: "blur" },
          {
            min: 2,
            max: 16,
            message: "长度在 2 到 16 个字符",
            trigger: "blur"
          }
        ]
      },
      dialogImageUrl: "",
      dialogVisible: false,
      disabled: false,
      formLabelAlign: {
        username: "",
        short_url: "",
        desc: "",
        address: "",
        cover: "",
        website: "",
        twitter: "",
        pic: ""
      },
      userpicurl: false
    };
  },
  created() {},
  async mounted() {
    this.getUserInfo();
  },

  methods: {
    async getUserInfo() {
      const resp = await $http.get(
        `/v1/account?address=${sessionStorage.getItem("address")}`
      );
      console.log(resp);
      if (resp.code == 200) {
        this.formLabelAlign.address = resp.data.user_address;
        this.formLabelAlign.desc = resp.data.user_desc;
        this.formLabelAlign.username = resp.data.user_name;
        this.formLabelAlign.cover = resp.data.user_cover;
        this.formLabelAlign.pic = resp.data.user_pic;
      } else {
      }
    },

    uploadFile() {
      this.$refs.upload.submit();
      setTimeout(() => {
        this.uploadSuccess();
      }, 1000);
    },

    async uploadSuccess(e) {
      if (e !== undefined) {
        this.formLabelAlign.pic = e.ipfs;
      }
    },

    async postUserEdit(e) {
      this.formLabelAlign.address = sessionStorage.getItem("address");
      this.$refs.formLabelAlign.validate(async valid => {
        if (valid) {
          if (e.pic == "") {
            this.$message({
              message: "头像不可为空",
              type: "warning"
            });
          } else {
            const formLabelAlign = { ...this.formLabelAlign };
            console.log(formLabelAlign);
            const resp = await $http.post("/v1/account/edit", {
              ...formLabelAlign
            });
            if (resp.code == 200) {
              this.$message({
                message: "更新成功",
                type: "success"
              });
              sessionStorage.setItem("userInfo", formLabelAlign.username);
              this.formLabelAlign = {};
              this.$router.push({
                name: "personalCenter",
                params: { address: sessionStorage.getItem("address") }
              });
            } else if (resp.code == 500) {
              this.$message.error("更新失败");
            }
          }
        } else {
          return false;
        }
      });
    }
  }
};
</script>

<style lang="less" scoped>
.preview {
  position: absolute;
  top: -200px;
  left: 7px;
  width: 100px;
  height: 100px;
  border-radius: 50%;
  border: 1px solid #ccc;
}
.uploading_a {
  position: absolute;
  top: -64px;
  left: -12px;
  width: 255px;
  height: 36px;
  font-size: 16px;
  font-family: Source Han Sans CN;
  font-weight: 500;
  color: #84858a;
  line-height: 23px;
}
/deep/.el-upload-list--picture-card .el-upload-list__item {
  position: relative;
  position: absolute;
  top: -200px;
  left: 7px;
  width: 100px;
  height: 100px;
  background: #f6f6f6;
  border-radius: 50%;
  border: 1px solid #f6f6f6;
}
/deep/.el-upload-list--picture-card .el-upload-list__item-thumbnail {
  width: 100%;
  height: 100%;
  position: absolute;
  left: 0;
  top: 0;
  border-radius: 50%;
}
/deep/.el-upload--picture-card {
  background-color: #fff;
  border: 0;
  border-radius: 6px;
  box-sizing: border-box;
  width: 115px;
  height: 37px;
  line-height: 0;
  vertical-align: top;
}
.uploading {
  position: absolute;
  top: 250px;
  left: 630px;
}
.rollback {
  padding: 30px 0 20px 0;
  font-size: 14px;
  cursor: pointer;
  font-family: Source Han Sans CN;
  font-weight: 500;
  color: #333333;
}
.el-form {
  width: 1200px;
  margin: 0 auto;
  margin: 30px 0;
}
/deep/ .el-input__inner {
  border: 0;
}

/deep/ .el-form-item {
  width: 476px;
  border-bottom: 1px solid #ccc;
}
/deep/ .el-form-item__label {
  font-size: 18px;
  color: #09090a;
}
.redactUser {
  position: relative;
  width: 1200px;
  margin: 0 auto;
  // margin-top: 50px;
}
/deep/.el-input-group__prepend {
  border: 0;
  padding: 0;
  color: #09090a;
  background-color: #fff;
  font-size: 18px;
}
</style>
