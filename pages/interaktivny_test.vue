<template>
  <div class="test-wrapper">
    <aside class="sidebar">
      <button class="home-button" @click="navigateHome">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
          <path stroke-linecap="round" stroke-linejoin="round" d="m2.25 12 8.954-8.955c.44-.439 1.152-.439 1.591 0L21.75 12M4.5 9.75v10.125c0 .621.504 1.125 1.125 1.125H9.75v-4.875c0-.621.504-1.125 1.125-1.125h2.25c.621 0 1.125.504 1.125 1.125V21h4.125c.621 0 1.125-.504 1.125-1.125V9.75M8.25 21h8.25" />
        </svg>
      </button>
      <nav class="question-nav">
        <button
          v-for="(question, index) in questions"
          :key="question.question_id"
          class="question-nav-item"
          @click="scrollToQuestion(index)">
          Otázka {{ question.question_id }}
        </button>
      </nav>
    </aside>
    <main class="question-content">
      <div
        v-for="(question, index) in questions"
        :key="index"
        class="question-block"
        :ref="'question-' + index">
        <p class="question-text">{{ question.question_text }}</p>
        <ul class="answers-list">
          <li v-for="answer in question.answers" :key="answer.answer_id"
              :class="{
      'answer': true,
      'incorrect': submissionAttempted && !correctAnswers[index][answer.answer_id]
    }">
            <label>
              <input type="checkbox" :value="answer" @change="selectAnswer(index, answer)"
                     :disabled="submissionAttempted">
              {{ answer.answer_text }}
            </label>
          </li>
        </ul>
      </div>
      <div class="submit-wrapper">
        <button class="submit-button" @click="submitAnswers" :disabled="submissionAttempted">Submit Test</button>
        <button class="download-pdf-button" @click="downloadPdf">Download PDF </button>
      </div>
      <div v-if="submissionAttempted" id="incorrectCountDisplay%">
        <span v-if="points < 0"> Your points: {{ points }} / {{ maxPoints }} 😢 Získal si záporný počet bodov 😢 0%</span>
        <span v-else-if="percentage.toFixed(2) > 80"> Tvoje bodíky: {{ points }} / {{ maxPoints }} ({{ percentage.toFixed(2) }} %) Good job little monke, medicina is here  🧑‍⚕️   </span>
        <span v-else>Your points: {{ points }} / {{ maxPoints }} ({{ percentage.toFixed(2) }} %)</span>
      </div>
      <div>
        <button v-if="submissionAttempted" class="refresh-button" @click="refreshTest">Refresh</button>
      </div>
    </main>
  </div>
</template>

<script>
import {componentsPlugin} from "bootstrap-vue";

export default {
  name: 'InteraktivnyTest',
  data() {
    return {
      questions: [],
      selectedAnswers: {},
      submissionAttempted: false,
      correctAnswers: {},
      points: 0,
      maxPoints: 0,
      percentage: 0,
      incorrectCount: 0,
      incorrectDetails: [],  // New array to store details of incorrect answers
      listofcor: []
    };
  },
  mounted() {
    this.fetchQuestions();
  },
  methods: {
    navigateHome() {
      this.$router.push('/');
    },
    fetchQuestions() {
      const params = this.$route.query;
      const query = new URLSearchParams({
        numQuestions: params.numQuestions || 100,
        startQuestion: params.startQuestion || 1,
        endQuestion: params.endQuestion || 1500,
        numAnswers: params.numAnswers || 4,
        categories: params.categories || ''
      }).toString();

      console.log("Fetching with parameters:", query);

      fetch(`https://medik-cloud-deploy-fast-xxtgwkr47a-lm.a.run.app/api/get_test_questions/?${query}`)
        .then(response => response.json())
        .then(data => {
          // console.log("Data received from API:", data);
          this.questions = data.questions; // Ensure this aligns with the received JSON structure
          console.log(this.questions)
          console.log("NIGGER",this.questions)
          this.maxPoints = this.questions.reduce((sum, question) => sum + question.answers.length, 0);
        })
        .catch(error => {
          // console.error('Fetching questions failed:', error);
        });
    },
    selectAnswer(questionIndex, answer) {

      // Initialize the answer array for the question if it does not exist
      if (!this.selectedAnswers[questionIndex]) {
        this.$set(this.selectedAnswers, questionIndex, []);
      }

      // Check if the answer is already selected
      const currentAnswers = this.selectedAnswers[questionIndex];
      const answerIndex = currentAnswers.findIndex(a => a.answer_id === answer.answer_id);

      // If the answer is already selected, remove it (toggle off)
      if (answerIndex > -1) {
        currentAnswers.splice(answerIndex, 1);
      } else {
        // Otherwise, add the answer to the selected list (toggle on)
        currentAnswers.push(answer);
      }

      // Since we are modifying an array directly, we need to ensure the changes are reactive
      this.$set(this.selectedAnswers, questionIndex, [...currentAnswers]);
    },

    submitAnswers() {
      this.submissionAttempted = true;
      this.points = 0;  // Start with zero points
      this.correctAnswers = {};  // Reset for new submission evaluations
      this.incorrectDetails = [];  // Reset the incorrect details

      this.questions.forEach((question, questionIndex) => {
        const selectedIds = (this.selectedAnswers[questionIndex] || []).map(a => a.answer_id);
        this.correctAnswers[questionIndex] = {};  // Initialize correctness tracking for this question

        question.answers.forEach(answer => {
          // Check if the answer ID is in the selected answers
          const isSelected = selectedIds.includes(answer.answer_id);
          const isCorrect = answer.is_correct;

          // Record if the selected answer is correct or not
          this.correctAnswers[questionIndex][answer.answer_id] = isSelected === isCorrect;

          if (isCorrect) {
            if (isSelected) {
              this.points++;  // Gain a point for selecting a correct answer
            } else {
              this.points--;  // Lose a point for not selecting a correct answer
            }
          } else {
            if (isSelected) {
              this.points--;  // Lose a point for selecting a wrong answer
            } else {
              this.points++;  // Gain a point for not selecting a wrong answer
            }
          }

          // Collect details of incorrect answers for analysis
          if (!this.correctAnswers[questionIndex][answer.answer_id]) {
            this.incorrectDetails.push({
              questionId: question.id,
              answerId: answer.answer_id
            });
          }
        });
      });

      this.percentage = (this.points / this.maxPoints) * 100;  // Calculate the percentage score
    },

    async downloadPdf() {
      try {
        const questionsData = {
          questions: this.questions
        };

        console.log(JSON.stringify(questionsData))

        const response = await fetch('https://medik-cloud-deploy-fast-xxtgwkr47a-lm.a.run.app/generate_pdf_new_method/', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(questionsData),
        });

        if (!response.ok) throw new Error('Network response was not ok.');

        const blob = await response.blob();
        const downloadUrl = window.URL.createObjectURL(blob);
        const link = document.createElement('a');
        link.href = downloadUrl;
        link.setAttribute('download', 'questions.pdf');
        document.body.appendChild(link);
        link.click();
        link.remove();
        window.URL.revokeObjectURL(downloadUrl);
      } catch (error) {
        console.error('Error during PDF generation or download:', error);
      }
    },

    refreshTest() {
      this.selectedAnswers = {};  // Clear selected answers
      this.submissionAttempted = false;  // Allow re-submission
      this.correctAnswers = {};  // Clear correctness tracking
      this.points = 0;  // Reset points
      this.percentage = 0;  // Reset percentage
      this.incorrectDetails = [];  // Clear details of incorrect answers

      this.fetchQuestions();


    },

    /*
    async generateTest() {
      try {
        const response = await this.$axios({
          //url: 'https://medik-cloud-i4zdozbjjq-lm.a.run.app/generate-pdf/', //deploy old
          //url: 'http://127.0.0.1:8081/generate-pdf/',                                        //localhost
          url: 'https://medik-cloud-deploy-xxtgwkr47a-uc.a.run.app/generate-pdf/',
          method: 'GET',
          responseType: 'blob',
          params: {
            numQuestions: this.numQuestions,
            startQuestion: this.startQuestion,
            endQuestion: this.endQuestion,
            numAnswers: this.numAnswers,
            categories: this.checkedCategories.join(',')
          },
        });

        const url = window.URL.createObjectURL(new Blob([response.data]));
        const link = document.createElement('a');
        link.href = url;
        link.setAttribute('download', 'tests_package.zip');
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        window.URL.revokeObjectURL(url);
      } catch (error) {
        console.error('Error during PDF generation or download:', error);
      }
    },
     */
    scrollToQuestion(index) {
      const questionRef = this.$refs['question-' + index][0];
      if (questionRef) {
        questionRef.scrollIntoView({ behavior: 'smooth', block: 'start' });
      }
    },
  }
};

</script>

<style>

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
