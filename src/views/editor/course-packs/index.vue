<template>
  <div class="container">
    <div class="left-section">
      <img src="../../../assets/logo.png" alt="Image" class="image" />
    </div>
    <div class="right-section">
      <div class="right-top">
        <div class="title-section">
          <h1 class="title">课程包列表</h1>
          <p class="description">可以创建、编辑和删除课程包。</p>
        </div>
        <button class="create-button" @click="clickShowDialog">创建课程包</button>
      </div>
      <div class="right-bottom">
        <div class="course-list" v-if="coursePacks.length">
          <div class="course-item" v-for="(coursePack, index) in coursePacks" :key="index">
            <img :src="coursePack.cover" alt="Cover" class="course-cover" />
            <div class="course-info">
              <h2 class="course-name">{{ coursePack.name }}</h2>
              <p class="course-description">{{ coursePack.description }}</p>
            </div>
            <div class="actions">
              <!-- 编辑按钮 -->
              <el-button class="edit-course-pack-button" @click="goToEditPage(coursePack.id)">
                <i class="el-icon-edit"></i> 编辑
              </el-button>
              <!-- 'V' 下拉框 -->
              <el-dropdown @command="handleDropdownChange(coursePack.id, $event)">
                <el-button class="edit-course-pack-button">
                  <i class="el-icon-arrow-down el-icon--right"></i>
                </el-button>
                <el-dropdown-menu>
                  <el-dropdown-item command="delete" class="course-dropdown">
                    <i class="el-icon-delete"></i> 删除
                  </el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </div>
          </div>
        </div>
        <div v-else class="no-courses">
          暂无课程包
        </div>
      </div>
    </div>
    <el-dialog
        title="创建课程包"
        :visible.sync="showDialog"
        :modal="false"
        width="35%" class="custom-dialog">
      <el-form :model="newCoursePackage">
        <el-form-item label="标题">
          <el-input v-model="newCoursePackage.title"></el-input>
        </el-form-item>
        <el-form-item label="描述">
          <el-input v-model="newCoursePackage.description"></el-input>
        </el-form-item>
        <el-form-item label="封面图">
          <el-upload
              class="upload-demo"
              action="https://jsonplaceholder.typicode.com/posts/"
              :http-request="handleCoverChange"
              :file-list="fileList"
              list-type="picture">
            <el-button slot="trigger" size="small" type="primary">点击上传</el-button>
<!--            <div slot="tip" class="el-upload__tip">点击或者拖动文件到该区域来上传课程封面图</div>-->
          </el-upload>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="showDialog = false">取 消</el-button>
        <el-button type="primary" @click="createCoursePackage">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'CoursePackageList',
  data() {
    return {
      loading: false,
      showDialog: false,
      newCoursePackage: {
        title: '',
        description: '',
        cover: ''
      },
      fileList: [],
      coursePacks: []
    };
  },
  created() {
    this.getData();
  },
  methods: {
    getData() {
      //this.query.token = localStorage.getItem("token");
      this.axios.get('/editor/course-packs', {
        //params: this.query
      }).then((res) => {
        console.log(res);
        if (res.data.code === "0"){
          this.coursePacks = res.data.result
        }
      }).catch(() => {
        this.loading = false;
      })
    },
    clickShowDialog(){
      this.showDialog = true;
      console.log(this.showDialog);
    },
    goToEditPage(coursePackId) {
      this.$router.push(`/course-packs/${coursePackId}`);
    },
    handleDropdownChange(coursePackId, command) {
      console.log(coursePackId);
      console.log(command);
      if (command === 'delete') {
        this.deleteCoursePackage(coursePackId);
      }
    },
    deleteCoursePackage(coursePackId) {
      //this.query.token = localStorage.getItem("token");
      this.axios.delete('/editor/course-pack'+'?id='+coursePackId, {}).then((res) => {
        console.log(res);
        if (res.data.code === "0"){
          this.$message({message: '删除成功', type: 'success'});
          this.getData();
        }else {
          this.$message(res.data.message);
        }
      }).catch(error => {
        // 关闭加载动画
        this.loadingInstance.close();
        if (error.response && error.response.data && error.response.data.message) {
          this.$message.error(error.response.data.message);
        } else {
          this.$message.error('请求失败，请稍后再试');
        }
      }).finally(
          error => {
            //this.loading = false;
          }
      )

      this.coursePackages = this.coursePackages.filter(course => course.id !== courseId);
    },
    handleCoverChange(options) {
      //this.query.token = localStorage.getItem("token");
      const formData = new FormData();
      formData.append('file', options.file);
      this.axios.post('/editor/course-pack/upload-cover', formData,{headers: {
        'Content-Type': 'multipart/form-data'
      }}).then((res) => {
        console.log(res);
        if (res.data.code === "0"){
          this.$message({message: '上传成功', type: 'success'});
          this.newCoursePackage.cover = res.data.result.filePath;
          console.log(this.newCoursePackage.cover);
        }else {
          this.$message(res.data.message);
        }
      }).catch(error => {
        // 关闭加载动画
        this.loadingInstance.close();
        if (error.response && error.response.data && error.response.data.message) {
          this.$message.error(error.response.data.message);
        } else {
          this.$message.error('请求失败，请稍后再试');
        }
      }).finally(
          error => {
            //this.loading = false;
          }
      )
    },
    createCoursePackage() {
      const newCourse = {
        title: this.newCoursePackage.title,
        description: this.newCoursePackage.description,
        cover: this.newCoursePackage.cover
      };
      //this.coursePackages.push(newCourse);
      this.showDialog = false;
      this.newCoursePackage = { title: '', description: '', cover: '' };
      this.fileList = [];

      //this.query.token = localStorage.getItem("token");
      this.axios.post('/editor/course-pack', newCourse).then((res) => {
        console.log(res);
        if (res.data.code === "0"){
          this.$message({message: '创建成功', type: 'success'});
          this.getData();
          this.showDialog = false;
        }else {
          this.$message(res.data.message);
        }
      }).catch(error => {
        // 关闭加载动画
        this.loadingInstance.close();
        if (error.response && error.response.data && error.response.data.message) {
          this.$message.error(error.response.data.message);
        } else {
          this.$message.error('请求失败，请稍后再试');
        }
      }).finally(
          error => {
            //this.loading = false;
          }
      )
    }
  }
}
</script>

<style scoped>
.container {
  display: flex;
  height: 100vh;
  margin: 0;
  padding: 0;
}

.left-section {
  width: 7%;
  background-color: #1e1e1e; /* 深黑色背景 */
  position: relative;
}

.image {
  position: absolute;
  top: 20px;
  left: 20px;
  width: 50px; /* 根据图片大小调整 */
  height: auto;
}

.right-section {
  width: 95%;
  display: flex;
  flex-direction: column;
  background-color: #2d2d2d; /* 暗黑色背景 */
}

.right-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 20px;
}

.title-section {
  flex: 1;
}

.title {
  font-size: 2.5em;
  color: white;
}

.description {
  font-size: 1.2em;
  color: white;
}

.create-button {
  padding: 10px 20px;
  font-size: 1.2em;
  background-color: #4CAF50; /* 绿色按钮 */
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

.create-button:hover {
  background-color: #45a049; /* 按钮悬停颜色 */
}

.right-bottom {
  flex: 1;
  padding: 20px;
  overflow-y: auto; /* 滚动条 */
}

.course-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.course-item {
  display: flex;
  align-items: center;
  background-color: #3d3d3d;
  padding: 10px;
  border-radius: 5px;
}

.course-cover {
  width: 100px;
  height: auto;
  margin-right: 20px;
}

.course-info {
  flex: 1;
}

.course-name {
  font-size: 1.5em;
  color: white;
}

.course-description {
  font-size: 1.2em;
  color: white;
}

.actions {
  display: flex;
  align-items: center;
  gap: 10px;
}

.edit-button {
  padding: 5px 10px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

.edit-button:hover {
  background-color: #45a049;
}

.edit-course-pack-button, .action-select {
  margin-left: 10px;
  background-color: #111827;
}

.dropdown {
  padding: 5px;
  border-radius: 5px;
}

.no-courses {
  color: white;
  font-size: 1.5em;
  text-align: center;
}
</style>

<style>
.custom-dialog {
  //background-color: #333333; /* 灰黑色背景 */
  color: white; /* 字体颜色为白色 */
  //background-color:  #333333; /* 设置背景颜色 */
  border: 1px solid #ccc; /* 设置边框 */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* 设置阴影 */
}

.custom-dialog .el-dialog__title {
  //color: white; /* 对话框标题颜色为白色 */
}

.custom-dialog .el-form-item__label {
  //color: white; /* 表单标签颜色为白色 */
}

.custom-dialog .el-input__inner {
  //background-color: #444444; /* 输入框背景颜色 */
  //color: white; /* 输入框字体颜色为白色 */
  //border-color: #555555; /* 输入框边框颜色 */
  //width: 80%;
}

.custom-dialog .el-button--primary {
  background-color: #4CAF50; /* 确定按钮背景颜色 */
  border-color: #4CAF50; /* 确定按钮边框颜色 */
}

.custom-dialog .el-button--primary:hover {
  background-color: #45a049; /* 确定按钮悬停背景颜色 */
  border-color: #45a049; /* 确定按钮悬停边框颜色 */
}

.custom-dialog .el-button--default {
  background-color: #555555; /* 取消按钮背景颜色 */
  border-color: #555555; /* 取消按钮边框颜色 */
  color: white; /* 取消按钮字体颜色为白色 */
}

.custom-dialog .el-button--default:hover {
  background-color: #666666; /* 取消按钮悬停背景颜色 */
  border-color: #666666; /* 取消按钮悬停边框颜色 */
}
</style>