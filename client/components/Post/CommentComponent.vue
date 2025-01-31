<script setup lang="ts">
import { useUserStore } from "@/stores/user";
import { formatDate } from "@/utils/formatDate";
import { fetchy } from "@/utils/fetchy";
import { storeToRefs } from "pinia";

const props = defineProps(["comment", "postId"]);
const emit = defineEmits(["editComment", "commentDeleted"]);
const { currentUsername } = storeToRefs(useUserStore());

const deleteComment = async () => {
  try {
    await fetchy(`/api/discussion/${props.comment._id}`, "DELETE");
    emit("commentDeleted", props.comment._id); // Notify parent that comment was deleted
  } catch (error) {
    console.error("Failed to delete comment:", error);
  }
};
</script>

<template>
  <div class="comment-container">
    <p class="author">{{ props.comment.author }}</p>
    <p class="commentContent">{{ props.comment.content }}</p>
    <div class="comment-actions">
      <menu v-if="props.comment.author === currentUsername">
        <li>
          <button class="button-error btn-small pure-button" @click="deleteComment">Delete</button>
        </li>
      </menu>
      <div class="timestamp">Posted on: {{ formatDate(props.comment.dateCreated) }}</div>
    </div>
  </div>
</template>

<style scoped>
.comment-container {
  border-top: 1px solid #ddd;
  padding: 0.5em 1em;
}

.author {
  font-weight: bold;
  font-size: 0.8em;
  margin-bottom: 0.1em;
}

.commentContent {
  font-size: 0.8em;
  margin-bottom: 0.1em;
}

menu {
  list-style-type: none;
  display: flex;
  flex-direction: row;
  gap: 0.5em;
  padding: 0;
  margin: 0;
}

.comment-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.timestamp {
  font-size: 0.65em;
  font-style: italic;
  margin-left: auto;
}

.timestamp p {
  margin: 0;
}
</style>
