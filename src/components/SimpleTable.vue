<template>
  <div class="container mx-auto p-4">
    <transition name="fade">
      <toast-errors :errors="errors" v-if="hasErrors"></toast-errors>
    </transition>
    <h1 class="text-2xl font-bold mb-4">{{ title }}</h1>
    <div class="flex flex-col gap-4 mb-7">
      <input
        v-model="email"
        type="text"
        class="border rounded p-2"
        placeholder="Email"
      />
      <input
        v-model="password"
        type="text"
        class="border rounded p-2"
        placeholder="Password"
      />
      <input
        v-model="name"
        type="text"
        class="border rounded p-2"
        placeholder="Name"
      />
      <button @click="addItem" class="bg-blue-500 text-white p-2 rounded">
        Add
      </button>
    </div>
    <div class="mb-4">
      <input
        v-model="searchName"
        type="text"
        class="border rounded p-2"
        placeholder="Search"
      />
    </div>
    <table class="table-auto w-full">
      <thead>
        <tr>
          <th class="px-4 py-2">Email</th>
          <th class="px-4 py-2">Name</th>
          <th class="px-4 py-2">Actions</th>
        </tr>
      </thead>
      <tbody>
        <template v-if="items.length === 0">
          <tr>
            <td class="border px-4 py-2" colspan="3">No data found</td>
          </tr>
        </template>
        <template v-else>
          <tr v-for="(item, index) in items" :key="index">
            <td class="border px-4 py-2">
              <input
                v-if="editIndex === index"
                v-model="item.email"
                type="text"
                class="border rounded p-2 w-full"
              />
              <span v-else>{{ item.email }}</span>
            </td>
            <td class="border px-4 py-2">
              <input
                v-if="editIndex === index"
                v-model="item.name"
                type="text"
                class="border rounded p-2 w-full"
              />
              <span v-else>{{ item.name }}</span>
            </td>
            <td class="border px-4 py-2">
              <button
                @click="edit(index)"
                class="bg-yellow-500 text-white p-2 rounded mr-2"
              >
                {{ editIndex ? "Cancel" : "Edit" }}
              </button>
              <button
                @click="update(item)"
                v-if="editIndex === index"
                class="bg-green-500 text-white p-2 rounded mr-2"
              >
                Update
              </button>
              <button
                @click="remove(item)"
                class="bg-red-500 text-white p-2 rounded"
              >
                Delete
              </button>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
    <div class="flex justify-center items-center mt-4">
      <button
        class="bg-blue-500 text-white p-2 rounded mr-2"
        :disabled="currentPage === 1"
        @click="previousPage"
      >
        Previous
      </button>
      <span>{{ currentPage }} / {{ totalPages }}</span>
      <button
        class="bg-blue-500 text-white p-2 rounded"
        :disabled="currentPage === totalPages"
        @click="nextPage"
      >
        Next
      </button>
    </div>
  </div>
</template>

<script>
import ToastErrors from "./ToastErrors.vue";
import axios from "axios";
export default {
  components: {
    ToastErrors,
  },
  props: {
    title: [],
  },
  data() {
    return {
      currentPage: 1,
      totalPages: 1,
      items: [],
      email: "",
      name: "",
      password: "",
      searchName: "",
      editIndex: null,
      editItem: "",
      hasErrors: false,
      errors: [],
      timer: null,
    };
  },
  watch: {
    searchName: function () {
      clearTimeout(this.timer);
      this.timer = setTimeout(() => {
        this.getList();
      }, 700);
    },
  },
  methods: {
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
        this.getList();
      }
    },
    previousPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
        this.getList();
      }
    },
    async getList() {
      try {
        const response = await axios.get(
          `http://127.0.0.1:8000/api/users?page=${this.currentPage}&name=${this.searchName}`
        );
        const responseData = response.data.data;
        this.items = responseData.data;
        this.totalPages = responseData.last_page;
      } catch (error) {
        this.handleErrors(error);
      }
    },
    async addItem() {
      try {
        const payload = {
          email: this.email,
          name: this.name,
          password: this.password,
        };
        const response = await axios.post(
          "http://127.0.0.1:8000/api/user",
          payload
        );
        console.log("response add ", response);
        if ([200, 201].includes(response.data.code)) {
          this.items.unshift(payload);

          this.resetState();
          this.getList();
        }
      } catch (error) {
        this.handleErrors(error);
      }
    },
    edit(index) {
      if (this.editIndex) {
        this.editIndex = null;
        return;
      }
      this.editIndex = index;
      this.editItem = this.items[index];
    },
    async update(item) {
      try {
        const payload = {
          email: item.email,
          name: item.name,
        };
        const response = await axios.put(
          `http://127.0.0.1:8000/api/user/${item?.id}`,
          payload
        );
        console.log("response edit ", response);
        if ([200, 201].includes(response.data.code)) {
          item = payload;
          this.resetState();
          this.editIndex = null;
          this.getList();
        }
      } catch (error) {
        this.handleErrors(error);
      }
    },
    async remove(item) {
      try {
        const response = await axios.delete(
          `http://127.0.0.1:8000/api/user/${item?.id}`
        );
        if ([200, 201].includes(response.data.code)) {
          this.getList();
        }
      } catch (error) {
        this.handleErrors(error);
      }
    },
    resetState() {
      this.email = "";
      this.name = "";
      this.password = "";
    },
    handleErrors(error) {
      this.hasErrors = true;
      const status = error?.response?.status;
      if (status === 500) {
        this.errors.push("Internal server error");
      }
      if (status === 404) {
        this.errors.push(error.response.data.message);
      }
      if ([400, 422].includes(status)) {
        if (error?.response?.data?.errors) {
          this.errors = [];
          for (const key in error.response.data.errors) {
            if (Object.hasOwnProperty.call(error.response.data.errors, key)) {
              const element = error.response.data.errors[key];
              this.errors.push(element[0]);
            }
          }
        }
      }

      setTimeout(() => {
        this.hasErrors = false;
      }, 5000);
    },
  },
  mounted() {
    this.getList();
  },
};
</script>

<style scoped>
.table-auto {
  border-collapse: collapse;
  width: 100%;
}
.table-auto th,
.table-auto td {
  border: 1px solid #ddd;
  padding: 8px;
}
</style>
