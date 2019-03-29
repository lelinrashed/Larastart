<style scoped>
    .widget-user .widget-user-header {
    height: 250px;
}
.widget-user .widget-user-image > img {
    width: 150px;
    height: 130px;
}
.widget-user .widget-user-image {
    top: 125px;
    left: 50%;
    margin-left: -87px;
}
.widget-user .card-footer {
    padding-top: 15px;
}
</style>
<template>
    <div class="container">
        <div class="row justify-content-center mt-5">
            <div class="col-md-12">
                <div class="card card-widget widget-user">
                    <!-- Add the bg color to the header using any of the bg-* classes -->
                    <div class="widget-user-header text-white" style="background: url('./img/user_cover.jpg') no-repeat center center / cover">
                        <h3 class="widget-user-username">{{ this.form.name }}</h3>
                        <h5 class="widget-user-desc">{{ this.form.bio }}</h5>
                    </div>
                    <div class="widget-user-image">
                        <img class="img-circle" :src="getProfilePhoto()" alt="img">
                    </div>
                    <div class="card-footer">
                        <div class="row">
                            <div class="col-sm-4 border-right">
                                <div class="description-block">
                                    <h5 class="description-header">3,200</h5>
                                    <span class="description-text">SALES</span>
                                </div>
                                <!-- /.description-block -->
                            </div>
                            <!-- /.col -->
                            <div class="col-sm-4 border-right">
                                <div class="description-block">
                                    <h5 class="description-header">13,000</h5>
                                    <span class="description-text">FOLLOWERS</span>
                                </div>
                                <!-- /.description-block -->
                            </div>
                            <!-- /.col -->
                            <div class="col-sm-4">
                                <div class="description-block">
                                    <h5 class="description-header">35</h5>
                                    <span class="description-text">PRODUCTS</span>
                                </div>
                                <!-- /.description-block -->
                            </div>
                            <!-- /.col -->
                        </div>
                        <!-- /.row -->
                    </div>
                </div>
            </div>
            <div class="col-md-12 mt-3">
                <div class="card">
                    <div class="card-header p-2">
                        <ul class="nav nav-pills">
                            <li class="nav-item"><a class="nav-link active" href="#activity" data-toggle="tab">Activity</a></li>
                            <li class="nav-item"><a class="nav-link" href="#settings" data-toggle="tab">Settings</a></li>
                        </ul>
                    </div><!-- /.card-header -->
                    <div class="card-body">
                        <div class="tab-content">
                            <div class="active tab-pane" id="activity">
                                This is activity page
                            </div>
                            <div class="tab-pane" id="settings">
                                <form class="form-horizontal">
                                    <div class="form-group">
                                        <label for="inputName" class="col-sm-2 control-label">Name</label>
                                        <div class="col-sm-12">
                                            <input v-model="form.name" :class="{ 'is-invalid': form.errors.has('name') }" type="Name" class="form-control" id="inputName" placeholder="Name" autofocus="true">
                                            <has-error :form="form" field="name"></has-error>
                                        </div>
                                    </div>
                                    <div class="form-group">
                                        <label for="inputEmail" class="col-sm-2 control-label">Email</label>
                                        <div class="col-sm-12">
                                            <input v-model="form.email" :class="{ 'is-invalid': form.errors.has('email') }" type="email" class="form-control" id="inputEmail" placeholder="Email">
                                            <has-error :form="form" field="email"></has-error>
                                        </div>
                                    </div>
                                    <div class="form-group">
                                        <label for="bio" class="col-sm-2 control-label">Bio</label>
                                        <div class="col-sm-12">
                                            <textarea v-model="form.bio" :class="{ 'is-invalid': form.errors.has('bio') }" class="form-control" id="bio" placeholder="User Bio"></textarea>
                                            <has-error :form="form" field="bio"></has-error>
                                        </div>
                                    </div>
                                    <div class="form-group">
                                        <label for="profilePhoto" class="col-sm-2 control-label">Profile Photo</label>
                                        <div class="col-sm-12">
                                            <input name="photo" type="file" v-on:change="updateProfilePhoto" class="" id="profilePhoto">
                                        </div>
                                    </div>
                                    <div class="form-group">
                                        <label for="password" class="col-sm-12 control-label">Password (leave empty if not changing)</label>
                                        <div class="col-sm-12">
                                            <input v-model="form.password" :class="{ 'is-invalid': form.errors.has('password') }" type="password" class="form-control" id="password" placeholder="Password">
                                            <has-error :form="form" field="password"></has-error>
                                        </div>
                                    </div>
                                    <div class="form-group">
                                        <div class="col-sm-offset-2 col-sm-10">
                                            <button type="submit" v-on:click.prevent="updateInfo" class="btn btn-secondary">Update</button>
                                        </div>
                                    </div>
                                </form>
                            </div>
                        </div>
                        <!-- /.tab-content -->
                    </div><!-- /.card-body -->
                </div>
                <!-- /.nav-tabs-custom -->
            </div>
        </div>
    </div>
</template>
<script>
export default {
    data() {
        return {
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
    mounted() {
        console.log('Component mounted.');
    },
    methods: {
        updateProfilePhoto(e) {
            let file = e.target.files[0];
            console.log(file);
            let reader = new FileReader();
            if (file['size'] < 2111775) {
                reader.onloadend = (file) => {
                    // console.log('RESULT', reader.result);
                    this.form.photo = reader.result;
                }
                reader.readAsDataURL(file);
            } else {
                toast.fire({
                    type: 'error',
                    title: 'Upload size max 2MB'
                })
            }
        },
        updateInfo() {
            this.$Progress.start();
            this.form.put('api/profile/')
                .then(() => {
                    this.getUser();
                    toast.fire({
                        type: 'success',
                        title: 'User info update successfully'
                    })
                    this.$Progress.finish();
                })
                .catch(() => {


                    this.$Progress.fail();
                })
        },
        getProfilePhoto() {
            let photo = (this.form.photo.length > 200) ? this.form.photo:"img/profile/" + this.form.photo;
            // return "img/profile/"+ this.form.photo;
            return photo
        },
        getUser() {
            axios.get('api/profile')
            .then(({ data }) => {
                this.form.fill(data)
            })
            .catch()
        }
    },
    created() {
        this.getUser()
    }
};

</script>
