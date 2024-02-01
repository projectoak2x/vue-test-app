<template>
  <div class="container">
    <div class="flex-center">
      <img alt="Vue logo" src="./assets/logo.png" height="70px" />
      <Title :msg="title" />
    </div>
    <div class="search-container">
      <input v-model="state.searchQuery" type="text" placeholder="Search topics..." @input="searchTopics" />
      <div>
        <button @click="toggleModal(true)">Create Topic</button>
      </div>

      <div class="overlay" v-if="state.showModal" @click="toggleModal(false)"></div>
      <div class="modal" v-if="state.showModal">
        <h3>Add Topic Name</h3>
        <input type="text" maxlength="50" v-model="state.newTopic" @keydown.enter.prevent="handleAddNewTopic" />
        <div class="button-container">
          <button class="mr-10" @click="handleAddNewTopic">submit</button>
          <button @click="toggleModal(false)">cancel</button>
        </div>

      </div>



    </div>
    <TopicList :topics="filteredPaginatedTopics" :persons="state.persons" @delete-topic="deleteTopic"
      @update-topic="updateTopic" @edit-topic="editTopic" @cancel-edit="cancelEdit" @delete-comment="deleteComment"
      @update-comment="updateComment" @edit-comment="editComment" @cancel-edit-comment="cancelEditComment"
      @add-comment="addComment" />
    <div class="pagination">
      <button @click="prevPage" :disabled="state.currentPage === 1">
        Prev
      </button>
      <span>Page {{ state.currentPage }} of {{ totalPages }}</span>
      <button @click="nextPage" :disabled="state.currentPage >= totalPages">
        Next
      </button>
    </div>
  </div>
</template>

<script setup>
import { reactive, computed, onMounted } from "vue";
import Title from "./components/Title.vue";
import TopicList from "./components/TopicList.vue";

const title = "Simple Feeds App";
const state = reactive({
  topics: [], // This will hold the data fetched from the API
  persons: [],
  currentPage: 1,
  itemsPerPage: 20,
  searchQuery: '',
  showModal: false,
  newTopic: ''
});

const fetchData = async () => {
  try {
    const response = await fetch(
      "https://atillc.blob.core.windows.net/data-collector/icode/test-data/topics.json"
    );
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const fetchedData = await response.json(); // Fetch and parse the JSON
    state.topics = fetchedData.topics; // Assuming the fetched data has a 'topics' property
    state.persons = fetchedData.persons; // Assuming the fetched data has a 'persons' property
  } catch (error) {
    console.error("Fetch error: " + error.message);
  }
};

const handleAddNewTopic = () => {
  const newTopicObject = {
    name: state.newTopic,
    guid: Date.now(),
    comments: []
  }
  state.topics.unshift(newTopicObject);
  state.newTopic = '';
  state.showModal = false;
  // Optionally, handle sending the new topic to the server here
};

const toggleModal = (show) => {
  state.showModal = show;
}

const addComment = ({ topicIndex, comment }) => {
  const start = (state.currentPage - 1) * state.itemsPerPage;
  const newComment = {
    id: Date.now(), // Or generate a unique ID in your preferred way
    comment,
    date: Date.now(),
    by: "AA"
    // any other necessary properties
  };
  state.topics[start + topicIndex].comments.push(newComment);
  // Optionally, handle sending the new comment to the server here
};

const editTopic = (index) => {
  const start = (state.currentPage - 1) * state.itemsPerPage;
  state.topics[start + index].isEditing = true;
};

const updateTopic = (index) => {
  const start = (state.currentPage - 1) * state.itemsPerPage;
  state.topics[start + index].isEditing = false;
  // Here you might want to send the updated topic to the server
};

const cancelEdit = (index) => {
  const start = (state.currentPage - 1) * state.itemsPerPage;
  state.topics[start + index].isEditing = false;
  // Here you might want to reset the topic name if you're storing the original somewhere
};

const deleteTopic = (index) => {
  if (window.confirm("Are you sure you want to delete this topic?")) {
    const start = (state.currentPage - 1) * state.itemsPerPage;
    state.topics.splice(start + index, 1);
  }
};

const deleteComment = ({ topicIndex, commentIndex }) => {
  const start = (state.currentPage - 1) * state.itemsPerPage;
  state.topics[start + topicIndex].comments.splice(commentIndex, 1);
};

const updateComment = ({ topicIndex, commentIndex }) => {
  const start = (state.currentPage - 1) * state.itemsPerPage;
  state.topics[start + topicIndex].comments[commentIndex].isEditing = false;
  // Handle updating the comment in the server if needed
};

const editComment = ({ topicIndex, commentIndex }) => {
  const start = (state.currentPage - 1) * state.itemsPerPage;
  state.topics[start + topicIndex].comments[commentIndex].isEditing = true;
};

const cancelEditComment = ({ topicIndex, commentIndex }) => {
  const start = (state.currentPage - 1) * state.itemsPerPage;
  state.topics[start + topicIndex].comments[commentIndex].isEditing = false;
  // Handle resetting the comment content if you're storing the original somewhere
};

const filteredTopics = computed(() => {
  if (!state.searchQuery) {
    return state.topics;
  }
  return state.topics.filter((topic) =>
    topic.name.toLowerCase().includes(state.searchQuery.toLowerCase())
  );
});

const filteredPaginatedTopics = computed(() => {
  const start = (state.currentPage - 1) * state.itemsPerPage;
  const end = start + state.itemsPerPage;
  return filteredTopics.value.slice(start, end);
});

const totalPages = computed(() => {
  return Math.ceil(filteredTopics.value.length / state.itemsPerPage);
});

const nextPage = () => {
  if (state.currentPage < totalPages.value) {
    state.currentPage++;
  }
};

const prevPage = () => {
  if (state.currentPage > 1) {
    state.currentPage--;
  }
};

onMounted(fetchData);
</script>
