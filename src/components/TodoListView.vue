<script setup>

import { ref } from 'vue';
import { statuses } from '../const/statuses';

// ローカルストレージからTodoリストを取得
const items = ref(JSON.parse(localStorage.getItem("items")) || []);

// 表示されている値を保持
const inputContent = ref();
const inputLimit = ref();
const inputState = ref();

// エラーメッセージ
let isErrMsg = ref(false);
let errMsg = ref('');

// モーダル表示フラグ
let isShowModal = ref(false);

// 削除関連
let deleteItemId;
let deleteItemContent = ref();

// 予定より遅れているか判定する日付の基準
const today = new Date().setHours(0, 0, 0, 0); // 日付比較なので時刻は無視

// タスクの編集処理
function onEdit(id) {

  // 編集中のタスクがないか調べる
  let isOnEditOther = false;
  items.value.map((item) => {
    if (item.onEdit) {
      isOnEditOther = true;
      return;
    }
  });

  // 他のタスクが編集中の場合は処理を中断
  if (isOnEditOther) {
    isErrMsg.value = true;
    errMsg = "他のタスクが編集中です。";
    return;
  }

  // 編集時に入力済みの値を表示させる
  inputContent.value = items.value[id].content;
  inputLimit.value = items.value[id].limit;
  inputState.value = items.value[id].state;

  // 編集中フラグON
  items.value[id].onEdit = true;

}

// タスクの更新反映
function onUpdate(id) {

  // 入力チェック
  if (inputContent.value == "" || inputLimit.value == "") {
    isErrMsg.value = true;
    errMsg = "タスクと日付の両方に入力が必要です。"
    return;
  }

  // 登録し直すリストの内容で生成
  const newItem = {
    id: id,
    content: inputContent.value,
    limit: inputLimit.value,
    state: inputState.value,
    onEdit: false,
  }

  items.value.splice(id, 1, newItem); // リストを入れ替え
  localStorage.setItem("items", JSON.stringify(items.value)); // ローカルストレージに登録
  isErrMsg.value = false; // エラーメッセージ表示フラグを初期化

}

// タスク削除
function onDeleteItem() {

  // 選択されたタスクを削除
  items.value.splice(deleteItemId, 1);

  // idを振りなおす
  items.value = items.value.map((item, index) => ({
    id: index,
    content: item.content,
    limit: item.limit,
    state: item.state,
    onEdit: item.onEdit,
  }));

  // ローカルストレージに登録
  localStorage.setItem("items", JSON.stringify(items.value));

  // モーダル非表示
  onHideModal();

}

// モーダル非表示
function onHideModal() {

  // モーダル表示フラグOFF
  isShowModal.value = false;

}


// 削除用モーダルを表示する
function showDeleteModal(id) {

  // 削除時に必要な情報を保持
  deleteItemId = id;
  isShowModal.value = true;
  deleteItemContent = items.value[id].content;

}

// 日付順に並び変える
function sortByLimit() {

  items.value.sort((a, b) => new Date(a.limit) - new Date(b.limit));
  localStorage.setItem("items", JSON.stringify(items.value));

}

// id順に並び変える
function sortById() {

  items.value.sort((a, b) => a.id - b.id);
  localStorage.setItem("items", JSON.stringify(items.value));

}

</script>

<template>
  <div class="todo-list-view">
    <p v-if="isErrMsg" class="error-msg">{{ errMsg }}</p>

    <div v-if="isShowModal" class="modal">
      <div class="modal-content">
        <p class="modal-text">{{ deleteItemContent }}を削除してもよろしいですか？</p>
        <div class="modal-buttons">
          <button @click="onDeleteItem()" class="modal-button confirm">はい</button>
          <button @click="onHideModal()" class="modal-button cancel">キャンセル</button>
        </div>
      </div>
    </div>

    <table class="todo-table">
      <thead>
        <tr>
          <th>ID <button @click="sortById()" class="sort-button">↓</button></th>
          <th>やること</th>
          <th>期限 <button @click="sortByLimit()" class="sort-button">↓</button></th>
          <th>状態</th>
          <th>編集</th>
          <th>削除</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in items" :key="item.id" :class="{ overdue: new Date(item.limit) <= today, completed: item.state.id == statuses.COMPLETED.id }">
          <td>{{ item.id }}</td>
          <td>
            <span v-if="!item.onEdit">{{ item.content }}</span>
            <input v-else v-model="inputContent" type="text" class="form-input">
          </td>
          <td>
            <p v-if="!item.onEdit">{{ item.limit }}</p>
            <input v-else v-model="inputLimit" type="date" class="form-input">
          </td>
          <td>
            <span v-if="!item.onEdit">{{ item.state.value }}</span>
            <select v-else v-model="inputState" class="form-select">
              <option v-for="state in statuses" :key="state.id" :value="state">{{ state.value }}</option>
            </select>
          </td>
          <td>
            <button v-if="!item.onEdit" @click="onEdit(item.id)" class="action-button edit">編集</button>
            <button v-else @click="onUpdate(item.id)" class="action-button save">完了</button>
          </td>
          <td>
            <button @click="showDeleteModal(item.id)" class="action-button delete">削除</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style>
.todo-list-view {
  font-family: 'Roboto', sans-serif;
  background: linear-gradient(135deg, #f0f4f8, #e2e8f0);
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1);
  max-width: 900px;
  margin: 20px auto;
}

.error-msg {
  color: #d9534f;
  background-color: #ffe9e9;
  border: 1px solid #d9534f;
  padding: 10px;
  border-radius: 6px;
  margin-bottom: 20px;
  text-align: center;
  font-weight: bold;
}

.todo-table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px 0;
}

.todo-table th,
.todo-table td {
  border: 1px solid #ddd;
  padding: 12px 15px;
  text-align: left;
}

.todo-table th {
  background-color: #1e3a8a;
  color: #fff;
  text-align: center;
  font-size: 14px;
}

.todo-table tr:nth-child(even) {
  background-color: #f9fafb;
}

.todo-table tr.overdue {
  background-color: #ffe6e6;
  color: #b91c1c;
}

.todo-table tr.completed {
  background-color: rgba(0, 0, 0, 0.2);
}

.sort-button {
  background: none;
  border: none;
  color: #fff;
  cursor: pointer;
  margin-left: 8px;
  font-size: 12px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: #fff;
  padding: 30px;
  border-radius: 12px;
  text-align: center;
  width: 300px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

.modal-text {
  font-size: 16px;
  margin-bottom: 20px;
}

.modal-buttons {
  display: flex;
  justify-content: space-between;
}

.modal-button {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
}

.modal-button.confirm {
  background-color: #10b981;
  color: #fff;
}

.modal-button.cancel {
  background-color: #d9534f;
  color: #fff;
}

.modal-button:hover {
  opacity: 0.9;
}

.action-button {
  padding: 8px 12px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
}

.action-button.edit {
  background-color: #1d4ed8;
  color: #fff;
}

.action-button.save {
  background-color: #10b981;
  color: #fff;
}

.action-button.delete {
  background-color: #b91c1c;
  color: #fff;
}

.action-button:hover {
  opacity: 0.85;
}

.form-input,
.form-select {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 14px;
  width: 100%;
}

.form-input:focus,
.form-select:focus {
  border-color: #1d4ed8;
  box-shadow: 0 0 6px rgba(29, 78, 216, 0.5);
}
</style>