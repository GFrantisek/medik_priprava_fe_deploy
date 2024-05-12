  <template>
    <div>
      <div v-if="isAuthenticated">
        <div class="user-header">
          <h3>Welcome, {{ user?.username }}!</h3>
          <NuxtLink :to="{name: 'index'}" class="btn btn-link">Back to Home</NuxtLink>
          <button type="button" class="btn btn-danger" @click="logout">Logout</button>
        </div>

        <ul v-if="tests.length" class="list-unstyled">
          <li v-for="(test, index) in tests" :key="test.id" class="test-summary">
            <NuxtLink :to="'/test-details/' + test.test_id" class="test-link">
              View Test Details for Test {{ index + 1 }}
            </NuxtLink>
            <div class="test-score">
              Score: {{ test.score }}/{{ test.max_score }}
              ({{ (test.score / test.max_score * 100).toFixed(2) }}%)
            </div>
            <div class="test-outcome">
              <span v-if="(test.score / test.max_score * 100) >= 80" class="accepted">Accepted ✅</span>
              <!-- <span v-else class="rejected">😢</span> !-->
            </div>
            <div class="test-date">
              Taken on: {{ new Date(test.test_date).toLocaleDateString() }} at {{ new Date(test.test_date).toLocaleTimeString() }}
            </div>
          </li>
        </ul>
        <p v-if="showNoTestsMessage"  v-else class="no-tests">
            Welcome to your profile, here you will have your test history. Right now, you don't have any tests ready to watch.
            Feel free to go to the menu and select your test.
        </p>
      </div>
    </div>
  </template>

  <script>
  export default {
    name: "User",
    middleware: 'auth',
    data() {
      return {
        tests: [],
        showNoTestsMessage: false
      };
    },
    computed: {
      isAuthenticated() {
        return this.$auth.loggedIn;
      },
      user() {
        return this.$auth.user;
      },
    },
    mounted() {
      this.fetchTestHistory();
      setTimeout(() => {
        if (!this.tests.length) {
          this.showNoTestsMessage = true;
        }
      }, 1241);
    },
    methods: {
      async fetchTestHistory() {
        if (!this.isAuthenticated) return;  // Ensure user is authenticated

        // Use user ID from auth module in API request
        const userId = this.user.id;
        const url = `/user/${userId}/tests/`;
        try {
          const response = await this.$axios.get(url);
          this.tests = response.data;
        } catch (error) {
          console.error("Failed to fetch test history:", error);
        }
      },
      async logout() {
        try {
          await this.$auth.logout();
          this.$router.replace({name: 'auth-login'});
        } catch (error) {
          console.error("Logout failed:", error);
        }
      }
    }
  }
  </script>

  <style scoped>
  .no-tests {
    padding: 20px;
    background-color: #f4f4f4;
    border-left: 4px solid #007bff;
    margin-top: 20px;
    color: #333;
    font-size: 1rem;
  }

  .user-header {

    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 25px 20px;
    background-color: #f1f1f1;
    border-bottom: 2px solid #ccc;
    margin-bottom: 20px;
  }


  .test-summary {
    margin-bottom: 20px;
    background-color: #f8f8f8;
    padding: 15px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }

  .test-link {
    color: #0056b3;
    text-decoration: none;
    font-weight: bold;
    display: block;
  }

  .test-link:hover {
    text-decoration: underline;
  }

  .test-score {
    color: #333;
    font-weight: bold;
  }

  .test-outcome {
    margin-top: 5px;
    font-size: 1.2rem;
  }

  .accepted {
    color: green;
    font-weight: bold;
  }

  .rejected {
    color: red;
    font-weight: bold;
  }

  .test-date {
    color: #555;
    font-size: 0.9rem;
    margin-top: 5px;
  }

  .btn-danger {
    color: #fff;
    background-color: #dc3545;
    border-color: #dc3545;
  }

  .btn-link {
    color: #007bff;
    text-decoration: none;
  }

  .btn-link:hover {
    text-decoration: underline;
  }
  </style>
