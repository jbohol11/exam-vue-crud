<script setup>
  import axios from 'axios';
</script>

<template>
  <div class="container">
    <h1>Simple CRUD list</h1>
    <div>
      <input type="text" v-model="newTopicName" placeholder="Enter topic name">
      <button @click="addTopic(newTopicName)">Add topic</button>
    </div>
    <div class="pagination">
      <button @click="previousPage" :disabled="currentPage === 1">Previous</button>
      <span>{{ currentPage }} / {{ totalPages }}</span>
      <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
    </div>
    <div class="topic-container">
      <div v-for="(topic) in paginatedTopics" :key="topic.guid" class="topic-wrapper">
        <div class="topic">
          <h2 v-if="editedTopicGuid !== topic.guid">{{ truncateTopicName(topic.name) }}</h2>
          <input v-else type="text" v-model="newTopicName" @keyup.enter="saveEditedTopic" @blur="saveEditedTopic">
          <ul class="comment-list">
            <li class="single-comment" v-for="(comment, commentIndex) in topic.comments" :key="commentIndex">
              <div class="comment-info">
                <b>Comment by:</b> {{  comment.by }}
              </div>
              <div class="comment-info">
                <b>Date time:</b> {{  comment.date }}
              </div>
              <div class="comment-info">
                <b>Comment:</b> {{  comment.comment }}
              </div>
            </li>
          </ul>
          <div class="topic-actions">
            <button v-if="editedTopicGuid !== topic.guid" @click="editTopic(topic.guid)">Edit</button>
            <button v-else @click="saveEditedTopic">Save</button>
            <button @click="confirmDeleteTopic(topic.guid)">Delete</button>
          </div>
        </div>
      </div>
    </div>
    <div class="pagination">
      <button @click="previousPage" :disabled="currentPage === 1">Previous</button>
      <span>{{ currentPage }} / {{ totalPages }}</span>
      <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HomePage',
  data: () => ({
    fetchedData: '',
    topics: [],
    persons: [],
    newTopicName: '',
    hoveredTopic: null,
    editedTopicGuid: null, // Track the GUID of the topic being edited
    currentPage: 1,
    itemsPerPage: 20,
  }),
  mounted() {
    this.fetchData();
  },
  methods: {
    async fetchData() {
      try {
        const res = await axios.get('https://atillc.blob.core.windows.net/data-collector/icode/test-data/topics.json');
        if (res.status === 200) {
          this.fetchedData = res.data;
          this.topics = this.fetchedData.topics;
          this.persons = this.fetchedData.persons;
        }
      } catch (error) {
        console.log('error', error);
      }
    },
    addTopic(newTopic) {
      if (!newTopic.trim()) {
        alert('Please enter a topic name');
        return;
      }

      const topicObject = { guid: this.generateGUID(), name: newTopic, comments: [] };
      this.topics.unshift(topicObject); // Prepend new topic to the topics array
      this.newTopicName = ''; // Clear input field after adding topic
    },
    editTopic(guid) {
      // Set the GUID of the topic being edited
      this.editedTopicGuid = guid;
      // Find the index of the topic with the given GUID
      const index = this.topics.findIndex(topic => topic.guid === guid);
      // Set the new topic name as the current topic name for editing
      this.newTopicName = this.topics[index].name;
    },
    saveEditedTopic() {
      if (!this.newTopicName.trim()) {
        alert('Please enter a topic name');
        return;
      }
      // Find the index of the topic with the edited GUID
      const index = this.topics.findIndex(topic => topic.guid === this.editedTopicGuid);
      // Update the topic name with the new value using direct assignment
      this.topics[index].name = this.newTopicName;
      // Clear the new topic name and reset the edited topic GUID
      this.newTopicName = '';
      this.editedTopicGuid = null;
    },
    confirmDeleteTopic(guid) {
      if (confirm('Are you sure you want to delete this topic?')) {
        // Find the index of the topic with the given GUID
        const index = this.topics.findIndex(topic => topic.guid === guid);
        this.deleteTopic(index);
      }
    },
    deleteTopic(index) {
      console.log('Deleting topic', this.topics[index]);
      this.topics.splice(index, 1);
    },
    truncateTopicName(name) {
      return name.length > 20 ? name.substring(0, 20) + '...' : name;
    },
    handleTopicHover(name) {
      this.hoveredTopic = name;
    },
    generateGUID() {
      // Generate a unique identifier (GUID)
      return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
        var r = Math.random() * 16 | 0,
          v = c === 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
      });
    },
    previousPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
  },
  computed: {
    // Calculate total number of pages
    totalPages() {
      return Math.ceil(this.topics.length / this.itemsPerPage);
    },
    // Calculate the topics to display for the current page
    paginatedTopics() {
      const startIndex = (this.currentPage - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      return this.topics.slice(startIndex, endIndex);
    },
  },
};
</script>

<style scoped>
.container {
  margin: 0 auto;
  padding: 20px;
}

.topic-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.topic-wrapper {
  margin: 10px;
}

.topic {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
  width: 400px;
  height: 200px;
  overflow: hidden; /* Hide overflow content */
  position: relative; /* Position relative for absolute positioning */
}

.topic h2 {
  font-size: 1.5rem;
  margin-bottom: 10px;
  overflow: hidden; /* Hide overflow content */
  text-overflow: ellipsis; /* Show ellipsis for overflow text */
  white-space: nowrap; /* Prevent text wrapping */
}
.topic input[type="text"] {
  font-size: 1.5rem;
  margin-bottom: 10px;
  width: calc(100% - 10px); /* Adjust input width */
  overflow: hidden; /* Hide overflow content */
  text-overflow: ellipsis; /* Show ellipsis for overflow text */
  white-space: nowrap; /* Prevent text wrapping */
}

.topic-actions {
  position: absolute; /* Position absolute for bottom placement */
  bottom: 10px; /* Adjust as needed */
  left: 0; /* Align buttons to the left */
  width: 100%; /* Full width of the topic */
  text-align: center; /* Center buttons horizontally */
}

.topic-actions button {
  margin: 0 5px; /* Add margin between buttons */
}

.comment-list {
  list-style-type: none;
  padding: 0;
  width: 100%;
  max-height: 120px; /* Limit height for comments */
  overflow-y: auto; /* Enable vertical scrolling for overflow */
}

.single-comment {
  border: 1px solid #ddd;
  padding: 5px;
  margin: 10px;
}

.comment-info {
  font-size: 0.9rem;
  margin-bottom: 5px;
  text-align: left;
}

.comment-text {
  font-size: 1rem;
  margin-top: 5px;
  text-align: left;
}

.pagination {
  margin-top: 20px;
  text-align: center;
}

.pagination button {
  margin: 0 5px;
}
</style>
