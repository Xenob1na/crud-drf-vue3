<template>
    <div class="container">
        <h1 class="text-center mt-5 mb-3">Employees List</h1>
        <div class="card" style="margin-bottom: 80px;">
            <div class="card-header text-center">Form</div>
            <div class="card-body">
                <form @submit.prevent="">
                    <div class="form-group">
                        <label for="fullname" class="mb-3 mt-3">fullname</label>
                        <input v-model="fullname" type="text" class="form-control" placeholder="Enter fullname">
                    </div>
                    <div class="form-group">
                        <label for="dep" class="mb-3 mt-3">Department</label>
                        <input v-model="dep" type="text" class="form-control" placeholder="Enter Department">
                    </div>
                    <div class="form-group">
                        <label for="birthdate" class="mb-3 mt-3">Birthdate</label>
                        <input v-model="birthdate" type="text" class="form-control" placeholder="Enter Birthdate">
                    </div>
                    <div class="form-group">
                        <label for="salary" class="mb-3 mt-3">Salary</label>
                        <input v-model="salary" type="text" class="form-control" placeholder="Enter Salary">
                    </div>
                    <button type="button" class="btn btn-primary mt-3" @click="submitEmployee">
                        {{ isEditing ? 'Update' : 'Submit' }}
                    </button>
                </form>
            </div>
        </div>
        <div class="card">
            <div class="card-header text-center">Table</div>
            <div class="card-body">
                <table class="table table-bordered">
                    <thead>
                        <tr>
                            <th>ID</th>
                            <th>Fullname</th>
                            <th>Dep</th>
                            <th>Birthdate</th>
                            <th>Salary</th>
                            <th width="250px">Action</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="employee in employees" :key="employee.id">
                            <td>{{ employee.id }}</td>
                            <td>{{ employee.fullname }}</td>
                            <td>{{ employee.dep }}</td>
                            <td>{{ employee.birthdate }}</td>
                            <td>{{ employee.salary }}</td>
                            <td>
                                <button class="btn btn-primary" style="margin-right: 10px;" @click="editEmployee(employee)">Edit</button>
                                <button class="btn btn-danger" @click="deleteEmployee(employee)">Delete</button>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import Swal from 'sweetalert2';

const employees = ref([])
const fullname = ref('')
const dep = ref('')
const birthdate = ref('')
const salary = ref('')
const id = ref('')

const isEditing = ref(false)


axios
    .get(`http://127.0.0.1:8000/api/employees`)
    .then((response) => {
        employees.value = response.data
    })
    .catch((error) => {
        console.log(error)
    })


const submitEmployee = () => {
    if (!isEditing.value) {
        axios.post('http://127.0.0.1:8000/api/employees/', {
            fullname: fullname.value,
            dep: dep.value,
            birthdate: birthdate.value,
            salary: salary.value
        })
        .then((response) => {
            Swal.fire({
            icon: 'success',
            title: 'Success!',
            showConfirmButton: false,
            timer: 1500
        })
                employees.value.push(response.data)
                fullname.value = '';
                dep.value = '';
                birthdate.value = '';
                salary.value = '';
            });

    } else {
        axios.put(`http://127.0.0.1:8000/api/employees/${id.value}/`, {
            fullname: fullname.value,
            dep: dep.value,
            birthdate: birthdate.value,
            salary: salary.value
        })
        .then((response) => {
            let objIndex = employees.value.findIndex((s) => s.id == id.value);
            let tmp = employees.value;
            tmp[objIndex] = response.data;
            employees.value = tmp;
            fullname.value = '';
            dep.value = '';
            birthdate.value = '';
            salary.value = '';
            id.value = '';
            isEditing.value = false;
        })
        .catch((error) => {
            console.log(error)
        })
    } 
}



const editEmployee = (employee) => {
    try {
        isEditing.value = true;
        id.value = employee.id;
        fullname.value = employee.fullname;
        dep.value = employee.dep;
        birthdate.value = employee.birthdate;
        salary.value = employee.salary;
    } catch (error) {
        console.log(error)
    }
}

const deleteEmployee = async (employee) => {
    Swal.fire({
        title: 'Are you sure?',
        text: "You won't be able to revert this!",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'Yes, delete it!'
    }).then((result) => {
        if (result.isConfirmed) {
            axios.delete(`http://127.0.0.1:8000/api/employees/${employee.id}`)
                .then((response) => {
                    Swal.fire({
                        icon: 'success',
                        title: 'Project deleted successfully !',
                        showConfirmButton: false,
                        timer: 1500
                    })
                    return console.log(response);
                })
                .catch((error) => {
                    Swal.fire({
                        icon: 'error',
                        title: 'Oops...',
                        text: 'Something went wrong!',
                        showConfirmButton: false,
                        timer: 1500
                    })
                    return console.log(error);
                })
        }
    })
}
</script>
