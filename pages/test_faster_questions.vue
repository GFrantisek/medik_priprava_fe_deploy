<template>
  <div>
    <h1>Quiz Questions</h1>
    <div v-for="(question, index) in questions" :key="question.question_id">
      <h3>{{ index + 1 }}. {{ question.question_text }}</h3>
      <ul>
        <li v-for="answer in question.answers" :key="answer.answer_id">
          {{ answer.answer_text }} - Correct: {{ answer.is_correct ? 'Yes' : 'No' }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      questions: []
    };
  },
  mounted() {
    this.fetchQuestions();
  },
  methods: {
    fetchQuestions() {
      axios.get('/api/get_test_questions/', {
        params: {
          numQuestions: 100,
          startQuestion: 1,
          endQuestion: 200,
          numAnswers: 4,
          categories: 'Science,Math'  // Example categories
        }
      })
        .then(response => {
          this.questions = response.data.questions;
        })
        .catch(error => {
          console.error("Error fetching questions:", error);
        });
    }
  }
}
</script>
