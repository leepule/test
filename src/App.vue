<template>
  <div class="app-container">
    <div class="main-wrapper">
      <!-- 标题 -->
      <div class="header-section">
        <h1 class="main-title">苍穹题库搜索系统</h1>
        <p class="subtitle">搜索题目，查看答案</p>
      </div>

      <!-- 搜索框 -->
      <div class="search-section">
        <div class="search-wrapper">
          <input
            v-model="searchQuery"
            type="text"
            placeholder="输入关键词搜索题目..."
            class="search-input"
            @keyup.enter="updateUrl"
          />
          <svg class="search-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
          </svg>
        </div>
      </div>

      <!-- 统计信息 -->
      <div class="stats-section">
        <div class="stats-container">
          <div class="stat-item">
            <span class="stat-label">单选题:</span>
            <span class="stat-value stat-single">{{ questionBank.singleChoice.length }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">判断题:</span>
            <span class="stat-value stat-judgment">{{ questionBank.judgment.length }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">多选题:</span>
            <span class="stat-value stat-multiple">{{ questionBank.multipleChoice.length }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">搜索结果:</span>
            <span class="stat-value stat-result">{{ filteredQuestions.length }}</span>
          </div>
        </div>
      </div>

      <!-- 题目列表 -->
      <div class="questions-container">
        <div
          v-for="question in filteredQuestions"
          :key="`${question.type}-${question.id}`"
          class="question-card"
        >
          <!-- 题目类型标签 -->
          <div class="question-header">
            <span :class="getTagClass(question.type)" class="question-tag">
              {{ getQuestionTypeLabel(question.type) }}
            </span>
            <span class="question-id">ID: {{ question.id }}</span>
          </div>

          <!-- 题目内容 -->
          <div class="question-content">
            <h3 class="question-title" v-html="highlightText(question.question)"></h3>
            
            <!-- 单选题和多选题选项 -->
            <div v-if="question.type !== 'judgment'" class="options-container">
              <div
                v-for="(option, index) in question.options"
                :key="index"
                :class="getOptionClass(question, index + 1)"
                class="option-item"
              >
                <div class="option-content">
                  <span class="option-letter">{{ String.fromCharCode(65 + index) }}</span>
                  <span class="option-text" v-html="highlightText(option)"></span>
                </div>
              </div>
            </div>

            <!-- 判断题答案 -->
            <div v-else class="judgment-container">
              <div :class="getJudgmentClass(question, true)" class="judgment-option">
                <span class="judgment-text">✓ 正确</span>
              </div>
              <div :class="getJudgmentClass(question, false)" class="judgment-option">
                <span class="judgment-text">✗ 错误</span>
              </div>
            </div>
          </div>

        </div>
      </div>

      <!-- 分页 -->
      <div v-if="totalPages > 1" class="pagination-section">
        <div class="pagination-container">
          <button
            @click="currentPage = Math.max(1, currentPage - 1)"
            :disabled="currentPage === 1"
            class="pagination-btn"
            :class="{ 'pagination-disabled': currentPage === 1 }"
          >
            ← 上一页
          </button>
          
          <div class="pagination-numbers">
            <button
              v-for="page in visiblePages"
              :key="page"
              @click="currentPage = page"
              :class="getPaginationClass(page)"
              class="pagination-number"
            >
              {{ page }}
            </button>
          </div>

          <button
            @click="currentPage = Math.min(totalPages, currentPage + 1)"
            :disabled="currentPage === totalPages"
            class="pagination-btn"
            :class="{ 'pagination-disabled': currentPage === totalPages }"
          >
            下一页 →
          </button>
        </div>
      </div>

      <!-- 无结果提示 -->
      <div v-if="filteredQuestions.length === 0 && searchQuery" class="no-results">
        <div class="no-results-icon">
          <svg fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.172 16.172a4 4 0 015.656 0M9 12h6m-6-4h6m2 5.291A7.962 7.962 0 0112 15c-2.34 0-4.29-1.175-5.5-2.969L3.5 15.5v-2.5h2.5L9.172 9.828A7.962 7.962 0 0112 7c2.34 0 4.29 1.175 5.5 2.969L20.5 6.5v2.5h-2.5l-3.172 3.172z" />
          </svg>
        </div>
        <h3 class="no-results-title">未找到相关题目</h3>
        <p class="no-results-text">请尝试使用其他关键词搜索</p>
      </div>
    </div>
  </div>
</template>

<script>
import timu from './data.json'
export default {
  name: 'QuestionBank',
  data() {
    return {
      searchQuery: '',
      questionBank: timu
    }
  },
  created() {
    // 从URL参数中获取搜索词
    const urlParams = new URLSearchParams(window.location.search)
    this.searchQuery = urlParams.get('q') || ''
  },
  computed: {
    filteredQuestions() {
      if (!this.searchQuery.trim()) {
        return this.allQuestions
      }
      
      const query = this.searchQuery.toLowerCase()
      return this.allQuestions.filter(question => {
        if (question.question.toLowerCase().includes(query)) {
          return true
        }
        
        if (question.options) {
          return question.options.some(option => 
            option.toLowerCase().includes(query)
          )
        }
        
        return false
      })
    },
    
    allQuestions() {
      const questions = []
      
      this.questionBank.singleChoice.forEach(q => {
        questions.push({
          ...q,
          type: 'single'
        })
      })
      
      this.questionBank.judgment.forEach(q => {
        questions.push({
          ...q,
          type: 'judgment'
        })
      })
      
      this.questionBank.multipleChoice.forEach(q => {
        questions.push({
          ...q,
          type: 'multiple'
        })
      })
      
      return questions
    }
  },
  methods: {
    updateUrl() {
      // 更新URL参数
      const url = new URL(window.location)
      url.searchParams.set('q', this.searchQuery)
      window.history.pushState({}, '', url)
    },
    getQuestionTypeLabel(type) {
      const labels = {
        single: '单选题',
        judgment: '判断题',
        multiple: '多选题'
      }
      return labels[type] || '未知'
    },
    
    getTagClass(type) {
      const classes = {
        single: 'tag-single',
        judgment: 'tag-judgment',
        multiple: 'tag-multiple'
      }
      return classes[type] || ''
    },
    
    getOptionClass(question, optionIndex) {
      return this.isCorrectOption(question, optionIndex) ? 'option-correct' : 'option-normal'
    },
    
    getJudgmentClass(question, value) {
      return question.correctAnswer === value ? 'judgment-correct' : 'judgment-normal'
    },
    
    getPaginationClass(page) {
      return page === this.currentPage ? 'pagination-active' : 'pagination-normal'
    },
    
    isCorrectOption(question, optionIndex) {
      if (question.type === 'single') {
        return question.correctAnswer === optionIndex
      } else if (question.type === 'multiple') {
        return question.correctAnswers && question.correctAnswers.includes(optionIndex)
      }
      return false
    },
    
    getCorrectAnswerText(question) {
      if (question.type === 'judgment') {
        return question.correctAnswer ? '正确' : '错误'
      } else if (question.type === 'single') {
        const index = question.correctAnswer - 1
        return `${String.fromCharCode(65 + index)}. ${question.options[index]}`
      } else if (question.type === 'multiple') {
        const correctOptions = question.correctAnswers.map(answerIndex => {
          const index = answerIndex - 1
          return `${String.fromCharCode(65 + index)}`
        })
        return correctOptions.join(', ')
      }
      return ''
    },
    
    highlightText(text) {
      if (!this.searchQuery.trim()) {
        return text
      }
      
      const query = this.searchQuery.trim()
      const regex = new RegExp(`(${query})`, 'gi')
      return text.replace(regex, '<mark class="highlight">$1</mark>')
    }
  }
}
</script>

<style scoped>
/* 基础重置 */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.app-container {
  min-height: 100vh;
  background: #f5f7fa;
}

.main-wrapper {
  margin: 0 auto;
  padding: 1rem;
}

/* 标题部分 */
.header-section {
  text-align: center;
  margin-bottom: 1rem;
}

.main-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #333;
  margin-bottom: 0.5rem;
}

.subtitle {
  color: #666;
  font-size: 1rem;
}

/* 搜索部分 */
.search-section {
  margin-bottom: 1rem;
}

.search-wrapper {
  position: relative;
  max-width: 100%;
  margin: 0 auto;
}

.search-input {
  width: 100%;
  padding: 0.75rem 0.75rem 0.75rem 2.5rem;
  border: 1px solid #ddd;
  border-radius: 0.5rem;
  font-size: 1rem;
  outline: none;
}

.search-icon {
  width: 1.25rem;
  height: 1.25rem;
  left: 0.75rem;
}

/* 统计信息 */
.stats-container {
  display: flex;
  gap: 1rem;
  background: #fff;
  border-radius: 0.5rem;
  padding: 0.75rem;
  margin-bottom: 1rem;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.stat-label {
  font-size: 0.75rem;
  color: #666;
}

.stat-value {
  font-size: 1.1rem;
  font-weight: 600;
}

/* 题目容器 */
.questions-container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.question-card {
  background: #fff;
  border-radius: 0.5rem;
  padding: 1rem;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.question-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
}

.question-tag {
  padding: 0.25rem 0.5rem;
  border-radius: 0.25rem;
  font-size: 0.75rem;
  font-weight: 500;
}

.question-id {
  font-size: 0.75rem;
  color: #666;
}

.question-title {
  font-size: 1rem;
  margin-bottom: 0.75rem;
  line-height: 1.4;
}

/* 选项样式 */
.options-container {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.option-item {
  padding: 0.5rem;
  border-radius: 0.25rem;
  border: 1px solid #eee;
}

.option-correct {
  background: #e6ffed;
  border-color: #28a745;
}

.option-content {
  display: flex;
  align-items: center;
}

.option-letter {
  width: 1.5rem;
  height: 1.5rem;
  margin-right: 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
}

.option-correct .option-letter {
  background: #28a745;
  color: white;
}

/* 判断题样式 */
.judgment-container {
  display: flex;
  gap: 0.5rem;
}

.judgment-option {
  padding: 0.5rem;
  border-radius: 0.25rem;
  border: 1px solid #eee;
}

.judgment-correct {
  background: #e6ffed;
  border-color: #28a745;
}

/* 答案总结 */
.answer-summary {
  margin-top: 0.75rem;
  padding: 0.5rem;
  background: #e6ffed;
  border-radius: 0.25rem;
  font-size: 0.875rem;
}

/* 高亮文本 */
:deep(.highlight) {
  background: #ffeb3b;
  padding: 0.125rem;
  border-radius: 0.125rem;
}

/* 移动端适配 */
@media (max-width: 640px) {
  .stats-container {
    flex-wrap: wrap;
  }
  
  .stat-item {
    flex: 1 0 45%;
    margin-bottom: 0.5rem;
  }
}
</style>