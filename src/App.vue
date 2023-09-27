<!-- src/App.vue -->

<template>
  <div id="app">
    <h1 class="app-title">The Sorting Hat</h1>
    <div class="chat-window">
      <transition-group name="list" tag="div">
          <ChatMessage 
            v-for="message in messages" 
            :key="message.id"
            :content="message.content"
            :sender="message.sender"
          />
        </transition-group>
    </div>

    <div v-if="currentQuestion" class="answers">
      <button v-for="(answer, index) in currentQuestion.answers" :key="index" @click="selectAnswer(answer)">
        {{ answer.title }}
      </button>
    </div>

    <div v-else>
      Your house is: {{ determinedHouse }}
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
      scores: { g: 0, r: 0, h: 0, s: 0 }
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
      for (let house in answer.scores) {
        this.scores[house] += answer.scores[house];
      }
      this.messages.push({ content: answer.title, sender: 'user' });

      // Check if it's the last question
      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++;
        this.messages.push({
          content: this.currentQuestion.title,
          sender: 'hat'
        });
      } else {
        // Display the result if it's the last question
        this.messages.push({
          content: `Your house is: ${this.determinedHouse}`,
          sender: 'hat'
        });
      }
    }
  },
  created() {
    this.messages.push({
      content: this.currentQuestion.title,
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
}

.chat-window {
  width: 96%;  /* Set to full width */
  height: 65%;
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
  width: calc(33% - 10px); /* Divide by three and subtract margin */
  text-align: center;
}

button:hover {
  background-color: #2c3e50; /* Slight color change on hover */
}

.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

</style>
