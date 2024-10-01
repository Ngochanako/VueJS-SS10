<script setup>
import {computed, reactive, ref} from 'vue'
//Initialize
let users=reactive(JSON.parse(localStorage.getItem('listUser'))||[]);
const showModalEdit=ref(false);
const showModalDelete=ref(false);
const showModalBlock=ref(false);
let typeSubmit='add';
const user=reactive({
  id:'',
  name:'',
  dateOfBirth:'',
  email:'',
  address:'',
  status:true,
})
//function resetUser
const resetUser=()=>{
  user.id='';
  user.name='';
  user.dateOfBirth='';
  user.email='';
  user.address='';
  user.status=true;
}
//click button Add New User
const handleFormEdit=()=>{
  showModalEdit.value=true;
}
//click button Edit User
const editUser=(id)=>{
    showModalEdit.value=true;
    typeSubmit='edit';
    const userEdit=users.find(u=>u.id===id);
    if(userEdit){
        user.id = userEdit.id;
        user.name = userEdit.name;
        user.dateOfBirth = userEdit.dateOfBirth;
        user.email = userEdit.email;
        user.address = userEdit.address;
        user.status = userEdit.status;
    }   
}
//close modal edit
const closeModalEdit=()=>{
  showModalEdit.value=false;
}
// add new User
const addUser=()=>{
  if (typeSubmit=='add'){
    user.id=new Date().getTime();
    users.push({...user});
    localStorage.setItem('listUser',JSON.stringify(users));
    resetUser();
    showModalEdit.value=false;
  }else{
    users=users.map(u=>u.id===user.id? {...user}:u);
    localStorage.setItem('listUser',JSON.stringify(users));
    resetUser();
    showModalEdit.value=false;
    typeSubmit='add';
  }
}
//delete User
const deleteUser=(id)=>{
  showModalDelete.value=true;
  user.id=id;
}
const cancelDelete=()=>{
    showModalDelete.value=false;
    user.id='';
}
const confirmDelete=()=>{
    users=users.filter(u=>u.id!==user.id);
    localStorage.setItem('listUser',JSON.stringify(users));
    showModalDelete.value=false;
    user.id='';
}
//modal block
const blockUser=(id)=>{
    user.id=id;
    showModalBlock.value=true;
}
const cancelBlock=()=>{
   user.id='';
   showModalBlock.value=false;
}
const confirmBlock=()=>{
   users=users.map(u=>u.id===user.id? {...u,status:!u.status}:u);
   localStorage.setItem('listUser',JSON.stringify(users));
   showModalBlock.value=false;
   user.id='';
}
//search email
const valueSearch=ref('');
const searchEmail=()=>{
  users=JSON.parse(localStorage.getItem('listUser'))||[];
  if(valueSearch.trim()!==''){
    users=users.filter(u=>u.email.toLowerCase().includes(valueSearch.toLowerCase()));
  }else{
    users=JSON.parse(localStorage.getItem('listUser'))||[];
  }
}
//pagination
const selectOption=ref('2');
const currentPage=ref(1);
let totalPages=computed(()=>{
  return Math.ceil(users.length/+selectOption.value);
})
const handlePage=(num)=>{
  currentPage.value=num;
}
</script>

<template>
     <body>
      <div class="w-[80%] m-auto mt-4 h-[100vh]">
      <main class="main">
        <header class="d-flex justify-content-between mb-3">
          <h3>Nhân viên</h3>
          <button @click="handleFormEdit" class="btn btn-primary">Thêm mới nhân viên</button>
        </header>
        <div class="d-flex align-items-center justify-content-end gap-2 mb-3">
          <input
            style="width: 350px"
            type="text"
            class="form-control"
            placeholder="Tìm kiếm theo email"
            v-model="valueSearch"
          />
          <i @click="searchEmail" class="fa-solid fa-arrows-rotate" title="Refresh"></i>
        </div>
        <table class="table table-bordered table-hover table-striped">
          <thead>
            <tr>
              <th>STT</th>
              <th>Họ và tên</th>
              <th>Ngày sinh</th>
              <th>Email</th>
              <th>Địa chỉ</th>
              <th>Trạng thái</th>
              <th colspan="2">Chức năng</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(btn,index) in users.slice((currentPage-1)*(+selectOption),currentPage*(+selectOption))" :key="btn.id">

              <td>{{ index+1 }}</td>
              <td>{{ btn.name }}</td>
              <td>{{ btn.dateOfBirth }}</td>
              <td>{{ btn.email }}</td>
              <td>{{ btn.address }}</td>
              <td>
                <div style="display: flex; align-items: center; gap: 8px">
                  <div class="status status-active"></div>
                  <span>{{ btn.status?'Đang hoạt động':'Không hoạt động' }} </span>
                </div>
              </td>
              <td>
                <span @click="blockUser(btn.id)" class="button button-block">{{btn.status?'Chặn':'Bỏ chặn' }}</span>
              </td>
              <td>
                <span @click="editUser(btn.id)" class="button button-edit">Sửa</span>
              </td>
              <td><span @click="deleteUser(btn.id)" class="button button-delete">Xóa</span></td>
            </tr>
          </tbody>
        </table>
        <footer class="d-flex justify-content-end align-items-center gap-3">
          <select class="form-select" v-model="selectOption">
            <option value="2" selected>Hiển thị 2 bản ghi trên trang</option>
            <option value="5">Hiển thị 5 bản ghi trên trang</option>
            <option value="10">Hiển thị 10 bản ghi trên trang</option>
            <option value="20">Hiển thị 20 bản ghi trên trang</option>
          </select>
          <ul class="pagination">
            <li class="page-item">
              <a class="page-link" href="#">Previous</a>
            </li>
            <li @click="handlePage(1)" class="page-item"><a class="page-link" href="#">1</a></li>
            <li @click="handlePage(2)" class="page-item"><a class="page-link" href="#">2</a></li>
            <li @click="handlePage(3)" class="page-item"><a class="page-link" href="#">3</a></li>
            <li class="page-item"><a class="page-link" href="#">Next</a></li>
          </ul>
        </footer>
      </main>
    </div>
     <div class="overlay" v-show="showModalEdit">
      <form class="form" @submit.prevent="addUser">
        <div class="d-flex justify-content-between align-items-center">
          <h4>Chỉnh sửa nhân viên</h4>
          <i @click="closeModalEdit" class="fa-solid fa-xmark"></i>
        </div>
        <div>
          <label class="form-label" for="userName">Họ và tên</label>
          <input id="userName" type="text" class="form-control" required v-model="user.name"/>
          <!-- <div class="form-text error">Họ và tên không được để trống.</div> -->
        </div>
        <div>
          <label class="form-label" for="dateOfBirth">Ngày sinh</label>
          <input id="dateOfBirth" type="date" class="form-control" required v-model="user.dateOfBirth" />
        </div>
        <!-- <div class="form-text error">
          Ngày sinh không được lớn hơn ngày hiện tại.
        </div> -->
        <div>
          <label class="form-label" for="email">Email</label>
          <input id="email" type="text" class="form-control" required v-model="user.email" />
        </div>
        <!-- <div class="form-text error">Email không được để trống.</div> -->
        <div>
          <label class="form-label" for="address">Địa chỉ</label>
          <textarea class="form-control" id="address" rows="3" required v-model="user.address"></textarea>
        </div>
        <div>
          <button type="submit" class="w-100 btn btn-primary">{{typeSubmit=='Add'?'Thêm mới':'Chỉnh sửa'}}</button>
        </div>
      </form>
    </div>

    <!-- Modal xác nhận chặn tài khoản -->
    <div class="overlay" v-if="showModalBlock">
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i class="fa-solid fa-xmark"></i>
        </div>
        <div class="modal-body-custom">
          <span>Bạn có chắc chắn muốn chặn tài khoản này?</span>
        </div>
        <div class="modal-footer-custom">
          <button @click="cancelBlock" class="btn btn-light">Hủy</button>
          <button @click="confirmBlock" class="btn btn-danger">Xác nhận</button>
        </div>
      </div>
    </div>

    <!-- Modal xác nhận xóa tài khoản -->
    <div class="overlay" v-if="showModalDelete">
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i class="fa-solid fa-xmark"></i>
        </div>
        <div class="modal-body-custom">
          <span>Bạn có chắc chắn muốn xóa tài khoản này?</span>
        </div>
        <div class="modal-footer-custom">
          <button @click="cancelDelete" class="btn btn-light">Hủy</button>
          <button @click="confirmDelete" class="btn btn-danger">Xác nhận</button>
        </div>
      </div>
    </div>
  </body>
</template>

<style scoped>
 
</style>
