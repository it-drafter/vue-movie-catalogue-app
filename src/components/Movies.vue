<script setup>
import { ref } from "vue";

const delMovieError = ref(false);
const selectedButton = ref(false);

const props = defineProps({
  element: Object,
});

const emit = defineEmits(["onDeleteMovie"]);

async function deleteThisMoviesHandler(movieID) {
  const delMethod = {
    method: "DELETE",
    headers: {
      "Content-Type": "application/json",
    },
  };

  try {
    const response = await fetch(
      `https://vue-movies-app-aa461-default-rtdb.europe-west1.firebasedatabase.app/movie/${movieID}.json`,
      delMethod
    );
    if (!response.ok) {
      throw new Error("Something went wrong!");
    }
    const data = await response.json();
    emit("onDeleteMovie");
  } catch (err) {
    delMovieError.value = err;
  }
}
</script>

<template>
  <ul class="list-group">
    <li
      v-for="(movie, index) in Object.entries(props.element)"
      :key="movie[0]"
      class="list-group-item"
    >
      <strong>Title:</strong> {{ movie[1].title || "[no title!]" }} -
      <strong>Release date:</strong>
      {{ movie[1].release_date || "[no release date!]" }} -
      <strong>Director:</strong> {{ movie[1].director || "[no director!]" }}
      <br />
      <button
        class="btn d-inline-block btn-outline-danger my-3"
        @click="
          ($event) => {
            deleteThisMoviesHandler(movie[0]);
            selectedButton = index;
          }
        "
      >
        Delete This Movie
      </button>
      <p
        class="d-inline-block text-danger err-del"
        :class="{ visible: selectedButton == index && delMovieError }"
      >
        &nbsp; {{ delMovieError }}
      </p>
    </li>
  </ul>
</template>

<style scoped>
.err-del {
  visibility: hidden;
}
.err-del.visible {
  visibility: visible;
}
</style>
