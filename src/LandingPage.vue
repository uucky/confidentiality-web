<template>
  <div class="layout">
    <h1 class="title">
      Take our survey.
    </h1>
    <p class="description">
      This survey was conducted by a group of SIAT students for the purpose of improving the participant’s experience of filling in survey form. By doing this survey, you will help us to get a better understanding of participants' experience with the questions and how the survey is set up.
    </p>
    <p
      v-if="error"
      class="errorStatus"
    >
      {{ error }}
    </p>
    <div class="button-container">
      <button
        type="button"
        class="btn-regular"
        :disabled="status === 'checking'"
        @click="startLogin"
      >
        {{ status === 'checking' ? 'Logging in...' : 'Start' }}
      </button>
    </div>
    <RandomFruit
      v-if="started"
    />
    <section class="footnote-container">
      <p class="footnote">
        By clicking Start, you agree to our
        <router-link to="/policies">
          <span class="underlined">Terms, Data and Cookies Policy</span>.
        </router-link>
      </p>
    </section>
  </div>
</template>
<script>
import RandomFruit from './components/RandomFruit';
import { auth, answerCollection } from '@/utils/firebase';

export default {
  name: 'Landing',
  components: {
    RandomFruit,
  },
  data() {
    return {
      status: 'ready',
      error: null,
      loggedIn: false,
      started: false,
      loginMsg: {
        loggingIn: 'Logging in...',
        failed: 'Bad Internet, try again?',
        success: 'Successfully logged in.',
      },
    };
  },
  methods: {
    async checkLogin() {
      try {
        this.status = 'checking';
        await auth.setPersistence('local');
        await auth.signInAnonymously();
        auth.onAuthStateChanged((user) => {
          if (user) {
            answerCollection.where('uid', '==', user.uid).get().then((querySnapshot) => {
              this.status = 'ready';
              if (querySnapshot.size > 1) throw new Error('Found more than one document.');
              if (querySnapshot.size === 1) {
                this.$store.commit('SET_FRUIT', { fruit: querySnapshot.docs[0].data().fruit });
                this.$router.push({ path: '/result' });
              } else {
                // no doc found means no answer yet submitted
                this.$store.commit('SET_UID', { uid: user.uid });
                this.started = true;
              }
            });
          } else {
            this.status = 'ready';
            this.error = 'Not logged in.';
          }
        });
      } catch (e) {
        this.status = 'ready';
        this.error = e.message;
      }
    },
    startLogin() {
      this.checkLogin();
    },
  },
};
</script>
<style scoped lang="postcss">
.title {
  @apply block w-full text-accent;
  font-family: 'Palanquin Dark', system-ui;
  font-size: 3.5em;
  font-weight: 500;
  margin-bottom: 2rem;

  line-height: 100%;
}

.layout {
  display: flex;
  flex-flow: row wrap;
  width: 100%;
  max-width: 40em;
  margin: 20% auto 0;

  padding-left: 1.8rem;
  padding-right: 1.8rem;
}

.button-container {
  width: 100%;
  text-align: center;
  margin-bottom: 2em;
}

.description {
  @apply w-full text-sub;
  font-size: 1em;
  font-family: 'Work Sans', system-ui;
  margin-bottom: 5.25em;

  line-height: 180%;
}

.errorStatus {
  text-align: center;
  padding: .25em 0;
  margin-bottom: 1em;
  font-family: 'Palanquin Dark', system-ui;
  font-size: 1.25em;
  color: #EB5757;
  background-color: #FFEBEB;
}

.footnote-container {
  width: 100%;
  text-align: center;
  padding: 0 3rem;
}


</style>
