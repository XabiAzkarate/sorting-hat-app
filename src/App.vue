<!-- src/App.vue -->

<template>
  <div id="app">
    <h1 class="app-title">The Sorting Hat</h1>
    <div class="chat-window">
      <ChatMessage 
        v-for="message in messages" 
        :key="message.id"
        :content="message.content"
        :sender="message.sender"
      />
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
        this.messages.push({
          content: this.currentQuestion.title,
          sender: 'hat'
        });
        return;
      }
      for (let house in answer.scores) {
        this.scores[house] += answer.scores[house];
      }
      this.messages.push({ content: answer.title, sender: 'user' });

      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++;
        this.messages.push({
          content: this.currentQuestion.title,
          sender: 'hat'
        });
      } else {
        this.isTestComplete = true;
      }
    },
    askName() {
      this.isNameAsked = true;
      this.messages.push({
        content: `Hello, ${this.username}! Let's start with your psychological research.`,
        sender: 'user'
      });
      this.messages.push({
        content: this.currentQuestion.title,
        sender: 'hat'
      });
    },
  },
  created() {
    this.messages.push({
      content: "Hello! What's your name?",
      sender: 'hat'
    });
  },
  updated() {
    const chatWindow = this.$el.querySelector(".chat-window");
    chatWindow.scrollTop = chatWindow.scrollHeight;
  }
}
</script>

<style>
body {
  margin: 0;
}

#app {
  display: flex;
  flex-direction: column;
  height: 100vh;
  font-family: 'Arial', sans-serif;
  background-color: #e5c9f59a;
  align-items: stretch;  /* Adjust this to stretch */
}

.app-title {
  text-align: center;
  margin-top: 2vh;
  font-size: 2em;
  color: #270d3b;
  height: auto;
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
}

.answers {
  margin-top: 20px;
  width: 100%;
  display: flex;
  flex-wrap: wrap; /* Allow wrapping */
  justify-content: center; /* Center horizontally */
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

.list-enter-active,
.list-leave-active {
  transition: all 5s ease; /* Adjust the timing if needed */
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateY(30px); /* Adjust for desired slide direction */
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
}

</style>
