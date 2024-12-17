<template>
  <q-page class="q-ma-xl">
    <q-form @submit="!selectedRow.id ? addTodo() : updateTodo()" class="q-ma-md">
      <q-input v-model="form.title" label="Input Field" />
      <q-btn
        type="submit"
        :label="!selectedRow.id ? 'Submit' : 'Update'"
        color="primary"
      />
      <q-btn
		  :loading="deleteBtnLoadingState"
        v-if="selectedRow.id"
	  @click="deleteTodo()"
        class="q-ml-md"
        label="Delete"
        color="negative"
      />

    </q-form>

    <q-table
      title="Todos"
      :rows="rows"
      :columns="columns"
      row-key="name"
      @row-click="onRowClick"
    />

  </q-page>
</template>

<script>
import { ref, onMounted } from "vue"; // Import necessary Vue features
import axios from "axios"; // Import axios

export default {
  setup() {
    const rows = ref([]); // Make sure rows is a reactive reference
    const columns = ref([
      {
        name: "title",
        label: "Title",
        align: "left",
        field: "title",
      },
      {
        name: "completed",
        label: "Completed",
        align: "left",
        field: "completed",
      },
    ]);

    let form = ref({
      userId: 1,
      title: null,
      completed: false,
    });
    
    let btnLoadingState = ref(false);
    const addTodo = () => {
      btnLoadingState.value = true;
      axios
        .post("https://jsonplaceholder.typicode.com/todos", form.value)
        .then((response) => {
          if (response.status === 201) {
            rows.value.unshift(response.data);
            form.value.title = null;
          }
	    btnLoadingState.value = false;
        });
    };

    // Fetch Todos data
    const getTodos = async () => {
      try {
        const response = await axios.get("https://jsonplaceholder.typicode.com/todos");
        rows.value = response.data;
      } catch (error) {
        console.error("Error fetching todos:", error);
      }
    };

    // Call getTodos when the component is mounted
    onMounted(() => {
      getTodos();
    });

    let selectedRow = ref({});
    const onRowClick = (evt, row) => {
      selectedRow.value = row;
      form.value.title = row.title;
    };


    const updateTodo = () => {
      btnLoadingState.value = true;
      axios
        .put(
          `https://jsonplaceholder.typicode.com/todos/${selectedRow.value.id}`,
          {
            title: form.value.title,
          }
        )
        .then((response) => {
          if (response.status === 200) {
            let index = rows.value.findIndex(
              (row) => row.id === selectedRow.value.id
            );
            rows.value[index].title = response.data.title;
            form.value.title = null;
            selectedRow.value = {};
          }
          btnLoadingState.value = false;
        });
    };

    let deleteBtnLoadingState = ref(false);
    const deleteTodo = () => {
      deleteBtnLoadingState.value = true;
      axios
        .delete(
          `https://jsonplaceholder.typicode.com/todos/${selectedRow.value.id}`
        )
        .then((response) => {
          if (response.status === 200) {
            rows.value = rows.value.filter(
              (row) => row.id !== selectedRow.value.id
            );
            form.value.title = null;
          }
          deleteBtnLoadingState.value = false;
        });
    };



    return {
      rows,
      columns,
      form,
      btnLoadingState,
      addTodo,
      selectedRow,
      onRowClick,
      updateTodo,
      deleteBtnLoadingState,
      deleteTodo,
    };

  },
   
};
</script>
