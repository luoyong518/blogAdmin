<!--
 * @Author: 471826078@qq.com
 * @Date: 2020-05-21 18:06:03
 * @LastEditors: 471826078@qq.com
 * @LastEditTime: 2020-06-08 17:34:40
--> 
<template>
  <div>
    <el-row style="margin-bottom:22px">
      <el-col :span="24" class="artItem">
        <span class="artItemTitle">文章标题：</span>
        <el-Input v-model="name"></el-Input>
      </el-col>
      <el-col :span="24" class="artItem">
        <span class="artItemTitle">文章简介：</span>
        <el-Input v-model="note"></el-Input>
      </el-col>
      <el-col :span="16">
        <el-col :span="24" class="artItem">
          <span class="artItemTitle">标签：</span>
          <el-checkbox-group v-model="labels">
            <el-checkbox :label="item.name" v-for="(item,index) in labelList" :key="index"></el-checkbox>
          </el-checkbox-group>
        </el-col>
        <el-col :span="12" class="artItem">
          <span class="artItemTitle">是否原创：</span>
          <el-radio-group v-model="isAuthor">
            <el-radio :label="1">是</el-radio>
            <el-radio :label="0">否</el-radio>
          </el-radio-group>
        </el-col>
        <el-col :span="12" class="artItem">
          <span class="artItemTitle">是否置顶：</span>
          <el-radio-group v-model="isTop">
            <el-radio :label="1">是</el-radio>
            <el-radio :label="0">否</el-radio>
          </el-radio-group>
        </el-col>
        <el-col :span="12" class="artItem">
          <span class="artItemTitle">直接上架：</span>
          <el-radio-group v-model="isPublish">
            <el-radio :label="1">是</el-radio>
            <el-radio :label="0">否</el-radio>
          </el-radio-group>
        </el-col>
      </el-col>
      <el-col :span="8">
        <el-upload
          class="avatar-uploader"
          action="/"
          :show-file-list="false"
          :on-success="handleAvatarSuccess"
          :before-upload="beforeAvatarUpload"
          :http-request="uploadFile"
        >
          <img v-if="imgUrl" :src="imgUrl" class="avatar" />
          <i v-else class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
      </el-col>
    </el-row>
    <mavon-editor
      v-model="content"
      ref="md"
      :ishljs="true"
      @change="change"
      @imgAdd="$imgAdd"
      @imgDel="$imgDel"
      style="min-height: 600px"
    />

    <button @click="submit">提交</button>
  </div>
</template>
<script>
import axios from "axios";
import { apiQueryLabel } from "@/api/label";
import { apiAddArticle } from "@/api/article";
import { mavonEditor } from "mavon-editor";
import "mavon-editor/dist/css/index.css";
export default {
  components: {
    mavonEditor
  },
  data() {
    return {
      labelList: [],
      name: "",
      note:'',
      labels: [],
      isAuthor: 1,
      isTop: 0,
      isPublish: 0,
      imgUrl: "",
      content: "", // 输入的markdown
      html: "", // 及时转的html
      img_file: {}
    };
  },
  methods: {
    getLabelList() {
      apiQueryLabel().then(res => {
        if (res.isSuccess) {
          this.labelList = res.data;
        } else {
          this.$message({
            type: "error",
            message: res.message
          });
        }
      });
    },
    // 所有操作都会被解析重新渲染
    change(value, render) {
      // render 为 markdown 解析后的结果[html]
      this.html = render;
      console.log(this.html);

      console.log(value);
    },
    // 提交
    submit() {
      console.log(this.TagList);
      console.log(this.html);
      const params = {};
      if (this.name) {
        params.name = this.name;
      } else {
        this.$message({
          type: "warning",
          message: "文章标题不能为空"
        });
        return false;
      }
      if (this.labels.length) {
        params.labels = this.labels;
      } else {
        this.$message({
          type: "warning",
          message: "请至少选择一个标签"
        });
        return false;
      }
      if (this.content) {
        params.content = this.content;
      } else {
        this.$message({
          type: "warning",
          message: "文章内容不能为空"
        });
        return false;
      }
      if (this.imgUrl) {
        params.imgUrl = this.imgUrl;
      } else {
        this.$message({
          type: "warning",
          message: "文章配图不能为空"
        });
        return false;
      }
      params.note = this.note
      params.isAuthor = this.isAuthor;
      params.isTop = this.isTop;
      params.isPublish = this.isPublish;
      params.imgUrl = this.imgUrl;
      params.htmlContent = this.html;

      apiAddArticle(params).then(res => {
        if (res.isSuccess) {
          this.$message({
            type: "success",
            message: res.message
          });
          setTimeout(() => {
            this.$router.replace("/Home/AcrticleList");
          }, 1500);
        } else {
          this.$message({
            type: "error",
            message: res.message
          });
        }
      });
    },
    handleAvatarSuccess() {},
    beforeAvatarUpload() {},

    uploadFile(file) {
      //   return this.$message.error("请配置接口!");
      let params = new FormData();
      params.append("file", file.file); //后台需要传这picture流
      let config = {
        contentType: false,
        processData: false,
        headers: {
          "Content-Type": "multipart/form-data" //后台接收类型是form-data
        }
      };
      //请求接口  可在api.js中自定义
      axios
        .post("api/index/uploadImage", params, config)
        .then(res => {
          if (res.data.code === 1) {
            this.imgUrl = res.data.path;
            this.$message.success("上传成功");
          } else {
            this.$message.error(res.data.retmsg);
          }
        })
        .catch(err => {
          console.log(err);
        });
    },
    // 绑定@imgAdd event
    $imgAdd(pos, $file) {
      // 第一步.将图片上传到服务器.
      var formdata = new FormData();
      formdata.append("file", $file);
      this.img_file[pos] = $file;
      axios({
        url: "api/index/uploadImage",
        method: "post",
        data: formdata,
        headers: { "Content-Type": "multipart/form-data" }
      }).then(res => {
        console.log(res);

        let _res = res.data;
        // 第二步.将返回的url替换到文本原位置![...](0) -> ![...](url)
        this.$refs.md.$img2Url(pos, _res.path);
        console.log( this.$refs.md);
        
      });
    },
    $imgDel(pos) {
      delete this.img_file[pos];
    }
  },
  mounted() {
    this.getLabelList();
  }
};
</script>
<style lang="less" scoped>
.avatar-uploader {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  width: 220px;
  height: 178px;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 220px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 220px;
  height: 178px;
  display: block;
}

.artItem {
  display: flex;
  align-items: center;
  height: 40px;
  margin-bottom: 22px;
  &Title {
    width: 100px;
    text-align: right;
  }
}
</style>