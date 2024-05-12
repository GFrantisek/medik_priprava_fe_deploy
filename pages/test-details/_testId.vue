<template>
  <div class="test-wrapper">
    <aside class="sidebar">
      <div class="button-container">
        <button class="home-button" @click="navigateHome">
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="icon">
            <path stroke-linecap="round" stroke-linejoin="round" d="m2.25 12 8.954-8.955c.44-.439 1.152-.439 1.591 0L21.75 12M4.5 9.75v10.125c0 .621.504 1.125 1.125 1.125H9.75v-4.875c0-.621.504-1.125 1.125-1.125h2.25c.621 0 1.125.504 1.125 1.125V21h4.125c.621 0 1.125-.504 1.125-1.125V9.75M8.25 21h8.25" />
          </svg>
        </button>
        <button class="profile-button" @click="navigateToProfile">
          <svg xmlns="http://www.w3.org/2000/svg" width="22" height="22" fill="currentColor" class="bi bi-person-circle" viewBox="0 0 16 16">
            <path d="M11 6a3 3 0 1 1-6 0 3 3 0 0 1 6 0"/>
            <path fill-rule="evenodd" d="M0 8a8 8 0 1 1 16 0A8 8 0 0 1 0 8m8-7a7 7 0 0 0-5.468 11.37C3.242 11.226 4.805 10 8 10s4.757 1.225 5.468 2.37A7 7 0 0 0 8 1"/>
          </svg>
        </button>
      </div>
      <nav class="question-nav">
        <button v-for="(question, index) in questions"
                :key="question.question_id"
                class="question-nav-item"
                @click="scrollToQuestion(index)">
          Otázka {{ question.question_id }}
          <span v-if="!question.hasIncorrect" class="correct-indicator">✔️</span>
          <span v-else class="incorrect-indicator">❌</span>
        </button>
      </nav>
    </aside>
    <main class="question-content">
      <div v-for="(question, index) in questions" :key="index"
           :class="{'question-block': true, 'incorrect-question': question.hasIncorrect}"
           :ref="'question-' + index">
        <p class="question-text"> (Otázka {{ question.question_id }}) {{ question.question_text }}</p>
        <ul class="answers-list">
          <li v-for="answer in question.answers" :key="answer.answer_id"
              :class="{'answer': true, 'incorrect': answer.is_incorrect}">
            <label>
              <input type="checkbox" :checked="answer.selected" disabled>
              {{ answer.answer_text }}
            </label>
          </li>
        </ul>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  data() {
    return {
      questions: [],
    };
  },
  computed: {
    testId() {
      return this.$route.params.testId;
    }
  },
  methods: {
    navigateHome() {
      this.$router.push('/');
    },
    navigateToProfile() {
      // Redirect to the user profile using a relative path
      window.location.href = '/auth/user';
    },
    fetchTestDetails() {
      const url = `https://medik-cloud-deploy-fast-beta-first-xxtgwkr47a-lm.a.run.app/api/user/${this.$auth.user.id}/test/${this.testId}/answers/new`;
      fetch(url)
        .then(response => {
          if (!response.ok) {
            throw new Error('Failed to fetch data');
          }
          return response.json();
        })
        .then(data => {
          this.questions = data.map(q => {
            const incorrectAnswersIds = JSON.parse(q.incorrect_answers_ids || '[]');
            const userAnswers = JSON.parse(q.user_answers || '{}');
            const hasIncorrect = incorrectAnswersIds.length > 0;
            return {
              question_id: q.question_id,
              question_text: q.question_text || 'No question text available',
              hasIncorrect: hasIncorrect,
              incorrectCount: incorrectAnswersIds.length, // Add this line to store the count of incorrect answers
              answers: q.answers_info
                .filter(a => q.answer_ids.includes(a.answer_id.toString()))
                .map(a => ({
                  answer_id: a.answer_id,
                  answer_text: a.answer_text || 'No answer text',
                  is_incorrect: incorrectAnswersIds.includes(a.answer_id),
                  selected: userAnswers[a.answer_id.toString()] === true || userAnswers[a.answer_id.toString()] === false
                }))
            };
          });
        })

        .catch(error => {
          console.error('Error fetching test details:', error);
        });
    },
    scrollToQuestion(index) {
      this.$nextTick(() => {  // Ensure the DOM has updated
        const questionRef = this.$refs['question-' + index];
        if (questionRef && questionRef[0]) {
          questionRef[0].scrollIntoView({ behavior: 'smooth', block: 'start' });
        } else {
          console.error('Reference to the question not found:', index);
        }
      });
    }
  },
  mounted() {
    this.fetchTestDetails();
  }
}
</script>

<style>

.button-container {
  display: flex;
  align-items: center;
  justify-content: center; /* Centers the buttons horizontally */
  padding: 10px;
}

.home-button, .profile-button {
  padding: 10px;
  background: #7C4DFF;
  color: white;
  border: none;
  margin: 0 5px; /* Adds spacing between buttons */
  cursor: pointer;
}

.home-button:hover, .profile-button:hover {
  background: #6841c9; /* Darker purple on hover */
}

.icon {
  width: 24px; /* Sets a standard size for the icons */
  height: 24px;
}
.correct-indicator {
  color: green;
}

.incorrect-indicator {
  color: red;
}
.answer.incorrect {
  background-color: #f8d7da; /* Light red background for incorrect answers */
  color: #721c24; /* Dark red text color for better readability */
}

.incorrect-question {
  background-color: #f8d7da; /* Light red for incorrect questions */
}
.home-button {
  padding: 10px;
  background: #7C4DFF;
  color: white;
}

.test-wrapper {
  display: flex;
  background: #F4F4F7;
}

.profile-button {
  padding: 10px;
  background: #7C4DFF;
  color: white;
}

.profile-button:hover {
  background: #6841c9; /* Darker purple on hover */
}

.sidebar {
  width: 200px;
  background: #7C4DFF;
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: sticky;
  top: 0; /* Adjust this value based on your header height or desired spacing */
  height: 100vh; /* Optional: Makes the sidebar take full height of the viewport */
  overflow-y: auto; /* Enables scrolling within the sidebar if items exceed the viewport height */
}

.question-nav {
  width: 100%;
}

.question-nav-item {
  width: 100%;
  padding: 10px;
  text-align: left;
  background: transparent;
  border: none;
  color: white;
  cursor: pointer;
}

.question-nav-item:hover,
.question-nav-item:focus {
  background: rgba(255, 255, 255, 0.2);
}

.question-content {
  flex-grow: 1;
  padding: 20px;
  background: #fff;
}

.question-block {
  background: #F8F8FA;
  margin-bottom: 20px;
  padding: 20px;
  border-radius: 8px;
}
.question-text {
  margin-bottom: 20px;
}

.answers-list {
  list-style: none;
  padding: 0;
  margin-top: 0;
}

.answer {
  margin-bottom: 10px;
}

.answer input[type="checkbox"] {
  margin-right: 10px;
}

.answer.correct {
  background-color: #d4edda;
}

.answer.incorrect {
  background-color: #f8d7da;
}

.submit-wrapper {
  display: flex;
  justify-content: center;
  padding: 20px;
}

.submit-button {
  padding: 10px 20px;
  background-color: #7C4DFF;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  margin-right: 5px;
}

.submit-button:hover {
  background-color: #6841c9;
}


.download-pdf-button {
  padding: 10px 20px;
  background-color: #7C4DFF;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.download-pdf-button:hover {
  background-color: #6841c9;
}

.refresh-button {
  padding: 10px 20px;
  background-color: #4CAF50; /* Green background */
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  margin-left: 5px; /* Space it out from other buttons */
}

.refresh-button:hover {
  background-color: #45a049; /* Darker green on hover */
}

</style>
