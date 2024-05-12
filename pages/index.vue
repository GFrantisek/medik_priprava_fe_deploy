  <template>
    <div class="container">
      <div class="navigation">
        <NuxtLink to="/" class="nav-item">LF UK BA</NuxtLink>
        <NuxtLink to="/" class="nav-item">JLF UK</NuxtLink>
        <NuxtLink to="/" class="nav-item">LF UPJŠ</NuxtLink>
        <NuxtLink to="/" class="nav-item">LF MU</NuxtLink>
        <NuxtLink to="/" class="nav-item">LF UPOL</NuxtLink>
        <NuxtLink to="/" class="nav-item">LF HK</NuxtLink>
        <NuxtLink to="/" class="nav-item">LFP UK</NuxtLink>
        <NuxtLink to="/" class="nav-item">1. LF UK</NuxtLink>
        <NuxtLink to="/" class="nav-item">2. LF UK</NuxtLink>
        <NuxtLink to="/" class="nav-item">3. LF UK</NuxtLink>
        <div class="auth-actions">
          <!-- Check if user is authenticated -->
          <div v-if="$auth.loggedIn">
            <!-- Display logout button if user is logged in -->
            <button class="profile-button" @click="navigateToProfile">
              <svg xmlns="http://www.w3.org/2000/svg" width="22" height="22" fill="currentColor" class="bi bi-person-circle" viewBox="0 0 16 16">
                <path d="M11 6a3 3 0 1 1-6 0 3 3 0 0 1 6 0"/>
                <path fill-rule="evenodd" d="M0 8a8 8 0 1 1 16 0A8 8 0 0 1 0 8m8-7a7 7 0 0 0-5.468 11.37C3.242 11.226 4.805 10 8 10s4.757 1.225 5.468 2.37A7 7 0 0 0 8 1"/>
              </svg>
            </button>
            <button @click="logout" class="logout-button">Logout</button>
          </div>
          <div v-else class='login-register'>
            <!-- Display login and register links if user is not logged in -->
            <nuxt-link to="/auth/login"  class="helpme">Login</nuxt-link>
            <nuxt-link to="/auth/register">Register</nuxt-link>
          </div>
        </div>
      </div>

      <div class="page-title">
        LF UK Bratislava
      </div>

      <div class="three-column-layout">
        <div class="text-column">
          <p>- biológia (100 otázok)</p>
          <p>  - chémia (100 otázok)</p>
          <p>  - pri prezenčnej aj pri online forme prijímacej skúšky test trvá 3 hodiny</p>
          <p>   - prijímacia skúška sa koná samostatne na každý študijný program:</p>
          <p>   15. 6. 2024 všeobecné lekárstvo: online, prezenčne v mestách: Bratislava, Nitra, Banská Bystrica, Košice, Prešov</p>
          <p>   16. 6. 2024 zubné lekárstvo: online, prezenčne v mestách: Bratislava, Banská Bystrica, Košice</p>
          <p>  19. 6. 2024 náhradný termín: online, prezenčne v Bratislave</p>
        </div>

        <div class="subjects-column">
          <button class="subject-item" @click="selectSubject('biologia')">
            <img src="~assets/icons/biologia.png" alt="Biologia" class="subject-icon">
            <span>Biologia</span>
          </button>
          <button class="subject-item" @click="selectSubject('chemia')">
            <img src="~assets/icons/chemia.png" alt="Chemia" class="subject-icon">
            <span>Chemia</span>
          </button>
          <button class="subject-item" @click="selectSubject('fyzika')">
            <img src="~assets/icons/fyzika.png" alt="Fyzika" class="subject-icon">
            <span>Fyzika</span>
          </button>
        </div>


        <div class="test-column">
          <div class="test-container">
            <button @click="generateInteractiveTest" class="standard-test-btn">ŠTANDARDNÝ TEST</button>
            <button @click="generateStandardTest">Stiahnuť ako pdf</button>
            <div class="custom-test">
              <div class="custom-test-title">alebo</div>
              <form class="custom-test-inputs">
                <label for="number-of-questions">Počet otázok:</label>
                <input type="number" id="numQuestions" v-model.number="numQuestions" min="1" max="1500" />
                <label for="number-of-answers">Počet odpovedí (1-8):</label>
                <input type="number" id="numQuestions" v-model.number="numAnswers" min="1" max="8" />
                <label for="from-question-id">Od otázky číslo:</label>
                <input type="number" id="startQuestion" v-model.number="startQuestion" min="1" max="1500" />
                <label for="to-question-id">Po otázku číslo:</label>
                <input type="number" id="endQuestion" v-model.number="endQuestion" min="1" max="1500" />
                <label for="to-question-id">Výber oblastí:</label>
              </form>
              <div class="custom-categories">
                <div v-for="(category, index) in allCategories" :key="index">
                  <input type="checkbox" :id="`category-${index}`" :value="category" v-model="checkedCategories">
                  <label :for="`category-${index}`">{{ category }}</label>
                </div>
              </div>

              <button @click="generateInteractiveTest" class="custom-test-btn">VLASTNÝ TEST</button>
              <button @click="generateTest">Stiahnuť ako pdf</button>
            </div>
          </div>
        </div>
      </div>

    </div>
  </template>

  <script>
  export default {
    computed:{
      isAuthenticated(){
        return this.$auth.loggedIn
      },

      user(){
        return this.$auth.user
      }
    },
    data() {
      return {
        numQuestions: 1,
        startQuestion: 1,
        endQuestion: 1500,
        numAnswers: 4,
        checkedCategories: [],
        allCategories: [
          'Všeobecné informácie', 'Bunková biológia', 'DNA a RNA', 'Genetika',
          'Virológia a baktérie', 'Rastliny a huby', 'Živočíchy', 'Človek',
          'Ekosystém a Zem', 'Populačné choroby'
        ],

      };
    },
    methods: {
      async logout(){
        try {
          let response = await this.$auth.logout()
          this.$router.replace({name: 'auth-login'})
        } catch (error) {
          console.log(error)
        }
      },
      navigateToProfile() {
        // Redirect to the user profile using a relative path
        window.location.href = '/auth/user';
      },
      async generateTest() {
        const normalizedNumQuestions = Math.max(1, Math.min(this.numQuestions, 1500));
        const normalizedNumAnswers = Math.max(1, Math.min(this.numAnswers, 8));
        let normalizedStartQuestion = Math.max(1, Math.min(this.startQuestion, 1500));
        let normalizedEndQuestion = Math.max(1, Math.min(this.endQuestion, 1500));

        try {
          const response = await this.$axios({
            url: 'https://medik-cloud-deploy-fast-beta-first-xxtgwkr47a-lm.a.run.app/generate_pdf_method_from_params/',  // Ensure this URL matches your deployment or development environment
            method: 'POST',
            responseType: 'blob',  // Important for handling binary data
            data: {
              numQuestions: normalizedNumQuestions,
              startQuestion: normalizedStartQuestion,
              endQuestion: normalizedEndQuestion,
              numAnswers: normalizedNumAnswers,
              categories: this.checkedCategories.join(',')
            },
          });

          const url = window.URL.createObjectURL(new Blob([response.data]));
          const link = document.createElement('a');
          link.href = url;
          link.setAttribute('download', 'tests_package.zip');  // Change to reflect that it's a zip file
          document.body.appendChild(link);
          link.click();
          document.body.removeChild(link);
          window.URL.revokeObjectURL(url);
        } catch (error) {
          console.error('Error generating or downloading the PDF:', error);
          alert('Failed to download the test package. Please try again.');  // Provide user feedback
        }
      },


      async generateStandardTest() {
        try {
          const response = await fetch('https://medik-cloud-deploy-fast-beta-first-xxtgwkr47a-lm.a.run.app/generate-pdf-new/');
          if (!response.ok) throw new Error('Network response was not ok.');
          const data = await response.blob();
          const downloadUrl = window.URL.createObjectURL(data);
          const link = document.createElement('a');
          link.href = downloadUrl;
          link.setAttribute('download', 'test_package.zip'); // Suggest a filename for saving
          document.body.appendChild(link);
          link.click();
          document.body.removeChild(link);
          window.URL.revokeObjectURL(downloadUrl);
        } catch (error) {
          console.error('Failed to download the file:', error);
        }
      },


      generateInteractiveTest() {
        const normalizedNumQuestions = Math.max(1, Math.min(this.numQuestions, 1500));
        const normalizedNumAnswers = Math.max(1, Math.min(this.numAnswers, 8));
        const normalizedStartQuestion = Math.max(1, Math.min(this.startQuestion, 1500));
        const normalizedEndQuestion = Math.max(1, Math.min(this.endQuestion, 1500));

        if (this.checkedCategories.length === 0){
          this.$router.push({
            path: '/interaktivny_test',
            query: {
              numQuestions: normalizedNumQuestions.toString(),
              startQuestion: normalizedStartQuestion.toString(),
              endQuestion: normalizedEndQuestion.toString(),
              numAnswers: normalizedNumAnswers.toString(),
              categories: this.checkedCategories.join(',')
            }
          });
        }else{
          this.$router.push({
            path: '/interaktivny_test',
            query: {
              numQuestions: normalizedNumQuestions.toString(),
              startQuestion: '1',
              endQuestion: '1500',
              numAnswers: normalizedNumAnswers.toString(),
              categories: this.checkedCategories.join(',')
            }
          });
        }

      },
    },

  };
  </script>

  <style scoped>
  .container {
    font-family: 'Arial', sans-serif;
    color: #333;
    padding-top: 60px;
  }
  .custom-font {
    font-family: 'Montserrat', sans-serif;
    letter-spacing: 0.1em;
  }
  .helpme{
    padding-right: 25px;
  }
  .login-register{
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-weight: 700;

  }

  .navigation {
    background-color: #BB99FF;
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100vw;
    box-sizing: border-box;
    margin: 0;
    position: fixed;
    top: 0;
    left: 0;
    z-index: 1000;
  }
  .nav-item,  .logout-button {
    color: #fff;
    text-decoration: none;
    font-weight: bold;
    margin-left: 20px;
  }

  .login {
    background-color: #B18BFF;
    padding: 10px 20px;
    border-radius: 20px;
  }

  .page-title {
    text-align: center;
    font-size: 4em;
    color:#99D9D9;
  }
  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }


  .three-column-layout {
    display: flex;
    width: 100vw;
    align-content: center;
    justify-content: space-between;
  }

  .text-column {
    flex: 1;
    padding-right: 20px;
    margin-left: 20px;
    text-align: left;
  }

  .subjects-column {
    flex: 1;
    padding: 0 10px;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
  .subject-item {
    display: flex;
    align-items: center;
    justify-content: start;
    background-color: #f5f5f5;
    border: 1px solid #ddd;
    margin-bottom: 10px;
    cursor: pointer;
    text-decoration: none;
    height: 10vw;
    width: 100%;
    box-sizing: border-box;
  }

  .subject-icon {
    margin-right: 10px;
    width: 350px;
    height: auto;
  }

  button.subject-item {
    padding: 10px;
    border: none;
    outline: none;
    background: none;
    width: 100%;
    text-align: left;
    transition: background-color 0.2s;
  }

  button.subject-item:hover {
    background-color: #eaeaea;
  }

  .test-column {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .subject-item {
    margin-bottom: 10px;
  }

  .subject-icon {
    width: 80px;
    height: auto;
  }


  .test-section {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px 0;
    width: 100%;
  }

  .test-container {
    text-align: center;
    background-color: #E6FAF8;
    padding: 10px;
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    border-radius: 10px;
  }

  .custom-test-inputs input {
    width: calc(100% - 20px);
    margin: 5px 0;
    padding: 5px;
    border: 1px solid #ccc;
    border-radius: 5px;
  }

  .standard-test-btn, .custom-test-btn {
    width: calc(100% - 40px);
    padding: 10px 20px;
    margin: 5px 0;
    border: none;
    border-radius: 5px;
    color: white;
    background-color: #005D60;
    cursor: pointer;
  }

  .custom-test-inputs label {
    display: block;
    text-align: left;
    margin-left: 10px;
    color: #666;
  }

  .custom-categories{
    text-align: left;
    margin-left: 10px;
    color: #666;
  }
  .footer {
    background-color: #37AEB2;
    color: #fff;
    text-align: center;
    padding: 10px;
    position: absolute;
    bottom: 0;
    width: 100%;
  }

  @media (max-width: 768px) {
    .subjects {
      flex-direction: column;
    }

    .navigation {
      flex-direction: column;
    }

    .nav-item:not(.login) {
      margin-bottom: 10px;
    }
  }

  .auth-actions {
    display: flex;
    align-items: center;
    justify-content: flex-end; /* Aligns items to the right but keeps them grouped */
    padding-right: 20px; /* Adds some padding on the right if needed */
  }
  .profile-button {
    background: none;
    border: none;
    color: inherit;
    cursor: pointer;
    padding: 20px 20px; /* Adjust padding to match your design */
    margin-top: 3px;
    transition: transform 0.3s ease; /* Adds smooth transition for transform property */
  }

  /* Ensure no background change on hover */
  .profile-button:hover {
    background: none; /* Explicitly set background to none on hover */
    transform: scale(1.4); /* Slightly enlarges the button on hover */
  }

  .logout-button {
    background: none;
    border: none;
    color: inherit;
    cursor: pointer;
    padding: 20px 20px; /* Adjust padding to match your design */
    transition: transform 0.3s ease; /* Adds smooth transition for transform property */
    margin-bottom: 3px;
  }

  .logout-button:hover {
    background: none; /* Explicitly set background to none on hover */
    transform: scale(1.4); /* Slightly enlarges the button on hover */
  }



  </style>
