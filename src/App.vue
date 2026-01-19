<template>
  <div class="app-container">
    <Header />

    <SessionStats 
      :win-streak="winStreak"
      :longest-streak="longestStreak"
      :win-loss-percentage="winLossPercentage"
    />
    <div>
      <form @submit.prevent autocomplete="off" class="guess-form">
        <div class="letter-input-container">
          <input
            class="five-letter-input"
            id="pending-guess"
            v-model="pendingGuess"
            maxlength="5"
            minlength="1"
            autofocus
            :readonly="isMobileDevice"
            :inputmode="isMobileDevice ? 'none' : undefined"
            @focus="handleInputFocus"
            @blur="handleInputBlur"
            @click="handleInputClick"
          />
          <input
            v-show="!isGuessingComplete"
            class="button"
            type="submit"
            value="Enter"
            @click="setCurrentGuess"
            ref="submit"
          />
        </div>
      </form>
      <GameBoard
        :current-winning-word="currentWinningWord"
        :current-guess="currentGuess"
        :used-words="usedWords"
        :is-guessing-complete="isGuessingComplete"
        :pending-guess="pendingGuess"
        :shake-word-guess="shakeWordGuess"
      />
    </div>
    <transition name="modal">
      <EndGameModal
        v-if="isGuessingComplete"
        :currentGuess="currentGuess"
        :winStreak="winStreak"
        :currentWinningWord="currentWinningWord"
        :isCurrentGuessCorrect="isCurrentGuessCorrect"
        :guesses-per-win="guessesPerWin"
        :total-wins="totalWins"
        @next-word="changeWordClearBoard()"
      />
    </transition>
    <Keyboard 
      :guessed-letters="guessedLetters" 
      @type-letter="typeLetter($event)"
      @backspace="backspace"
      @submit-guess="setCurrentGuess"
    />
  </div>
</template>

<script lang="ts">
import { computed, ComputedRef, defineComponent, onMounted, Ref, ref, watch } from "vue";
import EndGameModal from "./components/EndGameModal.vue";
import SessionStats from "./components/SessionStats.vue";
import GameBoard from "./components/GameBoard.vue";
import Header from "./components/Header.vue";
import Keyboard from "./components/Keyboard.vue";
import { fiveLetterWinningWords } from "./utils/FiveLetterWords";
import { isFiveLetterWord, isWordPlayable } from "./utils/WordChecker";
// const _ = require("lodash");

export default defineComponent({
  name: "App",
  components: {
    EndGameModal,
    GameBoard,
    Header,
    Keyboard,
    SessionStats
  },
  setup() {
    const pendingGuess: Ref<string> = ref("");
    const usedWords: Ref<string[]> = ref([]);
    const currentGuess: Ref<string> = ref("");
    const guessedLetters: Ref<string[][]> = ref([]);
    const winStreak: Ref<number> = ref(0);
    const longestStreak: Ref<number> = ref(0);
    const totalWins: Ref<number> = ref(0);
    const totalLosses: Ref<number> = ref(0);
    const guessesPerWin: Ref<number[]> = ref([])

    const shakeWordGuess = ref(false);

    const getRandomIndex = (): number => {
      return Math.floor(Math.random() * fiveLetterWinningWords.length) + 1;
    }

    const currentWinningWord: Ref<string> = ref('');

    // Detect if device is mobile
    const isMobileDevice: Ref<boolean> = ref(false);
    
    const detectMobileDevice = (): boolean => {
      if (typeof window === 'undefined') return false;
      return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent) ||
             (window.matchMedia && window.matchMedia('(max-width: 768px)').matches && 'ontouchstart' in window);
    }

    const handleInputFocus = (event: Event) => {
      // Prevent keyboard from showing on mobile only
      const target = event.target as HTMLInputElement;
      if (target && isMobileDevice.value) {
        target.setAttribute('readonly', 'readonly');
      }
    }

    const handleInputBlur = (event: Event) => {
      // Immediately refocus to keep input always focused (mobile only)
      const target = event.target as HTMLInputElement;
      if (target && !isGuessingComplete.value && isMobileDevice.value) {
        setTimeout(() => {
          target.focus();
        }, 0);
      }
    }

    const handleInputClick = (event: Event) => {
      // Prevent default behavior and ensure focus (mobile only)
      if (isMobileDevice.value) {
        event.preventDefault();
      }
      const target = event.target as HTMLInputElement;
      if (target && isMobileDevice.value) {
        target.focus();
      }
    }

    const setInputFocus = () => {
      const inputField = document.getElementById("pending-guess") as HTMLInputElement;

      if (inputField) {
        // Set initial focus (can also be done with autofocus attribute)
        inputField.focus();
        
        // Ensure readonly is set to prevent keyboard (mobile only)
        if (isMobileDevice.value) {
          inputField.setAttribute('readonly', 'readonly');
        }
      }
    }

    const updateCache = (updateKey: string, updateValue: string|number|null|string[][] = null) => {
      const currentSession = localStorage.getItem('currentSession') || '{}'
      const parsedSession = JSON.parse(currentSession)
      const updatedSessionObj = {...parsedSession}

      switch (updateKey) {
        case "pendingGuess":
          if (updateValue === 'clear') {
            updatedSessionObj[updateKey] = ''
          } else {
            // If less than 5 chars, add letter to pending guess
            if (pendingGuess.value.length <= 5) {
              updatedSessionObj[updateKey] = updateValue
            }
          }
          break;
        case "usedWords":
        case "guessesPerWin":
          if (updateValue === 'clear') {
            // Clear array
            updatedSessionObj[updateKey] = []
          } else {
            // Add value to cached array
            updatedSessionObj[updateKey] = [...parsedSession[updateKey], updateValue]
          }
          break;
        case "guessedLetters":
          if (updateValue === 'clear') {
            updatedSessionObj[updateKey] = []
          } else {
            updatedSessionObj[updateKey] = updateValue
          }
          break;
        case "winStreak":
        case "totalWins":
        case "totalLosses":
        case "longestStreak":
          // Clear or add 1
          if (updateValue === 'clear') {
            updatedSessionObj[updateKey] = 0
          } else {
            updatedSessionObj[updateKey] = updatedSessionObj[updateKey] + 1
          }
          break;
        case "currentGuess":
        case "currentWinningWord":
          updatedSessionObj[updateKey] = updateValue
          break;
          default:
            console.log('Something went wrong with updating cache')
      }

      localStorage.setItem('currentSession', JSON.stringify(updatedSessionObj))
    }

    const typeLetter = (letter: string) => {
      if (pendingGuess.value.length < 5) {
        pendingGuess.value = pendingGuess.value + letter
      }
      // Refocus input after typing letter (for laptops/desktop)
      if (!isMobileDevice.value) {
        setTimeout(() => {
          const inputField = document.getElementById("pending-guess") as HTMLInputElement;
          if (inputField && !isGuessingComplete.value) {
            inputField.focus();
          }
        }, 0);
      }
    }

    const backspace = () => {
      if (pendingGuess.value.length) {
        pendingGuess.value = pendingGuess.value.slice(0, -1)
      }
      // Refocus input after backspace (for laptops/desktop)
      if (!isMobileDevice.value) {
        setTimeout(() => {
          const inputField = document.getElementById("pending-guess") as HTMLInputElement;
          if (inputField && !isGuessingComplete.value) {
            inputField.focus();
          }
        }, 0);
      }
    }

    const setCurrentGuess = (): void => {
      if (isWordValid.value) {
        currentGuess.value = pendingGuess.value.toLowerCase();
        updateCache('currentGuess', currentGuess.value)

        usedWords.value = [...usedWords.value, pendingGuess.value.toLowerCase()];
        guessedLetters.value = usedWords.value.map((usedWord) => {
          return usedWord.split("").map((letter, letterIndex) => {
            return [letter, colorsForWord(usedWord, letterIndex)];
          });
        }).flat();
        updateCache('guessedLetters', guessedLetters.value)
        

        // Guess is CORRECT!
        if (isCurrentGuessCorrect.value) {
          winStreak.value = winStreak.value + 1;
          updateCache('winStreak')
          totalWins.value = totalWins.value + 1;
          updateCache('totalWins')
          guessesPerWin.value = [...guessesPerWin.value, usedWords.value.length]
          updateCache('guessesPerWin', usedWords.value.length)
          if (winStreak.value > longestStreak.value) {
            longestStreak.value = longestStreak.value + 1
            updateCache('longestStreak')
          }
        }

        pendingGuess.value = "";
        updateCache('usedWords', currentGuess.value)
        
        // Refocus input after submitting guess (for laptops/desktop)
        if (!isMobileDevice.value) {
          setTimeout(() => {
            const inputField = document.getElementById("pending-guess") as HTMLInputElement;
            if (inputField && !isGuessingComplete.value) {
              inputField.focus();
            }
          }, 0);
        }
      } else {
        shakeWordGuess.value = true;
        setTimeout(() => {
          shakeWordGuess.value = false;
        }, 500)
        
        // Refocus input even if guess was invalid (for laptops/desktop)
        if (!isMobileDevice.value) {
          setTimeout(() => {
            const inputField = document.getElementById("pending-guess") as HTMLInputElement;
            if (inputField && !isGuessingComplete.value) {
              inputField.focus();
            }
          }, 0);
        }
      }
    }

    const setNewWinningWord = (): void => {
      currentWinningWord.value = fiveLetterWinningWords[getRandomIndex()];
      updateCache('currentWinningWord', currentWinningWord.value)
    }

    const colorsForWord = (word: string, letterIndex: number): string => {
      const guessedLetters: string[] = word.split("");
      const winningLetters: string[] = currentWinningWord.value.split("");
      if (word === "") {
        return "";
      }
      return guessedLetters.map((letter, index) => {
        return letter === winningLetters[index]
          ? "green"
          : winningLetters.includes(letter)
          ? "yellow"
          : "black";
      })[letterIndex];
    }

    const changeWordClearBoard = (): void => {
      if (!isCurrentGuessCorrect.value) {
        winStreak.value = 0;
        updateCache('winStreak', 'clear')
        totalLosses.value = totalLosses.value + 1
        updateCache('totalLosses')
      }

      pendingGuess.value = "";
      updateCache('pendingGuess', 'clear')

      usedWords.value = [];
      updateCache('usedWords', 'clear')

      currentWinningWord.value = fiveLetterWinningWords[getRandomIndex()];
      updateCache('currentWinningWord', currentWinningWord.value)
      
      currentGuess.value = "";

      guessedLetters.value = [];
      updateCache('guessedLetters', 'clear')

      setInputFocus()
    }

    const isWordValid: ComputedRef<boolean> = computed(() => {
      return (
        isFiveLetterWord(pendingGuess.value.toLowerCase()) &&
        isWordPlayable(pendingGuess.value.toLowerCase()) &&
        !usedWords.value.includes(pendingGuess.value.toLowerCase())
      );
    })

    const isGuessingComplete: ComputedRef<boolean> = computed(() => {
      return usedWords.value.length === 6 || isCurrentGuessCorrect.value;
    })

    const isCurrentGuessCorrect: ComputedRef<boolean> = computed(() => {
      if (currentGuess.value.length != 5) {
        return false;
      }
      return currentGuess.value.toLowerCase() === currentWinningWord.value;
    })

    const winLossPercentage = computed(() => {
      if (totalWins.value === 0) {
        return 0;
      } else if (totalWins.value === totalWins.value + totalLosses.value) {
        return 100
      }

      return Math.round((totalWins.value / (totalWins.value + totalLosses.value)) * 100)
    })

    watch(pendingGuess, () => {
      updateCache('pendingGuess', pendingGuess.value)
    })

    watch(isGuessingComplete, (newValue) => {
      // When game completes, stop auto-focusing. When new game starts, resume focus (both mobile and desktop)
      const inputField = document.getElementById("pending-guess") as HTMLInputElement;
      if (inputField && !newValue) {
        setTimeout(() => {
          inputField.focus();
        }, 100);
      }
    })

    onMounted(() => {
      // Detect mobile device
      isMobileDevice.value = detectMobileDevice();
      
      const inputField = document.getElementById("pending-guess") as HTMLInputElement;
      const body = document.getElementsByTagName('body')[0];

      if (inputField) {
        // Set readonly and inputmode to prevent keyboard (mobile only)
        if (isMobileDevice.value) {
          inputField.setAttribute('readonly', 'readonly');
          inputField.setAttribute('inputmode', 'none');
        }
        
        // Set initial focus
        setTimeout(() => {
          inputField.focus();
        }, 100);
        
        // Re-focus the input field whenever it loses focus (mobile only)
        if (isMobileDevice.value) {
          inputField.addEventListener("blur", function() {
            if (!isGuessingComplete.value) {
              setTimeout(() => {
                inputField.focus();
              }, 0);
            }
          });
          
          // Keep input focused on any click (mobile only)
          body.addEventListener("click", function(e) {
            // Don't refocus if clicking on keyboard buttons
            const target = e.target as HTMLElement;
            if (!target.closest('.keyboard') && !target.closest('.enter-button') && !target.closest('.backspace-button')) {
              setTimeout(() => {
                inputField.focus();
              }, 0);
            }
          });

          // Prevent keyboard on touch devices
          inputField.addEventListener('touchstart', function(e) {
            e.preventDefault();
            inputField.focus();
          });

          inputField.addEventListener('touchend', function(e) {
            e.preventDefault();
            inputField.focus();
          });
        } else {
          // For laptops/desktop: refocus after blur
          inputField.addEventListener("blur", function() {
            if (!isGuessingComplete.value) {
              setTimeout(() => {
                inputField.focus();
              }, 0);
            }
          });
          
          // For laptops/desktop: refocus after any click (including keyboard buttons)
          body.addEventListener("click", function(e) {
            // Don't refocus if clicking on modal
            const target = e.target as HTMLElement;
            if (!target.closest('.modal-container') && 
                !target.closest('.end-game-modal')) {
              // Refocus after a short delay to allow button click handlers to complete
              setTimeout(() => {
                if (!isGuessingComplete.value) {
                  inputField.focus();
                }
              }, 10);
            }
          });
        }
      }
      // Retrieving data
      const currentSession: any = localStorage.getItem('currentSession');
      const parsedSession = JSON.parse(currentSession);

      if (!parsedSession) {
        // Saving data
        localStorage.setItem('currentSession', JSON.stringify({
          pendingGuess: '',
          currentGuess: '',
          usedWords: [],
          winStreak: 0,
          longestStreak: 0,
          currentWinningWord: '',
          totalWins: 0,
          totalLosses: 0,
          isCurrentGuessCorrect: false,
          guessesPerWin: [],
          guessedLetters: []
        }));
        setNewWinningWord();
      } else {
        console.log('parsedSession', parsedSession)
        pendingGuess.value = parsedSession.pendingGuess
        currentGuess.value = parsedSession.currentGuess
        usedWords.value = parsedSession.usedWords
        winStreak.value = parsedSession.winStreak
        longestStreak.value = parsedSession.longestStreak
        currentWinningWord.value = parsedSession.currentWinningWord
        totalWins.value = parsedSession.totalWins
        totalLosses.value = parsedSession.totalLosses
        guessesPerWin.value = parsedSession.guessesPerWin
        guessedLetters.value = parsedSession.guessedLetters
      }
    })

    return {
      backspace,
      shakeWordGuess,
      updateCache,
      typeLetter,
      setCurrentGuess,
      setNewWinningWord,
      changeWordClearBoard,
      isCurrentGuessCorrect,
      isGuessingComplete,
      guessesPerWin,
      usedWords,
      pendingGuess,
      currentGuess,
      guessedLetters,
      isWordValid,
      longestStreak,
      totalWins,
      totalLosses,
      winLossPercentage,
      winStreak,
      currentWinningWord,
      setInputFocus,
      handleInputFocus,
      handleInputBlur,
      handleInputClick,
      isMobileDevice,
    };
  },
});
</script>

<style>
html {
  background-color: rgb(62, 172, 62);
}

#app {
  height: 100vh;
  min-width: 100%;
  min-height: 100%;
}

#pending-guess {
  outline: none;
  background-color: transparent;
  margin-left: 6px;
  margin-top: 6px;
}

#pending-guess:focus {
  outline: none;
}

@media screen and (max-width: 480px) {
  #pending-guess {
    caret-color: transparent;
    color: transparent;
    text-shadow: 0 0 0 transparent;
  }

  #pending-guess::selection {
    background: transparent;
  }

  #pending-guess::-moz-selection {
    background: transparent;
  }
}

.guess-form {
  justify-self: flex-start;
  position: absolute;
  z-index: 1;
}

.app-container {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  margin: 0;
  height: 100%;
}

.letter-input {
  max-width: 19%;
}

.letter-input-container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  max-width: 100%;
  z-index: 1;
}

/* Modal transition animations */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
  transform: scale(0.95);
}

.modal-enter-to,
.modal-leave-from {
  opacity: 1;
  transform: scale(1);
}
</style>
