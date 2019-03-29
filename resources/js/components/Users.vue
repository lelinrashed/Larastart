<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminOrAuthor()">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users Table</h3>
                        <div class="card-tools">
                            <button data-toggle="modal" v-on:click="newModal" class="btn btn-primary">Add User &nbsp; <i class="fas fa-user-plus fa-fw"></i> </button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                            <tbody>
                                <tr>
                                    <th>ID</th>
                                    <th>Name</th>
                                    <th>Email</th>
                                    <th>Type</th>
                                    <th>Created at</th>
                                    <th>Modify</th>
                                </tr>
                                <tr v-for="user, i in users.data">
                                    <td>{{ i+1 }}</td>
                                    <td>{{ user.name }}</td>
                                    <td>{{ user.email }}</td>
                                    <td><span class="tag tag-success">{{ user.type | upText }}</span></td>
                                    <td>{{ user.created_at | myDate }}</td>
                                    <td>
                                        <a href="#" v-on:click="editModal(user)">
                                            <i class="fas fa-edit blue"></i>
                                        </a>
                                        <a href="#" @click="deleteUser(user.id)">
                                            &nbsp;&nbsp;<i class="fas fa-trash red"></i>
                                        </a>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                    <!-- /.card-body -->
                    <div class="card-footer">
                        <pagination :data="users" @pagination-change-page="getResults"></pagination>
                    </div>
                </div>
                <!-- /.card -->
            </div>
        </div>
        <div v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>
        <div class="modal fade" id="addNewUser" tabindex="-1" role="dialog" aria-labelledby="exampleModalCenterTitle" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 v-show="!editmode" class="modal-title" id="exampleModalCenterTitle">Add New User</h5>
                        <h5 v-show="editmode" class="modal-title" id="exampleModalCenterTitle">Update User's info</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editmode ? updateUser() :createUser()">
                        <div class="modal-body">
                            <div class="form-group">
                                <input v-model="form.name" placeholder="Name" type="text" name="name" class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>
                            <div class="form-group">
                                <input v-model="form.email" placeholder="Email address" type="email" name="email" class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                            </div>
                            <div class="form-group">
                                <textarea name="bio" v-model="form.bio" placeholder="Short bio for user (Optional)" class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                                <has-error :form="form" field="bio"></has-error>
                            </div>
                            <div class="form-group">
                                <select name="type" v-model="form.type" class="form-control" :class="{ 'is-invalid' : form.errors.has('type') }">
                                    <option value="">Select User Role</option>
                                    <option value="admin">Admin</option>
                                    <option value="user">Standard User</option>
                                    <option value="author">Author</option>
                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>
                            <div class="form-group">
                                <input v-model="form.password" placeholder="Password" type="password" name="password" class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                                <has-error :form="form" field="password"></has-error>
                            </div>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                            <button v-show="editmode" type="submit" class="btn btn-warning">Update User</button>
                            <button v-show="!editmode" type="submit" class="btn btn-success">Create User</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    data() {
        return {
            editmode: false,
            users: {},
            form: new Form({
                id: '',
                name: '',
                email: '',
                password: '',
                type: '',
                bio: '',
                photo: ''
            })
        }
    },
    methods: {
        createUser() {
            this.$Progress.start();
            this.form.post('api/user').then(() => {
                    this.loadUser();
                    $('#addNewUser').modal('hide');
                    // Fire.$emit('AfterCreate');            
                    toast.fire({
                        type: 'success',
                        title: 'User created successfully'
                    });
                    this.$Progress.finish();
                })
                .catch(() => {
                    this.$Progress.fail();
                    $('#addNewUser').modal('show');
                });
        },
        loadUser() {
            if (this.$gate.isAdminOrAuthor()) {
                axios.get('api/user')
                    .then(({ data }) => (this.users = data))
            }
        },
        deleteUser(id) {
            Swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                type: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Yes, delete it!'
            }).then((result) => {
                if (result.value) {
                    // Send request to the server
                    this.$Progress.start();
                    this.form.delete('api/user/' + id).then(() => {
                        this.loadUser();
                        Swal.fire(
                            'Deleted!',
                            'Your file has been deleted.',
                            'success'
                        )
                        this.$Progress.finish();
                    }).catch(() => {
                        this.$Progress.fail();
                        Swal.fire(
                            'Error!',
                            "Can't delete the user.",
                            'error'
                        )
                    });
                }
            });
        },
        newModal() {
            this.editmode = false;
            this.form.clear();
            this.form.reset();
            $('#addNewUser').modal('show');
        },
        editModal(user) {
            this.editmode = true;
            this.form.clear();
            this.form.reset();
            $('#addNewUser').modal('show');
            this.form.fill(user);
        },
        updateUser() {
            this.$Progress.start();
            this.form.put('api/user/' + this.form.id)
                .then(() => {
                    this.loadUser();
                    $('#addNewUser').modal('hide');
                    toast.fire({
                        type: 'success',
                        title: 'User updated successfully'
                    });
                    this.$Progress.finish();
                })
                .catch(() => {
                    this.$Progress.fail();

                });
        },
        getResults(page = 1) {
            axios.get('api/user?page=' + page)
                .then(response => {
                    this.users = response.data;
                });
        }
    },
    created() {
        this.loadUser();
        Fire.$on('searching', () => {
            let query = this.$parent.search;
            axios.get('api/findUser?q=' + query)
            .then((data) => {
                this.users = data.data
            })
            .catch(() => {

            })
        })
        // Fire.$on('AfterCreate', () => {
        //     this.loadUser();
        // });
        // setInterval(() => this.loadUser(), 3000)
    }
};

</script>
