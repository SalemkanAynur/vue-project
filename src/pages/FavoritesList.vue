<template>
  <div class="favorite-page">
    <h1>Favorite Jokes</h1>
    <ul>
      <li v-for="(joke, index) in favoriteJokes" :key="joke.id">
        {{ index + 1 }}. {{ joke.value }}
        <button class="btn" @click="removeFavorites(joke.id)">Удалить</button>
      </li>
    </ul>
  </div>
</template>
<script lang="ts">
  import { reactive, toRefs } from "vue";

  interface Joke {
    id: string;
    value: string;
  }

  interface State {
    favoriteJokes: Joke[];
  }

  export default {
    setup() {
      const state = reactive<State>({
        favoriteJokes: []
      });

      const loadFavoritesFromStorage = () => {
        const favorites = JSON.parse(localStorage.getItem("favoriteJokes"));
        if (Array.isArray(favorites)) {
          state.favoriteJokes = favorites;
        }
      };
      loadFavoritesFromStorage();

      const removeFavorites = (id: string) => {
        const index = state.favoriteJokes.findIndex((joke) => joke.id === id);
        if (index > -1) {
          state.favoriteJokes.splice(index, 1);
          localStorage.setItem("favoriteJokes", JSON.stringify(state.favoriteJokes));
        }
      };

      return {
        ...toRefs(state),
        removeFavorites
      };
    }
  };
</script>
<style>
  .favorite-page {
    display: flex;
    flex-direction: column;
    grid-gap: 20px;
    padding: 10px 30px;
  }

  .btn {
    background: #F7AB05;
    border-radius: 48px;
    border: none;
    width: 120px;
    height: 30px;
    color: white;
    padding: 5px 5px;
    text-align: center;
    text-decoration: none;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
  }
</style>
