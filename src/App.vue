<template>
  <div class="app-container">
    <h1>Quản lý mượn trả sách</h1>
    <button @click="openAddForm" class="add-button">Thêm thông tin</button>

    <div class="filter-container">
      <label for="filterStatus">Lọc theo trạng thái:</label>
      <select id="filterStatus" v-model="filterStatus">
        <option value="">Tất cả</option>
        <option value="borrowed">Chưa trả</option>
        <option value="returned">Đã trả</option>
      </select>
    </div>

    <div v-if="isFormVisible" class="form-container">
      <form @submit.prevent="saveBook">
        <div class="form-group">
          <label for="name">Tên sách:</label>
          <input id="name" v-model="book.name" required />
        </div>
        <div class="form-group">
          <label for="borrower">Tên người mượn:</label>
          <input id="borrower" v-model="book.borrower" required />
        </div>
        <div class="form-group">
          <label for="borrowDate">Ngày mượn:</label>
          <input id="borrowDate" type="date" v-model="book.borrowDate" @change="updateReturnDate" required />
        </div>
        <div class="form-group">
          <label for="returnDate">Ngày trả:</label>
          <input id="returnDate" type="date" v-model="book.returnDate" :min="book.borrowDate" required />
        </div>
        <div class="form-actions">
          <button type="submit" class="save-btn">Lưu</button>
          <button type="button" @click="closeForm" class="cancel-btn">Hủy</button>
        </div>
      </form>
    </div>

    <table class="book-table">
      <thead>
        <tr>
          <th>STT</th>
          <th>Tên sách</th>
          <th>Người mượn</th>
          <th>Ngày mượn</th>
          <th>Ngày trả</th>
          <th>Trạng thái</th>
          <th>Thao tác</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(book, index) in filteredBooks" :key="book.id">
          <td>{{ index + 1 }}</td>
          <td>{{ book.name }}</td>
          <td>{{ book.borrower }}</td>
          <td>{{ book.borrowDate }}</td>
          <td>{{ book.returnDate }}</td>
          <td>
            <select v-model="book.status" @change="handleStatusChange(book.id, book.status)">
              <option value="borrowed">Chưa trả</option>
              <option value="returned">Đã trả</option>
            </select>
          </td>
          <td>
            <button @click="openEditForm(book)" class="edit-btn">Sửa</button>
            <button @click="openDeleteModal(book)" class="delete-btn">Xóa</button>
          </td>
        </tr>
      </tbody>
    </table>

    <div v-if="isDeleteModalVisible" class="modal-overlay" @click="cancelDelete">
      <div class="modal-content" @click.stop>
        <p>Bạn có chắc chắn muốn xóa thông tin này?</p>
        <div class="modal-actions">
          <button @click="confirmDelete" class="confirm-btn">Xóa</button>
          <button @click="cancelDelete" class="cancel-btn">Hủy</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const books = ref(JSON.parse(localStorage.getItem('books')) || []);
const isFormVisible = ref(false);
const isDeleteModalVisible = ref(false);
const currentEditData = ref(null);
const bookToDelete = ref(null);
const book = ref({ name: '', borrower: '', borrowDate: '', returnDate: '', status: 'borrowed' });
const filterStatus = ref('');

const filteredBooks = computed(() => {
  if (!filterStatus.value) {
    return books.value;
  }
  return books.value.filter(book => book.status === filterStatus.value);
});

const openAddForm = () => {
  currentEditData.value = null;
  isFormVisible.value = true;
  resetBookForm();
};

const openEditForm = (editBook) => {
  currentEditData.value = editBook;
  isFormVisible.value = true;
  book.value = { ...editBook };
};

const saveBook = () => {
  if (currentEditData.value) {
    const index = books.value.findIndex(item => item.id === currentEditData.value.id);
    books.value.splice(index, 1, { ...book.value });
  } else {
    book.value.id = Date.now();
    books.value.push({ ...book.value });
  }
  localStorage.setItem('books', JSON.stringify(books.value));
  closeForm();
};

const handleStatusChange = (bookId, status) => {
  const bookItem = books.value.find(item => item.id === bookId);
  if (bookItem) {
    bookItem.status = status;
    localStorage.setItem('books', JSON.stringify(books.value));
  }
};

const openDeleteModal = (book) => {
  bookToDelete.value = book;
  isDeleteModalVisible.value = true;
};

const confirmDelete = () => {
  books.value = books.value.filter(book => book.id !== bookToDelete.value.id);
  localStorage.setItem('books', JSON.stringify(books.value));
  closeDeleteModal();
};

const cancelDelete = () => {
  closeDeleteModal();
};

const closeDeleteModal = () => {
  isDeleteModalVisible.value = false;
};

const closeForm = () => {
  isFormVisible.value = false;
};

const resetBookForm = () => {
  book.value = { name: '', borrower: '', borrowDate: '', returnDate: '', status: 'borrowed' };
};

const updateReturnDate = () => {
  if (book.value.returnDate && book.value.borrowDate) {
    const borrowDate = new Date(book.value.borrowDate);
    const returnDate = new Date(book.value.returnDate);
    if (returnDate < borrowDate) {
      book.value.returnDate = ''; 
    }
  }
};
</script>

<style scoped>
.app-container {
  padding: 20px;
  background-color: #f9f9f9; 
  border-radius: 8px; 
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  max-width: 800px; 
  margin: auto; 
}

h1 {
  text-align: center; 
  color: #333; 
  margin-bottom: 20px; 
}

.add-button {
  display: block;
  width: 100%; 
  padding: 10px;
  background-color: #4CAF50; 
  color: white;
  border: none; 
  border-radius: 5px; 
  font-size: 16px;
  cursor: pointer; 
  transition: background-color 0.3s; 
}

.add-button:hover {
  background-color: #45a049; 
}

.filter-container {
  margin: 20px 0;
}

.filter-container label {
  font-weight: bold;
  margin-right: 10px;
}

.form-container {
  margin-top: 20px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  background-color: #fff;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input, .form-group select {
  width: 100%;
  padding: 8px;
  box-sizing: border-box;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.form-actions {
  display: flex;
  justify-content: space-between;
}

.save-btn, .cancel-btn {
  padding: 10px 20px; 
  border-radius: 4px; 
  cursor: pointer; 
}

.save-btn {
  background-color: #4CAF50;
  color: white;
}

.cancel-btn {
  background-color: #f44336;
  color: white;
}

.book-table {
  margin-top: 20px;
  width: 100%;
  border-collapse: collapse;
}

.book-table th, .book-table td {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: left;
}

.book-table th {
  background-color: #f2f2f2;
}

.book-table tr:hover {
  background-color: #f1f1f1;
}

.edit-btn, .delete-btn {
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.edit-btn {
  background-color: #4CAF50;
  color: white;
}

.delete-btn {
  background-color: #f44336;
  color: white;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5); 
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: white; 
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2); 
  text-align: center; 
}

.modal-actions {
  display: flex;
  justify-content: space-around; 
  margin-top: 20px; 
}

.confirm-btn, .cancel-btn {
  padding: 10px 15px; 
  border-radius: 4px; 
  cursor: pointer; 
}

.confirm-btn {
  background-color: #f44336;
  color: white;
}

.cancel-btn {
  background-color: #4CAF50;
  color: white;
}
</style>
