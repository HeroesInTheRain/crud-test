<template>
  <div class="main-container">
    <div class="test-title">
        <h1 style="color: #2E4057;">CRUD Test</h1>
        <h3>Aneth Shariany Ibarra Márquez</h3>
    </div>
    <div v-if="showAlert"><b-alert variant="danger">{{ alertMessage }}</b-alert></div>
    <div v-if="showModal">
      <transition name="modal">
        <div class="modal-mask">
          <div class="modal-wrapper">
            <div class="modal-dialog" role="document">
              <div class="modal-content">
                <div class="modal-header">
                  <h5 class="modal-title">{{ modalTitle }}</h5>
                  <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true" @click="close">&times;</span>
                  </button>
                </div>
                <div class="modal-body">
                  <div class="input-group mb-3">
                    <input type="text" class="form-control" placeholder="e-mail" aria-label="email" aria-describedby="basic-addon1" v-model="newComment.email">
                  </div>
                  <div class="input-group mb-3">
                    <input type="text" class="form-control" placeholder="Title" aria-label="Title" aria-describedby="basic-addon1" v-model="newComment.name">
                  </div>
                  <div class="input-group">
                    <span class="input-group-text">Comment</span>
                    <textarea class="form-control" aria-label="Comment" v-model="newComment.body"></textarea>
                  </div>
                  <div v-if="isLoadingRequest" style="text-align: center; margin-top:15px;">L o a d i n g . . .</div>
                  <div v-if="requestError"><b-alert variant="danger">{{ alertRequestMessage }}</b-alert></div>
                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-secondary" @click="close">Close</button>
                  <button type="button" class="btn btn-warning" @click="request()">Save changes</button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </div>
    <div v-if="deleteAlertModal">
      <transition name="modal">
        <div class="modal-mask">
          <div class="modal-wrapper">
            <div class="modal-dialog" role="document">
              <div class="modal-content">
                <div class="modal-header">
                  <h5 class="modal-title">Alert!</h5>
                  <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true" @click="close">&times;</span>
                  </button>
                </div>
                <div class="modal-body">
                  <div class="input-group">
                    <span>Are you sure you want to delete this comment?</span>
                    <div v-if="isLoadingRequest" style="text-align: center; margin-top:15px;">L o a d i n g . . .</div>
                  </div>
                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-secondary" @click="close">Cancel</button>
                  <button type="button" class="btn btn-warning" @click="deleteComment()">Delete comment</button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </div>
    <div v-if="isLoading" style="text-align: center; margin-top:150px;">L o a d i n g . . .</div>
    <div v-else class="posts-container">
      <div v-for="post in posts" :key="post.id" class="post">
          <div class="post-details">
            <span>Post made by {{ post.username }}</span>
            <span>#{{ post.id }}</span>
          </div>
          <div>
            <h2>{{ post.title }}</h2>
          </div>
          <div >
            <span>{{ post.body }}</span>
          </div>
          <div class="post-comments" v-for="comment in post.comments" :key="comment.id">
            <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/7e/Circle-icons-profile.svg/768px-Circle-icons-profile.svg.png" style="width:86px;height:86px;margin-top:10px;margin-right:10px;">
            <div style="margin-left:10px; padding-right:10px; width:100%;">
              <span style="color: #F3A712; font-size:14px;">{{ comment.email }}</span>
              <h5>{{ comment.name }}</h5>
              <span>{{ comment.body }}</span>
              <div style="display: flex; justify-content:flex-end; margin-top:10px;">
                <button class="btn btn-outline-light" style="font-size:10px; width:45px;height:30px; margin-right:10px;" @click="editComment(post, comment)">Edit</button>
                <button class="btn btn-outline-light" style="font-size:10px; width:60px;height:30px;" @click="deleteCurrentComment(comment)">Delete</button>
              </div>
            </div>
          </div>
          <div class="post-details" style="justify-content: flex-end;">
              <span style="margin-top:5px; margin-right:15px;">Do you want to be part of this conversation?</span>
              <button class="btn btn-warning" style="color:white; font-weight:600; width:150px;height:40px;" @click="addComment(post)">Add a comment</button>
          </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'HelloWorld',
  data(){
    return {
      isLoading: false,
      isLoadingRequest: false,
      posts: [],
      users: [],
      alertMessage: null,
      showAlert: false,
      showModal: false,
      modalTitle: null,
      requestError: false,
      alertRequestMessage: null,
      deleteAlertModal: false,
      currentComment: null,
      newComment: {
        email: null,
        name: null,
        body: null,
        postId: null,
        commentId: null
      }
    }
  },
  methods: {
    sortComments(comments, posts){
      for (let i = 0; i < posts.length; i++){
        var commentsList = []
        for (let j = 0; j < comments.length; j++){
           if( posts[i].id == comments[j].postId){
             commentsList.push(comments[j])
           }
        }
        posts[i].comments = commentsList
        posts[i].username = this.users.find(user => user.id == posts[i].userId).username
        commentsList = []
      }
      this.posts = posts
      this.isLoading = false
    },
    addComment(post) {
      this.showModal = true
      this.modalTitle = 'Add a new comment to '+ post.username +'\'s post!'
      let editedComment = {
        email: null,
        name: null,
        body: null,
        postId: post.id,
        commentId: null
      }
      this.newComment = editedComment
    },
    editComment(post, comment) {
      this.showModal = true
      this.currentComment = comment
      this.modalTitle = 'Editing ' + comment.email + ' comment from '+ post.username +'\'s post'
      let editedComment = {
        email: comment.email,
        name: comment.name,
        body: comment.body,
        postId: post.id,
        commentId: comment.id
      }
      this.newComment = editedComment
    },
    deleteCurrentComment(comment){
      this.deleteAlertModal = true
      this.currentComment = comment
      console.log(this.currentComment)
    },
    deleteComment(){
      axios.delete('https://jsonplaceholder.typicode.com/comments/' + this.currentComment)
        .then((response) => {
          //Extra code so you can view the changes in comments
          console.log(response)
          let postToEdit = this.posts.findIndex(post => post.id == this.currentComment.postId)
          let newComments = this.posts[postToEdit].comments.filter((comment) => {
            return comment.id != this.currentComment.id
          });
          this.posts[postToEdit].comments = newComments
          /////////////////////////////////////////////////////////////
          this.deleteAlertModal = false
          this.isLoadingRequest = false
        })
        .catch((error) => {
          console.log(error)
          this.setAlert('Something went wrong while trying to delete a comment.')
        })
    },
    request(){
      this.isLoadingRequest = true
      if(this.newComment.commentId != null){
        axios.put('https://jsonplaceholder.typicode.com/comments/' + this.newComment.commentId, this.newComment)
          .then((response) => {
            //Extra code so you can view the changes in comments
            let editedComment = {
              email: response.data.email,
              name: response.data.name,
              body: response.data.body,
              postId: response.data.postId,
              id: response.data.id,
            }
            let postToEdit = this.posts.findIndex(post => post.id == editedComment.postId)
            let commentToEdit = this.posts[postToEdit].comments.findIndex(comment => comment.id == editedComment.id)
            this.posts[postToEdit].comments[commentToEdit] = editedComment
            /////////////////////////////////////////////////////////////

            this.isLoadingRequest = false
            this.showModal = false
          })
          .catch((error) => {
            console.log(error)
            this.isLoadingRequest = false
            this.setAlertRequest('Something went wrong while trying to edit this comment.')
          })
      }else{
          axios.post('https://jsonplaceholder.typicode.com/posts/' + this.newComment.postId + '/comments', this.newComment)
          .then((response) => {
            //Extra code so you can view the changes in comments
            let addedComment = {
              email: response.data.email,
              name: response.data.name,
              body: response.data.body,
              postId: response.data.postId,
              id: response.data.id,
            }
            let postToAdd = this.posts.find(post => post.id == addedComment.postId)
            postToAdd.comments.push(addedComment)
            /////////////////////////////////////////////////////////////

            this.isLoadingRequest = false
            this.showModal = false
          })
          .catch((error) => {
            console.log(error)
            this.isLoadingRequest = false
            this.setAlertRequest('Something went wrong while trying to add a new comment.')
          })
      }
    },
    close(){
      this.showModal = false
      this.isLoadingRequest = false
      this.currentComment = null
      this.deleteAlertModal = false
      this.newComment = {
        email: null,
        name: null,
        body: null,
        postId: null,
        commentId: null
      }
    },
    setAlert(message){
      this.alertMessage = message
      this.showAlert = true
      setTimeout(() => {
        this.showAlert = false
      }, 6000)
    },
    setAlertRequest(message){
      this.alertRequestMessage = message
      this.requestError = true
      setTimeout(() => {
        this.requestError = false
      }, 6000)
    }
  },
  mounted() {
    this.isLoading = true
    axios.get('https://jsonplaceholder.typicode.com/posts')
      .then((response) => {
        this.posts = response.data
        axios.get('https://jsonplaceholder.typicode.com/users')
          .then((response) => {
            this.users = response.data
            axios.get('https://jsonplaceholder.typicode.com/comments')
              .then((response) => {
                this.sortComments(response.data, this.posts)
              })
              .catch((error) => {
                console.log(error)
                this.setAlert('Something went wrong while loading Comments.')
              })
          })
          .catch((error) => {
            console.log(error)
            this.setAlert('Something went wrong while loading Users.')
          })
      })
      .catch((error) => {
        console.log(error)
        this.setAlert('Something went wrong while loading Posts.')
      })
  }

}
</script>
<style>

  .main-container {
    margin: 30px 60px 60px;
    align-content: center;
  }
  .test-title {
    background-color: #F3A712;
    color: #FFFFFF;
    border-radius: 15px;
    padding: 10px 50px 10px;
    margin-bottom: 20px;
    justify-content: space-between;
    display: flex;
    align-items: center;
  }
  .posts-container {
    background-color: #F3A712;
    border-radius: 15px;
    padding: 30px 300px 30px;
  }
  .post {
    background-color: #FFFFFF;
    color: #2E4057;
    border-radius: 15px;
    padding: 10px 100px 20px;
    margin-bottom: 20px;
  }
  .post-details{
    color: #F3A712;
    font-weight: 600;
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
    font-size: 18px;
  }
  .post-comments {
    background-color: #2E4057;
    color: #FFFFFF;
    border-radius: 15px;
    padding: 10px 15px 20px 30px;
    margin-top: 10px;
    display: flex;
  }

  .modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5);
  display: table;
  transition: opacity .5s ease;
  }

  .modal-wrapper {
    display: table-cell;
    vertical-align: middle;
  }
</style>
