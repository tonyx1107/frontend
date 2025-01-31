<script setup lang="ts">
import { useUserStore } from "@/stores/user";
import { formatDate } from "@/utils/formatDate";
import { storeToRefs } from "pinia";
import { fetchy } from "../../utils/fetchy";
import { ref, onMounted } from "vue";
import CommentComponent from "./CommentComponent.vue";
import CreateCommentForm from "./CreateCommentForm.vue";

const props = defineProps(["post"]);
const emit = defineEmits(["editPost", "refreshPosts"]);
const { currentUsername } = storeToRefs(useUserStore());
const comments = ref<Array<Record<string, string>>>([]);

const deletePost = async () => {
  try {
    await fetchy(`/api/posts/${props.post._id}`, "DELETE");
  } catch {
    return;
  }
  emit("refreshPosts");
};

async function getComments(rootId?: string) {
  let commentResults;
  let query: Record<string, string> = rootId ? { rootId } : {};
  console.log(query);
  try {
    commentResults = await fetchy(`/api/discussion/`, "GET", { query });
  } catch {
    return;
  }
  comments.value = commentResults;
}

async function processCommentAdded() {
  await getComments(props.post._id); // Refresh the comments
}

function processCommentDeleted(commentId: string) {
  comments.value = comments.value.filter((comment) => comment._id !== commentId); // Remove the deleted comment
}

onMounted(async () => {
  await getComments(props.post._id);
});
</script>

<template>
  <p class="author">{{ props.post.author }}</p>
  <p>{{ props.post.content }}</p>
  <div class="base">
    <menu v-if="props.post.author == currentUsername">
      <li><button class="btn-small pure-button" @click="emit('editPost', props.post._id)">Edit</button></li>
      <li><button class="button-error btn-small pure-button" @click="deletePost">Delete</button></li>
    </menu>
    <article class="timestamp">
      <p v-if="props.post.dateCreated !== props.post.dateUpdated">Edited on: {{ formatDate(props.post.dateUpdated) }}</p>
      <p v-else>Created on: {{ formatDate(props.post.dateCreated) }}</p>
    </article>
  </div>
  <div class="comments">
    <CommentComponent v-for="comment in comments" :key="comment._id" :comment="comment" @commentDeleted="processCommentDeleted" />
  </div>
  <CreateCommentForm :post="props.post" @commentAdded="processCommentAdded" />
</template>

<style scoped>
p {
  margin: 0em;
}

.author {
  font-weight: bold;
  font-size: 1.2em;
}

menu {
  list-style-type: none;
  display: flex;
  flex-direction: row;
  gap: 1em;
  padding: 0;
  margin: 0;
}

.timestamp {
  display: flex;
  justify-content: flex-end;
  font-size: 0.9em;
  font-style: italic;
}

.base {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.base article:only-child {
  margin-left: auto;
}
</style>
