<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminOrAuthor()">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">User Table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal" data-toggle="modal" data-target="#addNew">Add New <i class="fas fa-user-plus fw"></i>   </button>

                </div>
              </div>

              <!--card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Registered at</th>
                    <th>Modify</th>
                  </tr>

                  <tr v-for="user in users" :key="user.id">
                    <td>{{user.id}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td>{{user.type | upText}}</td>
                    <td>{{user.created_at | myDate}}</td>
                    <td>
                        <a href="#" @click="editModal(user)">
                          <i class="fa fa-edit blue"></i>
                          </a>
                        /
                        <a href="#" @click="deleteUser(user.id)">
                          <i class="fa fa-trash red"></i>
                          </a>
                    </td>
                  </tr>
                </tbody></table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
        </div>
        <div v-if="!$gate.isAdminOrAuthor()">
          <not-found></not-found>
        </div>
        <!-- Modal -->
<div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="exampleModalCenterTitle" aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="exampleModalCenterTitle">{{editMode?'Edit':'New'}}</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>

      <form @submit.prevent="editMode ? updateUser() : createUser()">
      <div class="modal-body">
        <div class="form-group">
            <input v-model="form.name" type="text" name="name" placeholder="name"
            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
            <has-error :form="form" field="name"></has-error>
        </div>

        <div class="form-group">
            <input v-model="form.email" type="text" name="email" placeholder="Email Address"
            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
            <has-error :form="form" field="email"></has-error>
        </div>

        <div class="form-group">
            <textarea v-model="form.bio" name="bio" id="bio" 
            placeholder="Short bio for user (optional)"
            class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
            <has-error :form="form" field="bio"></has-error>
        </div>

        <div class="form-group">
            <select name="type" v-model="form.type"  id="type"
                class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                <option value="">Select Usre Role</option>
                <option value="admin">Admin</option>
                <option value="user">Standard User</option>
                <option value="author">Author</option>
            </select>
            <has-error :form="form" field="type"></has-error>
        </div>

        <div class="form-group">
            <input v-model="form.password" type="password" name="password" placeholder="password"
            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
            <has-error :form="form" field="password"></has-error>
        </div>

      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
        <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
        <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
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
      editMode: false,
      users : {},
      form: new Form({
        id:'',
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
    updateUser(){
      this.$Progress.start()
      this.form.put('api/user/' + this.form.id)
      .then(() => {
        //success
          $('#addNew').modal('hide')
            swal(
            'Updated!',
            'Your file has been deleted.',
            'success'
                )
          Fire.$emit('RefreshNow')
          this.$Progress.finish()
      })
      .catch(() =>{
        swal("Failed!", "There was something wrong", "warning")
        this.$Progress.fail()
      })

    },
    editModal(user){
      this.editMode = true
      this.form.reset()
      $('#addNew').modal('show')
      this.form.fill(user)
    },
    newModal(){
      this.editMode = false
      this.form.reset()
      $('#addNew').modal('show')
    },
      deleteUser(id){
      swal({
          title: 'Are you sure?',
          text: "You won't be able to revert this!",
          type: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Yes, delete it!'
          }).then((result) => {
            // send request to the server
            if(result.value){
                this.$Progress.start()
                this.form.delete('api/user/' + id).then(()=>{
                swal(
                'Deleted!',
                'Your file has been deleted.',
                'success'
                )
                Fire.$emit('RefreshNow')
                this.$Progress.finish()
                }).catch(()=> {
                  swal("Failed!", "There was something wrong", "warning")
                  this.$Progress.fail()
                })
            }
       })
    },
    loadUsers(){
      if(this.$gate.isAdminOrAuthor()){
        axios.get("/api/user").then(({data}) => (this.users = data.data))
      }
    },
    createUser(){
          this.$Progress.start()
          this.form.post('api/user').then(() => {
              Fire.$emit('RefreshNow')
              $('#addNew').modal('hide')
            toast({
               type: 'success',
               title: 'User created successfully'
               })
            this.$Progress.finish()

          })
          .catch(() => {
            this.$Progress.fail()
          })
      }
  },
  created() {
    this.loadUsers()
    Fire.$on('RefreshNow',() => {
       this.loadUsers()
    })
    // setInterval(() => this.loadUsers(), 3000)
  }
}
</script>