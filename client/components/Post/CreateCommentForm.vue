<script setup lang="ts">
import { ref } from "vue";
import { fetchy } from "../../utils/fetchy";
import { useUserStore } from "@/stores/user";
import { storeToRefs } from "pinia";

const props = defineProps(["post"]);
const emit = defineEmits(["commentAdded"]);

const { currentUsername } = storeToRefs(useUserStore());
const newComment = ref("");

// Function to handle comment submission
const submitComment = async () => {
  if (newComment.value.trim() === "") return; // Prevent empty comments

  try {
    const response = await fetchy(`/api/discussion`, "POST", {
      body: {
        root: props.post._id,
        content: newComment.value,
      },
    });

    emit("commentAdded", response); // Emit an event with the new comment
    emptyForm(); // Clear the input
  } catch (error) {
    console.error("Error submitting comment:", error);
  }
};

// Function to clear the input
const emptyForm = () => {
  newComment.value = "";
};
</script>

<template>
  <form @submit.prevent="submitComment">
    <label for="commentContent">Add a comment:</label>
    <textarea id="commentContent" v-model="newComment" placeholder="Add a comment..." required></textarea>
    <button type="submit" class="pure-button-primary pure-button">Submit</button>
  </form>
</template>

<style scoped>
form {
  background-color: var(--base-bg);
  border-radius: 1em;
  display: flex;
  flex-direction: column;
  gap: 0.5em;
  padding: 1em;
}

textarea {
  font-family: inherit;
  font-size: inherit;
  height: 4em;
  padding: 0.5em;
  border-radius: 4px;
  resize: none;
  border: 1px solid #ccc;
}

button {
  align-self: flex-end;
}
</style>
