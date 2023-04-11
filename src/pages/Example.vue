<template>
  <div class="main-wrapper">
    <button class="button" @click="getJokes">GET JOKES</button>
    <p>{{ joke }}</p>

    <button class="button" @click="toggleUpdate">
      {{ autoUpdate ? "Stop Update" : "Update Jokes" }}
    </button>

    <p v-if="autoUpdate">{{ autoUpdatedJoke }}</p>

    <button class="button" @click="toggleFavorites">
      {{ isFavorites ? "Remove from favorites Jokes" : "Add to favorites Jokes" }}
    </button>

    <router-link to="/favorites" class="button"> Favorites Jokes</router-link>
  </div>
</template>

<script lang="ts">
  import { reactive, toRefs, watchEffect, ref } from "vue";
  import axios, { AxiosResponse } from "axios";

  interface Joke {
    id: string;
    value: string;
  }
  interface State {
    joke: string;
    autoUpdate: boolean;
    autoUpdatedJoke: string;
    isFavorites: boolean;
    favoriteJokes: Joke[] | null;
  }

  export default {
    setup() {
      const state: State = reactive({
        joke: "",
        autoUpdate: false,
        autoUpdatedJoke: "",
        isFavorites: false,
        favoriteJokes: JSON.parse(localStorage.getItem("favoriteJokes") ?? "null")
      });

      const getJokes = () => {
        axios
          .get("https://api.chucknorris.io/jokes/random")
          .then((response: AxiosResponse<Joke>) => {
            state.joke = response.data.value;
          })
          .catch((error: Error) => {
            console.log(error);
            console.log("bnm");
          });
      };

      const startAutoUpdate = () => {
        setInterval(() => {
          axios
            .get("https://api.chucknorris.io/jokes/random")
            .then((response: AxiosResponse<Joke>) => {
              state.autoUpdatedJoke = response.data.value;
            })
            .catch((error: Error) => {
              console.log(error);
            });
        }, 3000);
      };

      const stopAutoUpdate = () => {
        clearInterval(interval.value);
      };

      const interval: Ref<NodeJS.Timeout | null> = ref(null);

      watchEffect(() => {
        if (state.autoUpdate) {
          interval.value = startAutoUpdate();
        } else {
          stopAutoUpdate();
        }
      });

      const toggleUpdate = () => {
        state.autoUpdate = !state.autoUpdate;
      };

      const toggleFavorites = () => {
        if (state.isFavorites) {
          // удаляем шутку из списка избранных
          const index = state.favoriteJokes?.findIndex((joke) => joke.id === state.joke.id) ?? -1;
          if (index !== -1) {
            state.favoriteJokes?.splice(index, 1);
          }
        } else {
          // добавляем шутку в список избранных
          if (state.favoriteJokes === null) {
            state.favoriteJokes = [];
          }
          if (state.favoriteJokes.length >= 10) {
            state.favoriteJokes.shift();
          }
          state.favoriteJokes.push({ id: new Date().getTime().toString(), value: state.joke });
        }

        state.isFavorites = !state.isFavorites;
        localStorage.setItem("favoriteJokes", JSON.stringify(state.favoriteJokes));
      };

      return {
        ...toRefs(state),
        getJokes,
        toggleUpdate,
        toggleFavorites
      };
    }
  };
</script>
<style>
  .main-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    grid-gap: 20px;
  }
  .button {
    background-color: #4caf50;
    width: 400px;
    height: 60px;
    border: none;
    border-radius: 48px;
    color: white;
    padding: 20px 20px;
    text-align: center;
    text-decoration: none;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
  }
</style>
