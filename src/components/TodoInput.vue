<script setup>
import { ref } from 'vue';
import { statuses } from '../const/statuses'

const input = ref("");
const inputDate = ref("");

const isErrMsg = ref(false);
let errMsg = ref("");

// タスク登録処理
function onSubmitForm() {

  // 入力チェック
  if (input.value == "" || inputDate.value == "") {
    isErrMsg.value = true;
    errMsg.value = "タスク・期限を両方入力してください。";
    event.preventDefault();
    return;
  }

  // ローカルストレージから保存されたタスクの配列を取得
  const items = JSON.parse(localStorage.getItem("items")) || [];

  // 新しいタスクを生成
  const newItem = {
    id: items.length + 1, // id
    content: input.value, // タスクの内容
    limit: inputDate.value, // 期限
    state: statuses.NOT_START, // 状態（未着手など）
    onEdit: false, // 編集中フラグ
  }

  // TodoListに追加
  items.push(newItem);

  // ローカルストレージに最新のTodoListを反映
  localStorage.setItem("items", JSON.stringify(items));

}

</script>

<template>
  <div class="todo-app">
    <p v-if="isErrMsg" class="error-msg">{{ errMsg }}</p>
    <form @submit="onSubmitForm" class="todo-form">
      <label class="form-label">
        やること
        <input type="text" v-model="input" class="form-input">
      </label>
      <label class="form-label">
        期限
        <input type="date" v-model="inputDate" class="form-input">
      </label>
      <input type="submit" value="登録!" class="submit-button">
    </form>
  </div>
</template>

<style>
.todo-app {
  font-family: Arial, sans-serif;
  background-color: #f5f7fa;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  max-width: 600px;
  margin: auto;
}

.error-msg {
  color: #d9534f;
  background-color: #f9d6d5;
  border: 1px solid #d9534f;
  padding: 10px;
  border-radius: 4px;
  margin-bottom: 15px;
  text-align: center;
}

.todo-form {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.form-label {
  font-size: 14px;
  color: #333;
  margin-bottom: 5px;
}

.form-input {
  width: 100%;
  padding: 8px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #fff;
  font-size: 14px;
}

.form-input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
}

.submit-button {
  padding: 10px 20px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
  align-self: flex-start;
}

.submit-button:hover {
  background-color: #0056b3;
}
</style>