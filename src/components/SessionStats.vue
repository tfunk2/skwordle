<template>
  <section class="stats-section">
    <div class="stats-container" id="current-streak">
      <p class="streak-text">CURRENT</p>
      <p class="streak-text">STREAK</p>
      <b class="streak-number-text">{{ winStreak }}</b>
    </div>
    <div class="stats-container">
      <p class="streak-text">WIN</p>
      <p class="streak-text">PERCENT</p>
      <b class="streak-number-text">{{ winLossPercentage }}%</b>
    </div>
    <div class="stats-container" id="longest-streak">
      <p class="streak-text">LONGEST</p>
      <p class="streak-text">STREAK</p>
      <b class="streak-number-text">{{ longestStreak }}</b>
    </div>
  </section>
</template>

<script lang='ts'>
import { defineComponent } from "vue";

export default defineComponent({
  name: "SessionStats",
  props: {
    winStreak: {
      type: Number,
      required: true,
    },
    longestStreak: {
      type: Number,
      required: true,
    },
    winLossPercentage: {
      type: Number,
      required: true,
    },
  },
  data() {
    return {};
  },
  computed: {},
  methods: {},
});
</script>

<style scoped>
.streak-text {
  color: white;
  font-family: "Bungee Hairline", sans-serif;
  margin: 0px;
  line-height: 0.8;
  padding-bottom: 2px;
  font-weight: bold;
}

.streak-number-text {
  color: white;
  font-family: "Bungee Hairline", sans-serif;
  font-size: 36px;
  margin-top: 0px;
  line-height: 0.7
}

.stats-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  outline: 1px white solid;
  padding-top: 8px;
  padding-left: 15px;
  padding-right: 15px;
  padding-bottom: 6px;
}

.stats-section {
  display: flex;
  margin-top: 16px;
  margin-bottom: 16px;
}

#current-streak {
  border-radius: 4px 0px 0px 4px;
}

#longest-streak {
  border-radius: 0px 4px 4px 0px;
}

@media screen and (max-width: 480px) {
  .streak-text {
    padding-bottom: 1px;
    font-size: 0.9em;
  }

  .streak-number-text {
    font-size: 28px;
    line-height: 0.75;
  }

  .stats-container {
    padding-top: 6px;
    padding-bottom: 4px;
  }

  .stats-section {
    margin-top: 8px;
    margin-bottom: 8px;
  }
}
</style>
