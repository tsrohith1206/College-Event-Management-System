<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>College Event Management System</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
:root{
  --primary:#4f46e5;
  --bg:#f4f6ff;
  --card:#ffffff;
}

*{box-sizing:border-box;font-family:Segoe UI, sans-serif}

body{
  margin:0;
  background:var(--bg);
}

header{
  background:var(--primary);
  color:white;
  padding:15px;
  text-align:center;
  font-size:22px;
  font-weight:bold;
}

.container{
  max-width:900px;
  margin:auto;
  padding:20px;
}

.card{
  background:var(--card);
  padding:20px;
  border-radius:12px;
  box-shadow:0 8px 20px rgba(0,0,0,.08);
  margin-bottom:20px;
}

h2{margin-top:0}

button{
  background:var(--primary);
  color:white;
  border:none;
  padding:10px 18px;
  border-radius:8px;
  cursor:pointer;
  font-weight:600;
}

input,select{
  width:100%;
  padding:10px;
  margin:8px 0;
  border-radius:6px;
  border:1px solid #ccc;
}

.event{
  border-left:5px solid var(--primary);
  padding:10px;
  margin:10px 0;
}

.hidden{display:none}

.logout{
  float:right;
  font-size:14px;
  cursor:pointer;
}
</style>
</head>

<body>

<header>
College Event Management System
<span class="logout hidden" onclick="logout()">Logout</span>
</header>

<div class="container">

<!-- ROLE SELECTION -->
<div class="card" id="roleCard">
<h2>Select Role</h2>
<button onclick="selectRole('student')">Student</button>
<button onclick="selectRole('organizer')">Organizer</button>
</div>

<!-- LOGIN -->
<div class="card hidden" id="loginCard">
<h2>Login</h2>
<input id="email" placeholder="Email">
<input id="password" type="password" placeholder="Password">
<button onclick="login()">Login</button>
</div>

<!-- STUDENT DASHBOARD -->
<div class="hidden" id="studentDash">
<div class="card">
<h2>Student Dashboard</h2>
<div id="studentEvents"></div>
</div>
</div>

<!-- ORGANIZER DASHBOARD -->
<div class="hidden" id="organizerDash">

<div class="card">
<h2>Create Event</h2>
<input id="eTitle" placeholder="Event Title">
<input id="eDate" type="date">
<input id="eVenue" placeholder="Venue">
<button onclick="createEvent()">Create</button>
</div>

<div class="card">
<h2>My Events</h2>
<div id="organizerEvents"></div>
</div>

</div>

</div>

<script>
let role = "";

let events = JSON.parse(localStorage.getItem("events")) || [];

function selectRole(r){
  role = r;
  document.getElementById("roleCard").classList.add("hidden");
  document.getElementById("loginCard").classList.remove("hidden");
}

function login(){
  document.querySelector(".logout").classList.remove("hidden");
  document.getElementById("loginCard").classList.add("hidden");

  if(role==="student"){
    document.getElementById("studentDash").classList.remove("hidden");
    loadStudentEvents();
  }else{
    document.getElementById("organizerDash").classList.remove("hidden");
    loadOrganizerEvents();
  }
}

function logout(){
  location.reload();
}

function createEvent(){
  let event = {
    title:eTitle.value,
    date:eDate.value,
    venue:eVenue.value,
    approved:true,
    registrations:0
  };
  events.push(event);
  localStorage.setItem("events",JSON.stringify(events));
  loadOrganizerEvents();
}

function loadOrganizerEvents(){
  organizerEvents.innerHTML="";
  events.forEach(e=>{
    organizerEvents.innerHTML+=`
      <div class="event">
      <b>${e.title}</b><br>
      ${e.date} | ${e.venue}<br>
      Registrations: ${e.registrations}
      </div>`;
  });
}

function loadStudentEvents(){
  studentEvents.innerHTML="";
  events.filter(e=>e.approved).forEach((e,i)=>{
    studentEvents.innerHTML+=`
      <div class="event">
      <b>${e.title}</b><br>
      ${e.date} | ${e.venue}<br>
      <button onclick="register(${i})">Register</button>
      </div>`;
  });
}

function register(i){
  events[i].registrations++;
  localStorage.setItem("events",JSON.stringify(events));
  alert("Registered Successfully!");
}
</script>

</body>
</html>
college-event-system/
│
├── index.html        (login + role check)
├── student.html      (student dashboard)
├── organizer.html    (organizer dashboard)
├── admin.html        (admin panel)
│
├── css/
│   └── style.css
│
├── js/
│   ├── firebase.js   (Firebase config)
│   ├── auth.js       (login/signup)
│   ├── student.js
│   ├── organizer.js
│   └── admin.js
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.0/firebase-app.js";
import { getAuth } from "https://www.gstatic.com/firebasejs/10.7.0/firebase-auth.js";
import { getFirestore } from "https://www.gstatic.com/firebasejs/10.7.0/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "YOUR_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "XXXX",
  appId: "XXXX"
};

export const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);
</script>
import { auth, db } from "./firebase.js";
import { createUserWithEmailAndPassword, signInWithEmailAndPassword } 
from "https://www.gstatic.com/firebasejs/10.7.0/firebase-auth.js";
import { doc, setDoc, getDoc } 
from "https://www.gstatic.com/firebasejs/10.7.0/firebase-firestore.js";

export async function signup(email, password, role){
  const user = await createUserWithEmailAndPassword(auth, email, password);
  await setDoc(doc(db, "users", user.user.uid), {
    email,
    role
  });
}

export async function login(email, password){
  const user = await signInWithEmailAndPassword(auth, email, password);
  const snap = await getDoc(doc(db, "users", user.user.uid));
  return snap.data().role;
}
login(email, password).then(role=>{
  if(role==="student") location.href="student.html";
  if(role==="organizer") location.href="organizer.html";
  if(role==="admin") location.href="admin.html";
});
import { auth, db } from "./firebase.js";
import { collection, getDocs, addDoc } 
from "https://www.gstatic.com/firebasejs/10.7.0/firebase-firestore.js";

auth.onAuthStateChanged(async user=>{
  if(!user) location.href="index.html";

  const eventsDiv = document.getElementById("events");
  eventsDiv.innerHTML = "";

  const snap = await getDocs(collection(db,"events"));
  snap.forEach(doc=>{
    const e = doc.data();
    if(e.status === "approved"){
      eventsDiv.innerHTML += `
        <div class="card">
          <h3>${e.title}</h3>
          <p>${e.date} | ${e.venue}</p>
          <button onclick="register('${doc.id}')">Register</button>
        </div>`;
    }
  });
});

window.register = async (eventId)=>{
  await addDoc(collection(db,"registrations"),{
    eventId,
    student: auth.currentUser.email
  });
  alert("Registered successfully");
};
import { auth, db } from "./firebase.js";
import { addDoc, collection, getDocs, query, where } 
from "https://www.gstatic.com/firebasejs/10.7.0/firebase-firestore.js";

auth.onAuthStateChanged(user=>{
  if(!user) location.href="index.html";
});

window.createEvent = async ()=>{
  const title = eName.value;
  const date = eDate.value;
  const venue = eVenue.value;

  await addDoc(collection(db,"events"),{
    title,
    date,
    venue,
    organizer: auth.currentUser.email,
    status: "pending"
  });

  alert("Event submitted for approval");
  loadMyEvents();
};

async function loadMyEvents(){
  const q = query(
    collection(db,"events"),
    where("organizer","==",auth.currentUser.email)
  );
  const snap = await getDocs(q);
  myEvents.innerHTML="";
  snap.forEach(doc=>{
    myEvents.innerHTML += `
      <div class="card">
        ${doc.data().title} - ${doc.data().status}
      </div>`;
  });
}
import { auth, db } from "./firebase.js";
import { collection, getDocs, updateDoc, doc } 
from "https://www.gstatic.com/firebasejs/10.7.0/firebase-firestore.js";

auth.onAuthStateChanged(async user=>{
  if(!user) location.href="index.html";
  loadEvents();
});

async function loadEvents(){
  const snap = await getDocs(collection(db,"events"));
  adminEvents.innerHTML="";
  snap.forEach(d=>{
    const e = d.data();
    adminEvents.innerHTML += `
      <div class="card">
        <b>${e.title}</b> (${e.status})
        <button onclick="approve('${d.id}')">Approve</button>
        <button onclick="reject('${d.id}')">Reject</button>
      </div>`;
  });
}

window.approve = async id=>{
  await updateDoc(doc(db,"events",id),{status:"approved"});
  loadEvents();
};

window.reject = async id=>{
  await updateDoc(doc(db,"events",id),{status:"rejected"});
  loadEvents();
};
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {

    match /users/{uid} {
      allow read, write: if request.auth != null;
    }

    match /events/{id} {
      allow read: if true;
      allow create: if request.auth != null;
      allow update: if request.auth != null;
    }

    match /registrations/{id} {
      allow read, write: if request.auth != null;
    }
  }
}
