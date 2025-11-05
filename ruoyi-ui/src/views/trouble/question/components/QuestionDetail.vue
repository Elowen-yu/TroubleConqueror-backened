<template>
  <div class="question-detail-overlay" @click.self="handleClose" style="z-index: 2000;">
    <div class="question-detail-dialog">
      <!-- 头部 -->
      <div class="detail-header">
        <div class="header-info">
          <el-tag
            :type="getTypeTagType(question.questionType)"
            size="medium"
            class="header-tag"
          >
            {{ question.questionType || '未区分' }}
          </el-tag>
          <span class="header-time">{{ formatTime(question.createTime) }}</span>
        </div>
        <h2 class="header-title">题目详情</h2>
        <div class="header-actions">
          <el-button
            type="primary"
            size="small"
            icon="el-icon-edit"
            @click="handleEdit"
          >
            编辑错题
          </el-button>
          <el-button
            icon="el-icon-close"
            circle
            class="close-btn"
            @click="handleClose"
          />
        </div>
      </div>

      <!-- 内容区域 -->
      <div class="detail-content">
        <!-- 题目内容 -->
        <div class="detail-section">
          <div class="section-header">
            <div class="section-indicator blue"></div>
            <h3 class="section-title">题目内容</h3>
          </div>
          <div class="section-content question-content">
            {{ question.questionContent }}
          </div>
        </div>

        <!-- 题目图片 -->
        <div v-if="hasImages" class="detail-section">
          <div class="section-header">
            <div class="section-indicator purple"></div>
            <h3 class="section-title">题目图片</h3>
          </div>
          <div class="image-viewer">
            <el-carousel
              :interval="0"
              :arrow="imageArray.length > 1 ? 'hover' : 'never'"
              :height="carouselHeight"
              indicator-position="outside"
            >
              <el-carousel-item v-for="(img, idx) in imageArray" :key="idx">
                <div class="image-wrapper">
                  <el-image
                    :src="getImageUrl(img)"
                    fit="contain"
                    class="detail-image"
                    :preview-src-list="previewList"
                    :initial-index="idx"
                    :lazy="true"
                  />
                </div>
              </el-carousel-item>
            </el-carousel>
          </div>
        </div>

        <!-- 答案解析 -->
        <div class="detail-section">
          <div class="section-header">
            <div class="section-indicator green"></div>
            <h3 class="section-title">答案解析</h3>
          </div>
          <div class="section-content answer-content">
            {{ question.answerContent || '暂无答案' }}
          </div>
        </div>

        <!-- 答案图片 -->
        <div v-if="hasAnswerImages" class="detail-section">
          <div class="section-header">
            <div class="section-indicator orange"></div>
            <h3 class="section-title">答案图片</h3>
          </div>
          <div class="image-viewer">
            <el-carousel
              :interval="0"
              :arrow="answerImageArray.length > 1 ? 'hover' : 'never'"
              :height="carouselHeight"
              indicator-position="outside"
            >
              <el-carousel-item v-for="(img, idx) in answerImageArray" :key="idx">
                <div class="image-wrapper">
                  <el-image
                    :src="getImageUrl(img)"
                    fit="contain"
                    class="detail-image"
                    :preview-src-list="answerPreviewList"
                    :initial-index="idx"
                    :lazy="true"
                  />
                </div>
              </el-carousel-item>
            </el-carousel>
          </div>
        </div>

        <!-- 知识点标签 -->
        <div v-if="question.tags" class="detail-section">
          <div class="section-header">
            <div class="section-indicator orange"></div>
            <h3 class="section-title">知识点标签</h3>
          </div>
          <div class="tags-container">
            <el-tag
              v-for="(tag, idx) in tagArray"
              :key="idx"
              type="warning"
              size="medium"
              class="detail-tag"
            >
              <i class="el-icon-collection-tag"></i>
              {{ tag }}
            </el-tag>
          </div>
        </div>
      </div>
    </div>

    <!-- 编辑对话框 -->
    <question-edit-dialog
      ref="editDialog"
      :question-id="question.questionId"
      @success="handleEditSuccess"
    />
  </div>
</template>

<script>
import QuestionEditDialog from "./QuestionEditDialog.vue";

export default {
  name: "QuestionDetail",
  components: {
    QuestionEditDialog
  },
  props: {
    question: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      editDialogVisible: false,
      carouselHeight: '400px'
    };
  },
  mounted() {
    this.updateCarouselHeight();
    window.addEventListener('resize', this.updateCarouselHeight);
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.updateCarouselHeight);
  },
  computed: {
    hasImages() {
      return this.question.questionImages && this.question.questionImages.trim() !== '';
    },
    imageArray() {
      if (!this.hasImages) return [];
      return this.question.questionImages.split(',').filter(img => img.trim());
    },
    previewList() {
      return this.imageArray.map(img => this.getImageUrl(img));
    },
    hasAnswerImages() {
      return this.question.answerImages && this.question.answerImages.trim() !== '';
    },
    answerImageArray() {
      if (!this.hasAnswerImages) return [];
      return this.question.answerImages.split(',').filter(img => img.trim());
    },
    answerPreviewList() {
      return this.answerImageArray.map(img => this.getImageUrl(img));
    },
    tagArray() {
      if (!this.question.tags) return [];
      return this.question.tags.split(',').filter(tag => tag.trim());
    }
  },
  methods: {
    updateCarouselHeight() {
      // 移动端使用自适应高度，桌面端使用固定高度
      if (window.innerWidth <= 767) {
        // 移动端：使用视口高度的50%，但最小200px，最大500px
        const viewportHeight = window.innerHeight;
        const calculatedHeight = Math.min(Math.max(viewportHeight * 0.5, 200), 500);
        this.carouselHeight = `${Math.floor(calculatedHeight)}px`;
      } else {
        this.carouselHeight = '400px';
      }
    },
    handleClose() {
      this.$emit('close');
    },
    handleEdit() {
      this.editDialogVisible = true;
      this.$nextTick(() => {
        this.$refs.editDialog.open();
      });
    },
    handleEditSuccess() {
      this.$emit('refresh');
      this.$message.success('修改成功');
    },
    getImageUrl(imagePath) {
      if (!imagePath) return '';
      if (imagePath.startsWith('http')) {
        return imagePath;
      }
      const baseUrl = process.env.VUE_APP_BASE_API;
      return baseUrl + (imagePath.startsWith('/') ? imagePath : '/' + imagePath);
    },
    formatTime(time) {
      if (!time) return '';
      return this.parseTime(time, '{y}-{m}-{d} {h}:{i}:{s}');
    },
    getTypeTagType(type) {
      const typeMap = {
        '选择题': 'success',
        '填空题': 'warning',
        '解答题': 'danger',
        '未区分': 'info'
      };
      return typeMap[type] || 'info';
    },
    parseTime(time, pattern) {
      if (!time) return '';
      const date = new Date(time);
      const formatObj = {
        y: date.getFullYear(),
        m: date.getMonth() + 1,
        d: date.getDate(),
        h: date.getHours(),
        i: date.getMinutes(),
        s: date.getSeconds()
      };
      const format = pattern || '{y}-{m}-{d} {h}:{i}:{s}';
      return format.replace(/{(\w+)}/g, (result, key) => {
        const value = formatObj[key];
        return value < 10 ? '0' + value : value;
      });
    }
  }
};
</script>

<style scoped>
.question-detail-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  padding: 20px;
  animation: fadeIn 0.2s ease-out;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.question-detail-dialog {
  background: white;
  border-radius: 24px;
  max-width: 900px;
  width: 100%;
  max-height: 90vh;
  overflow: hidden;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  animation: slideUp 0.3s ease-out;
  display: flex;
  flex-direction: column;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.detail-header {
  background: linear-gradient(135deg, #409eff 0%, #67c23a 100%);
  padding: 24px;
  color: white;
  position: relative;
  flex-shrink: 0;
  padding-top: calc(env(safe-area-inset-top, 0px) + 24px);
}

.header-info {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 12px;
}

.header-tag {
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  border: none;
  color: white;
}

.header-time {
  font-size: 14px;
  opacity: 0.9;
}

.header-title {
  font-size: 24px;
  font-weight: bold;
  margin: 0;
  color: white;
}

.header-actions {
  position: absolute;
  top: 16px;
  right: 16px;
  display: flex;
  align-items: center;
  gap: 8px;
  z-index: 10;
}

.header-actions >>> .el-button {
  padding: 8px 12px;
  font-size: 13px;
  min-width: 44px;
  min-height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.header-actions >>> .el-button--small {
  padding: 6px 12px;
  font-size: 12px;
  min-width: 40px;
  min-height: 40px;
}

.header-actions >>> .el-button--circle {
  width: 44px;
  height: 44px;
  padding: 0;
}

.close-btn {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  color: white;
}

.close-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}

.detail-content {
  padding: 32px;
  overflow-y: auto;
  flex: 1;
  padding-bottom: calc(32px + env(safe-area-inset-bottom, 0px) + 20px);
  -webkit-overflow-scrolling: touch;
}

.detail-section {
  margin-bottom: 32px;
}

.detail-section:last-child {
  margin-bottom: 0;
  padding-bottom: 20px;
}

.section-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
}

.section-indicator {
  width: 4px;
  height: 24px;
  border-radius: 2px;
}

.section-indicator.blue {
  background: #409eff;
}

.section-indicator.purple {
  background: #9c27b0;
}

.section-indicator.green {
  background: #67c23a;
}

.section-indicator.orange {
  background: #e6a23c;
}

.section-title {
  font-size: 18px;
  font-weight: 600;
  color: #303133;
  margin: 0;
}

.section-content {
  background: #f5f7fa;
  border-radius: 12px;
  padding: 20px;
  line-height: 1.8;
  white-space: pre-wrap;
  word-break: break-word;
}

.question-content {
  font-size: 16px;
  color: #303133;
}

.answer-content {
  background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%);
  border: 1px solid #bae6fd;
  font-size: 15px;
  color: #303133;
}

.image-viewer {
  border-radius: 12px;
  overflow: hidden;
  background: #1a1a1a;
}

.image-wrapper {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 200px;
  padding: 10px;
  box-sizing: border-box;
}

.detail-image {
  width: 100%;
  max-width: 100%;
  height: auto;
  max-height: 100%;
  display: block;
  object-fit: contain;
}

.detail-image >>> .el-image__inner {
  width: 100% !important;
  height: auto !important;
  max-height: 100% !important;
  object-fit: contain !important;
}

.image-viewer >>> .el-carousel__container {
  height: auto !important;
  min-height: 200px;
}

.image-viewer >>> .el-carousel__item {
  height: auto !important;
  display: flex;
  align-items: center;
  justify-content: center;
}

.tags-container {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}

.detail-tag {
  padding: 8px 16px;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 6px;
}

@media (max-width: 767px) {
  .question-detail-overlay {
    padding: 0;
    align-items: flex-start;
    padding-top: env(safe-area-inset-top, 0px);
  }

  .question-detail-dialog {
    max-height: 100vh;
    border-radius: 0;
    width: 100%;
    max-width: 100%;
    margin: 0;
    display: flex;
    flex-direction: column;
  }

  .detail-header {
    padding: 60px 16px 16px 16px;
    padding-top: calc(env(safe-area-inset-top, 0px) + 60px);
    position: sticky;
    top: 0;
    z-index: 100;
  }

  .header-info {
    margin-bottom: 8px;
  }

  .header-title {
    font-size: 18px;
    margin-bottom: 0;
  }

  .header-actions {
    top: 12px;
    right: 12px;
    flex-direction: column;
    gap: 6px;
  }

  .header-actions >>> .el-button {
    padding: 8px 12px;
    font-size: 12px;
    min-width: 44px;
    min-height: 44px;
  }

  .header-actions >>> .el-button--small {
    padding: 6px 10px;
    font-size: 11px;
    min-width: 40px;
    min-height: 40px;
  }

  .header-actions >>> .el-button--circle {
    width: 44px;
    height: 44px;
    padding: 0;
  }

  .detail-content {
    padding: 16px;
    padding-bottom: calc(16px + env(safe-area-inset-bottom, 0px) + 40px);
    flex: 1;
    overflow-y: auto;
  }

  .detail-section {
    margin-bottom: 24px;
  }

  .detail-section:last-child {
    padding-bottom: 40px;
  }

  .section-header {
    margin-bottom: 12px;
  }

  .section-title {
    font-size: 16px;
  }

  .section-content {
    padding: 14px;
    font-size: 14px;
  }

  .image-viewer {
    border-radius: 8px;
  }

  .image-wrapper {
    min-height: 150px;
    padding: 8px;
  }

  .image-viewer >>> .el-carousel__container {
    min-height: 150px;
  }

  .tags-container {
    gap: 8px;
    padding-bottom: 10px;
  }

  .detail-tag {
    padding: 6px 12px;
    font-size: 12px;
  }
}
</style>
