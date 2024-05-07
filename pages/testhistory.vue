<template>
  <div>
    <h1>Your Test History</h1>
    <ul>
      <li v-for="test in tests" :key="test.test_id">
        Test on: {{ new Date(test.test_date).toLocaleDateString() }} - Score: {{ test.score }}/{{ test.total_possible_score }}
        <p>Details:</p>
        <ul>
          <li v-for="answer in test.answers" :key="answer.id">
            Question: {{ answer.question.question_text }}
            Your Answer: {{ answer.selected_answer.answer_text }}
            Correct: {{ answer.is_correct ? 'Yes' : 'No' }}
          </li>
        </ul>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const tests = ref([])

const fetchTestHistory = async () => {
  try {
    const response = await axios.get('/api/test-history/')
    tests.value = response.data
  } catch (error) {
    console.error("Error fetching test history:", error)
  }
}

onMounted(fetchTestHistory)
</script>
