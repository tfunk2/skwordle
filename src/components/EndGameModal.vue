<template>
  <div class="modal-container">
    <div class="end-game-modal">
      <h2 class="end-game-reaction">{{ endGameReaction }}</h2>
      <StatsChart 
        :total-wins="totalWins" 
        :highlighted-number="highlightedNumber"
        :is-current-guess-correct="isCurrentGuessCorrect"
        :stats-chart-data="statsChartData"
      />
      <div v-if="isCurrentGuessCorrect" class="bottom-content">
        <p class="streak-text">Streak: <span class="streak-number">{{ winStreak }}</span></p>
        <p class="win-text">
          <span id="winning-word">{{ currentWinningWord.toUpperCase() }}</span>
          was the word
        </p>
      </div>
      <div v-else class="bottom-content">
        <p class="win-text">
          <span id="losing-word">{{ currentWinningWord.toUpperCase() }}</span>
          was the word
        </p>
      </div>
      <p class="next-word-button" @click="emitNextWord">NEXT WORD</p>
    </div>
  </div>
</template>

<script lang='ts'>
import { computed, ComputedRef, defineComponent, PropType } from "vue";
import StatsChart from "./StatsChart.vue"

export default defineComponent({
  name: "EndGameModal",
  components: {
    StatsChart
  },
  props: {
    currentGuess: {
      type: String,
      required: true,
    },
    winStreak: {
      type: Number,
      required: true,
    },
    currentWinningWord: {
      type: String,
      required: true,
    },
    isCurrentGuessCorrect: {
      type: Boolean,
      required: true,
    },
    totalWins: {
      type: Number,
      required: false,
      default: 0
    },
    guessesPerWin: {
      type: Array as PropType<number[]>,
      required: false,
      default: () => []
    }
  },
  setup(props, { emit }) {
    const emitNextWord = (): void => {
      emit("next-word");
    }

    const highlightedNumber = computed(() => {
      return props.guessesPerWin[props.guessesPerWin.length - 1]
    })

    const endGameReaction = computed(() => {
      if (!props.isCurrentGuessCorrect) {
        return 'So close!'
      }

      switch (highlightedNumber.value) {
        case 1:
          return 'Simply Unbelievable!'
        case 2:
          return 'Incredible!'
        case 3:
          return 'Impressive!'
        case 4:
          return 'You did it!'
        case 5:
          return 'Just in time!'
        case 6:
          return 'Phew! That was close.'
        default: 
          return 'Good job!'
      }
    })

    const statsChartData: ComputedRef<Number[]> = computed(() => {
      const counts1 = props.guessesPerWin.filter((guessCount) => guessCount === 1).length
      const counts2 = props.guessesPerWin.filter((guessCount) => guessCount === 2).length
      const counts3 = props.guessesPerWin.filter((guessCount) => guessCount === 3).length
      const counts4 = props.guessesPerWin.filter((guessCount) => guessCount === 4).length
      const counts5 = props.guessesPerWin.filter((guessCount) => guessCount === 5).length
      const counts6 = props.guessesPerWin.filter((guessCount) => guessCount === 6).length

      return [counts1, counts2, counts3, counts4, counts5, counts6]
    })

    return {
      endGameReaction,
      emitNextWord,
      highlightedNumber,
      statsChartData
    }
  }
});
</script>

<style scoped>
.modal-container {
  position: absolute;
  z-index: 100;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100vw;
  background-color: rgba(50, 50, 50, 0.7);
  padding: 1rem;
  padding-top: calc(1rem + env(safe-area-inset-top));
  padding-bottom: calc(1rem + env(safe-area-inset-bottom));
  padding-left: calc(1rem + env(safe-area-inset-left));
  padding-right: calc(1rem + env(safe-area-inset-right));
  box-sizing: border-box;
}

.end-game-modal {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  border-radius: 8px;
  width: 100%;
  max-width: 500px;
  min-height: 400px;
  max-height: 90vh;
  background: #0a0a0a;
  cursor: default;
  padding: 1.5rem 1rem;
  box-sizing: border-box;
  overflow-y: auto;
}

.end-game-reaction {
  color: white;
  padding-top: 0.5rem;
  margin: 0 0 1rem 0;
  font-family: 'Bungee Hairline', sans-serif;
  font-size: clamp(1.4rem, 5vw, 2rem);
  text-align: center;
  width: 100%;
}

.bottom-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  width: 100%;
  padding: 0.5rem 0;
}

.win-text {
  font-family: 'Bungee Hairline', sans-serif;
  color: white;
  font-size: clamp(1rem, 3vw, 1.4rem);
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0;
  text-align: center;
  line-height: 1.4;
  font-weight: bold;
}

.streak-text {
  font-family: 'Bungee Hairline', sans-serif;
  color: white;
  font-size: clamp(1.2rem, 4vw, 1.8rem);
  margin: 0;
  text-align: center;
  font-weight: bold;
}

.streak-number {
  font-family: "Monofett", sans-serif;
  color: rgb(255, 174, 0);
  font-size: clamp(1.8rem, 6vw, 2em);
}

#winning-word {
  font-family: "Monofett", sans-serif;
  color: rgb(62, 172, 62);
  font-size: clamp(2rem, 6vw, 2.8em);
  letter-spacing: 2px;
  line-height: 1.2;
  padding-bottom: 0.5rem;
  word-break: break-word;
}

#losing-word {
  font-family: "Monofett", sans-serif;
  color: rgb(214, 40, 40);
  font-size: clamp(1.5rem, 5vw, 2em);
  letter-spacing: 2px;
  line-height: 1.2;
  padding-bottom: 0.5rem;
  word-break: break-word;
}

.next-word-button {
  font-family: "Monofett", sans-serif;
  color: white;
  font-size: clamp(2.2rem, 6vw, 3em);
  cursor: pointer;
  margin-top: 0.5rem;
  padding: 0.5rem 1rem;
  transition: color 0.2s ease;
  text-align: center;
}

.next-word-button:hover {
  color: rgb(200, 200, 200);
}

.next-word-button:active {
  transform: scale(0.98);
}

/* Tablet and larger screens */
@media screen and (min-width: 768px) {
  .end-game-modal {
    padding: 2rem 1.5rem;
    min-height: 500px;
  }

  .end-game-reaction {
    margin-bottom: 1.5rem;
  }

  .bottom-content {
    gap: 1.5rem;
    padding: 1rem 0;
  }
}

/* Mobile-specific adjustments */
@media screen and (max-width: 480px) {
  .modal-container {
    padding: 0.5rem;
  }

  .end-game-modal {
    padding: 1rem 1.5rem;
    min-height: auto;
    max-height: 95vh;
  }

  .end-game-reaction {
    padding-top: 0.25rem;
    margin-bottom: 0.75rem;
  }

  .bottom-content {
    gap: 0.75rem;
    padding: 0.5rem 0;
  }

  #winning-word {
    font-size: clamp(2.5rem, 8vw, 3.5em);
  }

  #losing-word {
    font-size: clamp(2.5rem, 8vw, 3.5em);
  }

  .next-word-button {
    margin-top: 0.25rem;
    padding: 0.4rem 0.8rem;
    font-size: clamp(2.2rem, 8vw, 3.2em);
  }
}
</style>
