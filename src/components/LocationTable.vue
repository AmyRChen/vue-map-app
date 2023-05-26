<template>
  <div>
    <button class="delete-button" @click="deleteSelectedHistory">Delete Selected History</button>
    <table>
      <thead>
        <tr>
          <th>Select</th>
          <th>Search History</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="location in paginatedLocations" :key="location">
          <!--When you use v-model on an input element or a form component, 
          such as checkboxes, radio buttons, or select dropdowns, Vue.js 
          automatically sets up the necessary event listeners and updates 
          the data property based on user input.-->

          <td>
            <input type="checkbox" v-model="selectedLocations" :value="location" />
          </td>
          <td>{{ location }}</td>
        </tr>
      </tbody>
    </table>

    <div class="pagination">
      <button @click="previousPage">Previous</button>
      <span>{{ currentPage }}/{{ totalPages }}</span>
      <button @click="nextPage">Next</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    locations: {
      type: Array,
      required: true,
    },
    // future maintain
    pageSize: {
      type: Number,
      default: 10,
    },
  },
  data() {
    return {
      currentPage: 1,
      selectedLocations: [], // New data property to store selected locations
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.locations.length / this.pageSize);
    },
    paginatedLocations() {
      const startIndex = (this.currentPage - 1) * this.pageSize;
      const endIndex = startIndex + this.pageSize;
      return this.locations.slice(startIndex, endIndex);
    },
  },
  methods: {
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
    deleteSelectedHistory() {
      // event: 'delete-selected'
      this.$emit('delete-selected', this.selectedLocations);
      // Clear the selectedLocations array to reset the checkboxes
      this.selectedLocations = [];
    },
  },
};
</script>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 10px;
}

th,
td {
  padding: 8px;
  border-bottom: 1px solid #ddd;
}

.pagination {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 10px;
}

.pagination button {
  margin: 0 5px;
}
</style>