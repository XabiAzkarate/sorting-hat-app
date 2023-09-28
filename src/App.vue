<!-- src/App.vue -->

<template>
  <div id="app">
    <h1 class="app-title">The Sorting Hat</h1>
    <div class="chat-window">
      <transition-group name="message-fade" tag="div">
        <ChatMessage 
          v-for="message in messages" 
          :key="message.id"
          :content="message.content"
          :sender="message.sender"
        />
      </transition-group>
    </div>
    <div v-if="!isNameAsked" class="input-block">
      <input type="text" v-model="username" @keyup.enter="askName" placeholder="Enter your name" />
      <button @click="askName">Submit</button>
    </div>
    <div v-else-if="currentQuestion && !isTestComplete" class="answers">
      <button v-for="(answer, index) in currentQuestion.answers" :key="index" @click="selectAnswer(answer)">
        {{ answer.title }}
      </button>
    </div>
    <div v-if="isTestComplete" class="result">
      Congratulations, {{ username }}! Your house is: {{ determinedHouse }}
    </div>
  </div>
</template>

<script>
import ChatMessage from './components/ChatMessage.vue';
import questionsData from './data/questions.json';

export default {
  components: {
    ChatMessage
  },
  data() {
    return {
      questions: questionsData,
      messages: [],
      currentQuestionIndex: 0,
      scores: { g: 0, r: 0, h: 0, s: 0 },
      username: '',
      isNameAsked: false,
      isTestComplete: false
    };
  },
  computed: {
    currentQuestion() {
      return this.questions[this.currentQuestionIndex];
    },
    determinedHouse() {
      let maxScore = -Infinity;
      let house = null;
      for (let key in this.scores) {
        if (this.scores[key] > maxScore) {
          maxScore = this.scores[key];
          house = key;
        }
      }
      let houseMapping = {
        g: "Gryffindor",
        r: "Ravenclaw",
        h: "Hufflepuff",
        s: "Slytherin"
      };
      return houseMapping[house];
    }
  },
  methods: {
    selectAnswer(answer) {
      if (!this.isNameAsked) {
        this.username = answer.title;
        this.isNameAsked = true;
        setTimeout(() => {
          this.messages.push({
            id: Date.now(),
            content: this.currentQuestion.title,
            sender: 'hat'
          });
        }, 800);
        return;
      }

      for (let house in answer.scores) {
        this.scores[house] += answer.scores[house];
      }
      
      this.messages.push({ id: Date.now(), content: answer.title, sender: 'user' });

      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++;
        setTimeout(() => {
          this.messages.push({
            id: Date.now(),
            content: this.currentQuestion.title,
            sender: 'hat'
          });
        }, 800);
      } else {
        setTimeout(() => {
          this.isTestComplete = true;
        }, 800);
      }
    },
    askName() {
      this.isNameAsked = true;
      this.messages.push({
        id: Date.now(),
        content: `Hello, ${this.username}! Let's start with your psychological research.`,
        sender: 'user'
      });
      this.messages.push({
        id:  Date.now(),
        content: this.currentQuestion.title,
        sender: 'hat'
      });
    },
    smoothScroll(target) {
      const chatWindow = this.$el.querySelector(".chat-window");
      const startPos = chatWindow.scrollTop;
      const distance = target - startPos;
      const duration = 800;
      let startTime = null;

      function animation(currentTime) {
        if (startTime === null) startTime = currentTime;
        const timeElapsed = currentTime - startTime;
        const run = ease(timeElapsed, startPos, distance, duration);
        chatWindow.scrollTop = run;
        if (timeElapsed < duration) requestAnimationFrame(animation);
      }

      function ease(t, b, c, d) {
        t /= d / 2;
        if (t < 1) return c / 2 * t * t + b;
        t--;
        return -c / 2 * (t * (t - 2) - 1) + b;
      }

      requestAnimationFrame(animation);
    },
  },
  created() {
    setTimeout(() => {
      this.messages.push({
        id: Date.now(),
        content: "Hello! What's your name?",
        sender: 'hat'
      });
    }, 800);
  },
  updated() {
    this.$nextTick(() => {
      setTimeout(() => {
        const chatWindow = this.$el.querySelector(".chat-window");
        this.smoothScroll(chatWindow.scrollHeight);
      }, 200); // 100ms delay
    });
  }
}
</script>

<style>
body {
  margin: 0;
  background-image: url('/public/images/hat image.jpg');
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center center;
}

#app {
  display: flex;
  flex-direction: column;
  height: 100vh;
  font-family: 'Arial', sans-serif;
  /* background-color: #e5c9f59a; */
  align-items: stretch;
}

.app-title {
  text-align: center;
  font-size: 2em;
  height: auto;
  background-color: white;
  opacity: 0.7;
  margin: 1% 2%;
  border-radius: 10px;
}

.chat-window {
  width: 96%;  /* Set to full width */
  height: 75%;
  min-height: 200px;
  border: 1px solid #e1e1e1;
  overflow-y: auto;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); 
  display: flex;
  flex-direction: column;
  margin: 0% 2%;
  opacity: 0.7;
  font-weight: bold;
  scroll-behavior: smooth;
}

.answers {
  margin-top: 20px;
  width: 100%;
  display: flex;
  flex-wrap: wrap; /* Allow wrapping */
  justify-content: center; /* Center horizontally */
  font-weight: bold;
}

button {
  margin: 5px; /* Add horizontal spacing */
  padding: 10px 20px;
  background-color: #270d3b;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  width: calc(33% - 10px); /* Divide by three and subtract margin for desktop view */
  text-align: center;
}

/* Media query for screens with a width of 768 pixels or less */
@media (max-width: 768px) {
  button {
    width: 90%; /* Make buttons take almost the full width on small screens */
    margin: 10px 5%; /* Center the buttons with a 5% margin on each side */
  }
}

button:hover {
  background-color: #2c3e50; /* Slight color change on hover */
}

/* Enter and leave transitions */
.message-fade-enter-active,
.message-fade-leave-active {
  transition: all 0.8s ease;
}
.message-fade-enter-from,
.message-fade-leave-to {
  opacity: 0;
  transform: translateX(-50px);
}

.result {
  font-size: 1.5em;
  text-align: center;
  margin-top: 2rem;
  background-color: #270d3b;
  color: white;
  padding: 1rem;
}

input {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
  margin-right: 10px;
  width: 100%;
}

.input-block {
  width: 96%;  /* Set to full width */ 
  display: flex;
  flex-direction: horizontal;
  margin: 0% 2%;
  padding-top: 1%;
}

.input-block button {
  width: 40%;
  font-weight: bold;
}

/* Styles for Chrome, Safari, Edge */
.chat-window::-webkit-scrollbar {
    width: 10px; /* Adjust width of the scrollbar */
}

.chat-window::-webkit-scrollbar-track {
    background: #f1f1f1; /* Color of the track */
    border-radius: 10px; /* Roundness of the track */
}

.chat-window::-webkit-scrollbar-thumb {
    background: #888; /* Color of the scroll thumb */
    border-radius: 10px; /* Roundness of the scroll thumb */
}

.chat-window::-webkit-scrollbar-thumb:hover {
    background: #555; /* Color of the scroll thumb on hover */
}

/* Styles for Firefox */
.chat-window {
    scrollbar-width: thin; /* Adjust width of the scrollbar */
    scrollbar-color: #888 #f1f1f1; /* First color is thumb, second is track */
}

</style>
