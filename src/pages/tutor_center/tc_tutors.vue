<template>
  <div class="bg-light" style="height: 100vh;">

    <div class="container-fluid" style="position: relative;">
      <div class="row">
        <div class="col-lg-2">
        <!--side nav-->
        <aside>
          <p>{{ currentUser }}</p>
          <router-link to="/dashboard" class="activeBtn">
            <i class="bi bi-speedometer"></i>
            Dashboard
          </router-link>
          <button id="current" type="button" disabled>
            <i class="bi bi-clipboard-check-fill"></i>
            My Tutors
          </button>
          <router-link to="/reviews" class="activeBtn">
            <i class="bi bi-telephone-plus-fill"></i>
            Reviews
          </router-link>
          <router-link to="/myaccount" class="activeBtn">
            <i class="bi bi-telephone-plus-fill"></i>
            My Account
          </router-link>
          <router-link to="/login" class="activeBtn logoutBtn"
          @click="logoutClicked()">
            <i class="bi bi-box-arrow-left"></i>
              Logout
          </router-link>
        </aside>
        <!--end of side nav-->  
      </div>
      
      <div class="col-lg-10 pt-5">
        <h4>Your Current Tutors</h4>
        <!-- <router-link class="btn btn-primary"  to="/add-tutor">
            Add Tutor/s
          </router-link> -->
          <button type="button" class="btn btn-primary" 
          data-bs-toggle="modal" data-bs-target="#exampleModal"
          @click="">
            Register a Tutor
          </button>
        <table class="table table-responsive bg-light">
          <thead>
          <tr>
            <th>Full Name</th>
            <th>Email</th>
            <th>Birthdate</th>
            <th>Contact #</th>
            <th>Interests</th>
            <th>Rating</th>
            <!-- <th>Remove</th> -->
          </tr>
          </thead>
          <tbody>
            <tr v-for="learner of tutorList">
              <td>{{ learner.userFirstname + ' ' + learner.userLastname }}</td>
              <td>{{ learner.userEmail }}</td>
              <td>{{ learner.userBdate }}</td>
              <td>{{ learner.userContact }}</td>
              <td>
                <span class="badge text-bg-dark" v-for="tag in learner.userTag">
                  {{ tag }}
                </span><br>
              </td>
              <td>{{ learner.userRating }}</td>
              <!-- <td>
                <i class="bi bi-x-circle-fill text-danger buttonIcon"
                @click="remove()"></i>
              </td> -->
            </tr>
          </tbody>
        </table>
      </div>
      </div>
    </div>
  </div>
  <div class="modal fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="exampleModalLabel">
            Add a Tutor to {{ currentUser }}
          </h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          <input class="modal-input" type="text" placeholder="Email" 
            v-model="userEmail">
          <input class="modal-input" type="text" placeholder="Firstname" 
            v-model="userFirstname">
          <input class="modal-input" type="text" placeholder="Lastname"
            v-model="userLastname">
          <input class="modal-input" type="date" 
            v-model="userBdate">
          <input class="modal-input" type="text" placeholder="Address"
            v-model="userAddress">
          <input class="modal-input" placeholder="Contact #"
            v-model="userContact">
          <input class="modal-input" placeholder="Topic I'm interested"
            v-model="tags">
          <input class="modal-input" placeholder="Password"
            v-model="userPassword">
          <!-- <input class="modal-input" placeholder="Confirm Password"
            v-model="userPassword2"> -->
          <!-- <label>Tutor's Valid ID</label>
          <input class="modal-input" type="file" name="myImage" 
            ref="fileInput" accept="image/png, image/gif, image/jpeg"
            @change="handleFileChange()"/>

          <label>Tutor's Selfie</label>  
          <input class="modal-input" type="file" name="myImage" 
            ref="fileInput" accept="image/png, image/gif, image/jpeg"
            @change="handleFileChange()"/> -->
          <div class="modal-buttons">
            <button id="register" type="button" class="btn btn-primary" 
            data-bs-dismiss="modal" @click="modalConfirm">
            Confirm
            </button>
            <button id="cancel" type="button" class="btn btn-danger" data-bs-dismiss="modal"
              @click="popupCancel">
              <i class="bi bi-x-lg"></i>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import { ref } from 'vue';
  import { auth } from '../../firebase';
  import { db } from '../../firebase';
  import router from '../../router';
  import { doc, setDoc, getDoc, arrayUnion } from "firebase/firestore";
  import { getAuth, setPersistence, signInWithEmailAndPassword, browserSessionPersistence, onAuthStateChanged, createUserWithEmailAndPassword } from "firebase/auth";
  import { collection, getDocs, updateDoc, deleteDoc } from "firebase/firestore";
  import { getStorage, getDownloadURL } from "firebase/storage";
  import { ref as storageRef } from "firebase/storage";
  import * as bootstrap from 'bootstrap';
  import axios from 'axios';

    export default{
      data(){
        return {
          currentUser: '',
          tutorList: [],
          // tags: [],
          tags: '',

          //Tutor info
          userEmail: '',
          userFirstname: '',
          userLastname: '',
          userAddress: '',
          userPassword: '',
          userBdate: '',
          userContact: '',

          userAge: '',
          userRating: '',
          userSessionPrice: '',
          userTag: [],
          userTutoringCenter: '',
          userType: 'tutor',
          userUid: '',
          userAbout: '',
          password2: '',
        }
      },
      setup(){
      },
      methods:{
        logoutClicked(){
          auth.signOut();
          sessionStorage.setItem("isLoggedIn", false);
          console.log(sessionStorage.getItem("isLoggedIn"))
          sessionStorage.setItem("userType", null);
          console.log(this.currentUser);
        },
        async modalConfirm(){
          try {
            alert("Registration sent!");
            await setDoc(doc(db, "all_users", "tutor", "users", this.userEmail), {
              userEmail: this.userEmail,
              userFirstname: this.userFirstname,
              userLastname: this.userLastname,
              userAddress: this.userAddress,
              userPassword:  this.userPassword,
              userBdate: this.userBdate,
              userContact: this.userContact,

              userAge: '',
              userRating: '0',
              userSessionPrice: '0',
              userTag: [
                this.tags,
              ],
              userTutoringCenter: this.currentUser,
              userType: 'tutor',
              userUid: crypto.randomUUID(),
              userAbout: '',
            });

            await updateDoc(doc(db, "all_users", "tutor_center", "users", this.currentUser), {
              tutors: arrayUnion(this.userEmail)
            });

            const auth = getAuth();
            createUserWithEmailAndPassword(auth, this.userEmail, this.userPassword)

            .then(async (userCredential) => {
              // Signed up
              const user = userCredential.user;
              console.log(user);
            }).catch((error) => {
              const errorCode = error.code;
              const errorMessage = error.message;
              alert(error.message);
            });

            this.loadTable();
          } catch (error) {
            alert(`Error approving registration: ` + error.message);
          }
        },
        async loadTable(){
          const querySnapshot = await getDocs(collection(db, "all_users/tutor/users"));
          querySnapshot.forEach((doc) => {
            if(doc.data().userTutoringCenter === this.currentUser)
              this.tutorList.push(doc.data());
            console.log(this.tutorList)
          });
        }
      },
      async created(){
        const auth = getAuth();
        onAuthStateChanged(auth, (user) => {
          if (user) {
            // User is signed in, see docs for a list of available properties
            // https://firebase.google.com/docs/reference/js/auth.user
            // ...
            // this.currentUser = user;
            // console.log(user.email);
            this.currentUser = user.email;
            
          } else {
            auth.signOut();
            sessionStorage.setItem("isLoggedIn", false);
            console.log(sessionStorage.getItem("isLoggedIn"))
            sessionStorage.setItem("userType", null);
            console.log(this.currentUser);
          }
        });

        const querySnapshot = await getDocs(collection(db, "all_users/tutor/users"));
        querySnapshot.forEach((doc) => {
          if(doc.data().userTutoringCenter === this.currentUser) {
            this.tutorList.push(doc.data());
          }
        });
      }
    };
</script>

<style scoped>
  @import "bootstrap/dist/css/bootstrap.css";

  .buttonIcon {
    cursor: pointer;
  }
  
  aside {
    color: #fff;
    width: 200px;
    padding-left: 20px;
    height: 100vh;
    background:  #006df3;
    border-top-right-radius: 40px;
  }

  aside a {
    font-size: 12px;
    color: #fff;
    display: block;
    padding: 12px;
    padding-left: 30px;
    text-decoration: none;
    -webkit-tap-highlight-color:transparent;
  }

  aside .activeBtn{
    font-size: 12px;
    color: #fff;
    display: block;
    padding: 12px;
    padding-left: 30px;
    text-decoration: none;
    -webkit-tap-highlight-color:transparent;
    border: 0;
    text-decoration: none;
    background:  #006df3;
    width: 100%;
    text-align: left;
    transition: 0.2s ease;
  }

  aside .activeBtn:hover {
    color: #3f5efb;
    background: #fff;
    outline: none;
    position: relative;
    background-color: #fff;
    border-top-left-radius: 20px;
    border-bottom-left-radius: 20px;
  }

  aside .activeBtn i, aside button i {
    margin-right: 5px;

  }

  aside .activeBtn:hover::after {
    content: "";
    position: absolute;
    background-color: transparent;
    bottom: 100%;
    right: 0;
    height: 35px;
    width: 35px;
    border-bottom-right-radius: 18px;
    box-shadow: 0 20px 0 0 #fff;
  }

  aside .activeBtn:hover::before {
    content: "";
    position: absolute;
    background-color: transparent;
    top: 38px;
    right: 0;
    height: 35px;
    width: 35px;
    border-top-right-radius: 18px;
    box-shadow: 0 -20px 0 0 #fff;
  }

  aside button[disabled] {
    font-size: 12px;
    color: #334dbe;
    display: block;
    padding: 12px;
    padding-left: 30px;
    text-decoration: none;
    border-top-left-radius: 20px;
    border-bottom-left-radius: 20px;
    -webkit-tap-highlight-color:transparent;
    border: 0;
    text-decoration: none;
    background:#ffdd02;
    width: 100%;
    text-align: left;
    transition: 0.2s ease;
  }

  aside p {
    margin: 0;
    padding: 40px 0;
  }
  aside .logoutBtn:hover {
    color: #ffffff !important;
    background:  #ff3045  !important;
  }
  aside .logoutBtn:hover::after {
    box-shadow: 0 20px 0 0 #ff3045 !important;
  }
  aside .logoutBtn:hover:before {
    box-shadow: 0 -20px 0 0 #ff3045   !important;
  }

  .modal-input{
    width: 100%;
    margin-bottom: 2%;
  }
  .modal-buttons{
    width: 100%;
  }
  #register{
    width: 88%;
    margin-right: 2%;
  }
  #cancel{
    width: 10%;
  }
</style>