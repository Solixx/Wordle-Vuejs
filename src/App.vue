<script setup>
  import SimpleKeyboard from "./components/SimpleKeyboard.vue";
  import WordRow from "./components/WordRow.vue";
  import { reactive, onMounted, computed } from "vue";

  const state = reactive({
    solution: "books",
    guesses: ["", "", "", "", "", ""],
    currentGuessIndex: 0,
    guessedLetters: {
      miss: [],
      found: [],
      hint: [],
    },
  })

  const wonGame = computed(() => state.guesses[state.currentGuessIndex - 1] === state.solution)

  const lostGame = computed(() => !wonGame.value && state.currentGuessIndex >= 6)

  const handleInput = (key) => {
    if(state.currentGuessIndex >= 6 || wonGame.value)  return

    const currentGuess = state.guesses[state.currentGuessIndex]

    if(key == "{enter}"){

      if(currentGuess.length == 5){
        state.currentGuessIndex++;

        for(var i = 0; i < currentGuess.length; i++){
          let c = currentGuess.charAt(i)
          if(c == state.solution.charAt(i)){
            state.guessedLetters.found.push(c)
          } else if(state.solution.indexOf(c) != -1){
            state.guessedLetters.hint.push(c)
          } else{
            state.guessedLetters.miss.push(c)
          }
        }
      }
    } else if(key == "{bksp}"){

      state.guesses[state.currentGuessIndex] = currentGuess.slice(0, -1)
    } else if(currentGuess.length < 5){

      const alphaRegex = /[a-zA-Z]/
      if(alphaRegex.test(key)){
        state.guesses[state.currentGuessIndex] += key
      }
    }
  }

  onMounted(() => {
    window.addEventListener("keyup", (e) => {
      e.preventDefault()
      let key = 
        e.keyCode == 13
          ? "{enter}"
          : e.keyCode == 8
          ? "{bksp}"
          : String.fromCharCode(e.keyCode).toLowerCase()
      handleInput(key)    
    })
  })
</script>

<template>
  <div class="flex flex-col h-screen max-w-md mx-auto justify-evenly">
    <div>
      <WordRow 
        v-for="(guess, i) in state.guesses"
        :key="i"
        :value="guess"
        :solution="state.solution"
        :submitted="i < state.currentGuessIndex"
      />
    </div>
    <p v-if="wonGame" class="text-center"> Congrats you solve it! </p>
    <p v-else-if="lostGame" class="text-center"> Out of tries. </p>
    <SimpleKeyboard 
      @onKeyPress="handleInput" 
      :guessedLetters="state.guessedLetters"
    />
  </div>
</template>

<style scoped>

</style>
