<template>
  <div class="container">
    <!-- 左边部分 -->
    <div class="left">
      <img src="../../../assets/logo.png" alt="Left Image" class="left-image">
    </div>

    <!-- 右边部分 -->
    <div class="right">
      <!-- 右上部分 -->
      <div class="right-top">
        <!-- 右上部分的上 -->
        <div class="right-top-top">
          <span @click="showCoursePackageList" class="course-package-span">课程包列表 ></span>
          <span class="course-package-title">{{ coursePack.title }}</span>
        </div>
        <!-- 右上部分的下 -->
        <div class="right-top-bottom">
          <!-- 右上部分的下左 -->
          <div class="right-top-bottom-left">
            <div class="course-package-title-large">{{ coursePack.title }}</div>
            <div class="course-package-description">{{ coursePack.description }}</div>
          </div>
          <!-- 右上部分的下右 -->
          <div class="right-top-bottom-right">
<!--            <button @click="PlayItUp" class="edit-button">下架</button>-->
<!--            <button @click="PlayItUp" class="edit-button">更新</button>-->
            <button @click="PlayItUp" class="edit-button">玩起来</button>
<!--            <button @click="PlayItUp" class="edit-button">发布</button>-->
            <button @click="showEditDialog" class="edit-button">编辑</button>
            <button @click="showCreateCourseDialog" class="edit-button">创建课程</button>
          </div>
        </div>
      </div>

      <!-- 右下部分 -->
      <div class="right-bottom">
        <div class="course-list" v-for="(course, index) in courseList" :key="index">
          <span class="course-title">{{ course.title }}</span>
          <!-- 编辑按钮 -->
          <el-button class="edit-course-button" @click="handleEdit(course.id)">
            <i class="el-icon-edit"></i> 编辑
          </el-button>
          <!-- 'V' 下拉框 -->
          <el-dropdown @command="handleAction(course.id, $event)">
            <el-button class="edit-course-button">
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

    <!-- 编辑对话框 -->
    <el-dialog
        title="编辑课程包"
        :visible.sync="showDialog"
        :modal="false"
        width="35%" class="dialog dark-dialog">
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
        <el-button @click="showDialog = false" class="dialog-button">取 消</el-button>
        <el-button type="primary" @click="updateCoursePackage" class="dialog-button confirm-button">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 创建课程对话框 -->
    <el-dialog
        title="创建课程"
        :visible.sync="createCourseDialogVisible"
        :modal="false"
        width="35%" class="dialog dark-dialog">
      <el-form :model="newCourse">
        <el-form-item label="标题">
          <el-input v-model="newCourse.title"></el-input>
        </el-form-item>
        <el-form-item label="描述">
          <el-input v-model="newCourse.description"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="createCourseDialogVisible = false" class="dialog-button">取 消</el-button>
        <el-button type="primary" @click="confirmCreate" class="dialog-button confirm-button">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loading: false,
      showDialog: false,
      editDialogVisible: false,
      createCourseDialogVisible: false,
      editTitle: '',
      editDescription: '',
      createTitle: '',
      createDescription: '',
      courseList: [],
      coursePack:[],
      newCoursePackage: {
        title: '',
        description: '',
        cover: ''
      },
      newCourse: {
        title: '',
        description: ''
      },
      fileList: [],
    };
  },
  created() {
    this.getData();
    this.getOneCoursePackData();
  },
  methods: {
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
    updateCoursePackage() {
      var coursePackId = this.$route.params.coursePackId;
      const newCoursePack = {
        id: coursePackId,
        title: this.newCoursePackage.title,
        description: this.newCoursePackage.description,
        cover: this.newCoursePackage.cover
      };
      //this.coursePackages.push(newCourse);
      this.showDialog = false;
      this.newCoursePackage = { title: '', description: '', cover: '' };
      this.fileList = [];

      //this.query.token = localStorage.getItem("token");
      this.axios.put('/editor/course-pack', newCoursePack).then((res) => {
        console.log(res);
        if (res.data.code === "0"){
          this.$message({message: '更新成功', type: 'success'});
          this.getData();
          this.getOneCoursePackData();
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
    },
    getData() {
      //this.query.token = localStorage.getItem("token");
      console.log(this.$route.params.coursePackId);
      var coursePackId = this.$route.params.coursePackId;
      this.axios.post('/editor/courses', {
        //params: this.query
        coursePackId: coursePackId
      }).then((res) => {
        console.log(res);
        if (res.data.code === "0"){
          this.courseList = res.data.result
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
    getOneCoursePackData() {
      //this.query.token = localStorage.getItem("token");
      console.log(this.$route.params.coursePackId);
      var coursePackId = this.$route.params.coursePackId;
      this.axios.get('/editor/course-pack'+'?id='+coursePackId, {
        //params: this.query
      }).then((res) => {
        console.log(res);
        if (res.data.code === "0"){
          this.coursePack = res.data.result;
          this.newCoursePackage = res.data.result;
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
    showCoursePackageList() {
      // 显示课程包列表的逻辑
      this.$router.push(`/course-packs`);
    },
    PlayItUp(){
      window.open('https://earthworm.cuixueshe.com', '_blank');
    },
    showEditDialog() {
      this.showDialog = true;
    },
    showCreateCourseDialog() {
      this.createCourseDialogVisible = true;
    },
    handleEdit(courseId) {
      var coursePackId = this.$route.params.coursePackId;
      // 编辑课程的逻辑
      console.log('编辑课程:', coursePackId,courseId);
      this.$router.push(`/course-packs/${coursePackId}/courses/${courseId}`);
    },
    handleAction(courseId,command) {
      console.log(courseId);
      console.log(command);
      if (command === 'delete') {
        this.deleteCourse(courseId);
      }
    },
    deleteCourse(courseId) {
      //this.query.token = localStorage.getItem("token");
      this.axios.delete('/editor/course'+'?id='+courseId, {}).then((res) => {
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

      this.courseList = this.courseList.filter(c => c !== course);
    },
    confirmEdit() {
      // 确认编辑的逻辑
      this.editDialogVisible = false;
    },
    confirmCreate() {
      var coursePackId = this.$route.params.coursePackId;
      const newCourse = {
        coursePackId: coursePackId,
        title: this.newCourse.title,
        description: this.newCourse.description
      };
      //this.coursePackages.push(newCourse);
      this.createCourseDialogVisible = false;
      this.newCourse = { title: '', description: '', cover: '' };

      //this.query.token = localStorage.getItem("token");
      this.axios.post('/editor/course', newCourse).then((res) => {
        console.log(res);
        if (res.data.code === "0"){
          this.$message({message: '创建成功', type: 'success'});
          this.getData();
          this.createCourseDialogVisible = false;
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

      // 确认创建课程的逻辑
      this.createCourseDialogVisible = false;
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      // 处理文件上传的逻辑
    }
  }
};
</script>

<style scoped>
.container {
  display: flex;
  height: 100vh;
  margin: 0;
  padding: 0;
}

.left {
  width: 5%;
  background-color: #111;
  display: flex;
  align-items: flex-start;
  justify-content: flex-start;
  padding: 10px;
}

.left-image {
  width: 50px;
  height: 50px;
}

.right {
  width: 95%;
  display: flex;
  flex-direction: column;
  background-color: #222;
}

.right-top, .right-bottom {
  padding: 20px;
}

.right-top {
  display: flex;
  flex-direction: column;
  background-color: #333;
  color: white;
}

.right-top-top {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.right-top-bottom {
  display: flex;
  justify-content: space-between;
  align-items: center; /* 垂直居中 */
}

.right-top-bottom-left {
  display: inline-block;
  flex-direction: column;
  margin-right: 20px;
}

.right-top-bottom-right {
  display: flex;
  gap: 10px; /* 在按钮之间添加10px的间隔 */
}

.right-top-bottom-right {
  align-items: flex-end;
}

.course-package-button {
  margin-right: 10px;
  background-color: #FFFFFF85;
}

.course-package-span {
  margin-right: 10px;
  color:#FFFFFF85;
}

.course-package-title {
  margin-right: 10px;
  color:#FFFFFFD1;
}

.course-package-title-large {
  font-size: 1.5em;
  margin-bottom: 10px;
}

.course-package-description {
  margin-bottom: 10px;
}

.play-link {
  margin-bottom: 10px;
  color: white;
}

.edit-button, .create-course-button {
  //margin-bottom: 10px;
  padding: 5px 10px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

.right-bottom {
  flex: 1;
  overflow-y: auto;
}

.course-list {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  //background-color: #6366f1;
  background-color: #4CAF50;
  color: white;
  padding: 10px;
}

.course-title {
  flex: 1;
}

.edit-course-button, .action-select {
  margin-left: 10px;
  background-color: #111827;
}

.course-dropdown {
  //background-color: #111827;
}

.dialog {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.dialog-content {
  background-color: #333;
  padding: 20px;
  border-radius: 5px;
  color: white;
  width: 300px;
}

.input-field, .textarea-field {
  width: 100%;
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #555;
  background-color: #444;
  color: white;
}

.upload-area {
  margin-bottom: 10px;
}

.file-input {
  display: none;
}

.upload-text {
  cursor: pointer;
  padding: 10px;
  border: 1px dashed #555;
  text-align: center;
}

.dialog-buttons {
  display: flex;
  justify-content: flex-end;
}

.dialog-button {
  margin-left: 10px;
}

.confirm-button {
  background-color: #00008B;
  color: white;
}
</style>