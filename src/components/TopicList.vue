<template>
  <div class="topic-list">
    <!-- Loop through each topic in the topics array -->
    <li v-for="(topic, index) in topics" :key="topic.guid" :class="{ listOpen: topic.isOpen, listClose: !topic.isOpen }">
      <div>
        <div v-if="topic.isEditing" class="flex-between">
          <input class="topic-input" type="text" v-model="topic.name" />
          <div class="action-list edit">
            <button @click="saveTopic(index)">Save</button>
            <button @click="cancelEdit(index)">Cancel</button>
          </div>
        </div>
        <h2 v-else class="flex-between">
          <span @click="toggleTopic(topic)">{{ topic.name }}</span>
          <div class="action-list">
            <EditIcon @click="editTopic(index)" />
            <DeleteIcon @click="deleteTopic(index)" />
            <ArrowIcon :type="topic.isOpen ? 'down' : 'up'" @click="toggleTopic(topic)" />
          </div>
        </h2>
      </div>
      <ul class="comment-list" v-if="topic.isOpen">
        <!-- Loop through each comment in the topic's comments array -->
        <li class="region-child" v-for="(comment, commentIndex) in topic.comments" :key="comment.guid">
          <div class="comment-wrapper" v-if="comment.isEditing">
            <span class="by">{{ comment.by }}</span>
            <div class="comment">
              <div class="by-name">{{ byName(comment.by) }}</div>
              <EditableDiv v-model="comment.comment" />
            </div>
            <button @click="saveComment(index, commentIndex)">Save</button>
            <button @click="cancelEditComment(index, commentIndex)">
              Cancel
            </button>
          </div>
          <div v-else class="comment-wrapper">
            <span class="by">{{ comment.by }}</span>
            <div class="comment">
              <div class="by-name">{{ byName(comment.by) }}</div>
              <div class="comment-content">{{ comment.comment }}</div>
              <span class="comment-date">{{ formatDate(comment.date) }}</span>
            </div>
            <div class="action-list">
              <EditIcon @click="editComment(index, commentIndex)" />
              <DeleteIcon @click="deleteComment(index, commentIndex)" />
            </div>
          </div>
        </li>
        <!-- <span v-if="topic.comments.length <= 0">No comments to show</span> -->
        <div class="comment-wrapper">
          <span class="by">AA</span>
          <EditableDiv class="add-comment" v-model="newComments[index]" @keydown.enter.prevent="addComment(index)" />
          <SendIcon class="self-center" @click="addComment(index)" />
        </div>
      </ul>
    </li>
  </div>
</template>

<script setup>
import ArrowIcon from "./Icons/ArrowIcon.vue";
import DeleteIcon from "./Icons/DeleteIcon.vue";
import EditIcon from "./Icons/EditIcon.vue";
import SendIcon from "./Icons/SendIcon.vue";
import EditableDiv from "./EditableDiv.vue";
import { ref, defineEmits } from "vue";

// Define props
const props = defineProps({
  topics: Array,
  persons: Array,
});

const byName = (by) => {
  const person = props.persons.find((person) => person.guid === by);
  return person ? person.first + " " + person.last : by === 'AA' ? 'Admin' : by; // return by if not found, return Admin if AA
};

const emit = defineEmits([
  "delete-topic",
  "update-topic",
  "edit-topic",
  "cancel-edit",
  "delete-comment",
  "update-comment",
  "edit-comment",
  "cancel-edit-comment",
  "add-comment"
]);

const newComments = ref(props.topics.map(() => "")); // Initialize an array to keep track of new comment texts

const addComment = (topicIndex) => {
  if (newComments.value[topicIndex].trim() !== "") {
    emit("add-comment", { topicIndex, comment: newComments.value[topicIndex] });
    newComments.value[topicIndex] = ""; // Clear the input field after submitting
  }
};

const formatDate = (dateString) => {
  const dateObj = new Date(dateString);

  const options = { year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: true };
  const beautifulDate = dateObj.toLocaleDateString('en-US', options);
  return beautifulDate;
}

const deleteTopic = (index) => {
  emit("delete-topic", index);
};

const editTopic = (index) => {
  emit("edit-topic", index);
};

const saveTopic = (index) => {
  emit("update-topic", index);
};

const cancelEdit = (index) => {
  emit("cancel-edit", index);
};

// Comments Crud functions
const deleteComment = (topicIndex, commentIndex) => {
  emit("delete-comment", { topicIndex, commentIndex });
};

const saveComment = (topicIndex, commentIndex) => {
  emit("update-comment", { topicIndex, commentIndex });
};

const editComment = (topicIndex, commentIndex) => {
  emit("edit-comment", { topicIndex, commentIndex });
};

const cancelEditComment = (topicIndex, commentIndex) => {
  emit("cancel-edit-comment", { topicIndex, commentIndex });
};

// Method to toggle the open/close state of a topic
const toggleTopic = (topic) => {
  topic.isOpen = !topic.isOpen;
};
</script>