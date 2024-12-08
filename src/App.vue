<template>
  <div id="app">
    <div class="container">
      <h2 class="text-center mb-4">代辦清單</h2>

      <div class="search_area d-flex mb-4 gap-2 align-items-center">
        <input type="text" v-model.trim="searchText" @input="searchTodo" class="rounded-2 border border-success" />
        <button type="button" class="btn-close" aria-label="Close" @click="resetInputs('searchText')">
          X
        </button>
        <button type="button" class="btn btn-secondary" @click="searchTodo">搜尋</button>
      </div>

      <div class="add_area d-flex mb-4 gap-2 align-items-center">
        <input type="date" v-model.trim="addDate" :min="minDate" ref="addDateInput"
          class="text-center rounded-2 border border-success" />
        <input type="search" v-model.trim="addText" @keyup.enter="addTodo" class="rounded-2 border border-success" />
        <button type="button" class="btn-close" aria-label="Close" @click="resetInputs('addText')">X</button>
        <button type="button" class="btn btn-secondary" @click="addTodo">新增</button>
      </div>

      <div class="list_tab">
        <button type="button" class="btn btn-outline-success" :class="{ active: currentTab === 'all' }"
          @click="changeTab('all')">全部</button>
        <button type="button" class="btn btn-outline-success" :class="{ active: currentTab === 'unfinished' }"
          @click="changeTab('unfinished')">待完成</button>
        <button type="button" class="btn btn-outline-success" :class="{ active: currentTab === 'finished' }"
          @click="changeTab('finished')">已完成</button>
      </div>

      <p>清單數量：{{ listCount }} 筆</p>

      <div class="list_date" v-show="data.length > 1">
        <button type="button" class="btn btn-outline-success" :class="{ active: currentList === 'ascending' }"
          @click="listByDate(true)">日期升序排列</button>
        <button type="button" class="btn btn-outline-success" :class="{ active: currentList === 'descending' }"
          @click="listByDate(false)">日期降序排列</button>
        <button type="button" class="btn btn-outline-success" :class="{ active: currentList === 'cancel' }"
          @click="cancelList">取消排列</button>
      </div>

      <div v-for="(item, index) in data" :key="item.id">
        <div v-if="!item.editing" class="d-flex gap-2 mb-3 align-items-center ms-5">
          <input type="checkbox" class="unfinished_check" v-show="currentTab === 'unfinished'"
            @click="checkToFinished(index, item)" />
          {{ item.date }} : {{ item.thing }}
          <button type="button" class="btn-close" aria-label="Close" @click="removeItem(index, item)">X</button>
          <button type="button" class="btn btn-secondary" @click="editItem(item)">編輯</button>
        </div>
        <div v-else>
          <input type="text" v-model="item.date" />
          <input type="text" v-model="item.thing" />
          <button @click="saveItem(item)">儲存</button>
          <button @click="cancelEdit(item)">取消</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, computed, onMounted } from "vue";

interface TodoItem {
  id: number;
  date: string;
  thing: string;
  state: boolean;
  editing: boolean;
  originalCopy?: TodoItem | null;
}

export default defineComponent({
  name: "TodoList",
  setup() {
    const todo = reactive<TodoItem[]>([
      { id: 1, date: "2023-08-10", thing: "買衣服", state: true, editing: false },
      { id: 2, date: "2023-08-12", thing: "看電影", state: true, editing: false },
      { id: 3, date: "2023-08-18", thing: "唱歌", state: true, editing: false },
      { id: 4, date: "2023-08-28", thing: "玩桌遊", state: true, editing: false },
      { id: 5, date: "2023-07-01", thing: "和朋友吃飯", state: false, editing: false },
      { id: 6, date: "2023-06-02", thing: "學習英文", state: false, editing: false },
    ]);

    const data = ref<TodoItem[]>([]);
    const originalData = ref<TodoItem[]>([]);
    const addText = ref<string>("");
    const addDate = ref<string>("");
    const searchText = ref<string>("");
    const currentTab = ref<string>("all");
    const currentList = ref<string>("");
    const minDate = ref<string>(new Date().toISOString().split("T")[0]);

    const listCount = computed(() => data.value.length);

    const changeTab = (tabName: string) => {
      currentTab.value = tabName;
      if (tabName === "finished") {
        data.value = todo.filter((message) => message.state === false);
      } else if (tabName === "unfinished") {
        data.value = todo.filter((message) => message.state === true);
      } else {
        data.value = todo;
      }
    };

    const addTodo = () => {
      if (!addText.value || !addDate.value) {
        alert("請填寫日期與內容");
      } else {
        const addItem: TodoItem = {
          id: todo.length + 1,
          date: addDate.value,
          thing: addText.value,
          state: true,
          editing: false,
        };
        data.value.push(addItem);
        originalData.value.push(addItem);
        addText.value = "";
        addDate.value = "";
      }
    };

    const listByDate = (ascending: boolean) => {
      data.value.sort((a, b) => (new Date(`${ascending ? a.date : b.date}`).getTime() - new Date(`${ascending ? b.date : a.date}`).getTime()));
      currentList.value = ascending ? "ascending" : "descending";
    };

    const cancelList = () => {
      data.value = [...originalData.value];
      currentList.value = "cancel";
    };

    const resetInputs = (inputField: string) => {
      if (inputField === "searchText") {
        searchText.value = "";
        data.value = [...originalData.value];
      } else if (inputField === "addText") {
        addText.value = "";
        addDate.value = "";
      }
    };

    const searchTodo = () => {
      if (searchText.value) {
        data.value = originalData.value.filter((item) => item.thing.includes(searchText.value));
      } else {
        data.value = [...originalData.value];
      }
    };

    const checkToFinished = (itemIndex: number, item: TodoItem) => {
      item.state = false;
      data.value.splice(itemIndex, 1);
    };

    const removeItem = (itemIndex: number, item: TodoItem) => {
      data.value.splice(itemIndex, 1);
      const index = todo.findIndex((todoItem) => todoItem.id === item.id);
      if (index > -1) todo.splice(index, 1);
      const originalIndex = originalData.value.findIndex((originalItem) => originalItem.id === item.id);
      if (originalIndex > -1) originalData.value.splice(originalIndex, 1);
    };

    const editItem = (item: TodoItem) => {
      item.originalCopy = { ...item };
      item.editing = true;
    };

    const saveItem = (item: TodoItem) => {
      if (item.thing && item.date) {
        item.editing = false;
        item.originalCopy = null;
      } else {
        alert("請輸入日期或內容");
      }
    };

    const cancelEdit = (item: TodoItem) => {
      if (item.editing && item.originalCopy) {
        Object.assign(item, item.originalCopy);
        item.editing = false;
        item.originalCopy = null;
      }
    };
    onMounted(() => {
      data.value = todo;
      originalData.value = [...todo];
    });

    return {
      todo,
      data,
      addText,
      addDate,
      searchText,
      currentTab,
      currentList,
      minDate,
      listCount,
      changeTab,
      addTodo,
      listByDate,
      cancelList,
      resetInputs,
      searchTodo,
      checkToFinished,
      removeItem,
      editItem,
      saveItem,
      cancelEdit,
    };
  },
});
</script>

<style scoped>
body {
  margin: 0 auto;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #f0f0f0;
}

.container {
  width: 100%;
  min-width: 600px;
  height: 100%;
  min-height: 700px;
  padding: 50px;
  background-color: rgb(188, 214, 214);
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.list_date {
  margin: 20px auto;
  display: flex;
  justify-content: center;
  gap: 15px;
}

.list_date .active {
  background-color: rgb(61, 118, 52);
}

.list_tab {
  padding: 10px;
  margin: 0 auto;
  display: flex;
  justify-content: center;
  gap: 15px;
}

.list_tab .active {
  background-color: rgb(61, 118, 52);
}

.unfinished_check {
  transform: scale(1.5);
}
</style>
