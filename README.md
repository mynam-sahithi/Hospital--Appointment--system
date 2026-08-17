# Hospital--Appointment--system
Technologies
Frontend: React
Backend: Node.js + Express
Database: SQLite
Authentication: JWT
API: REST API
3. Simple folder structure
hospital-appointment/
│
├── backend/
│   ├── server.js
│   ├── database.js
│   ├── package.json
│   └── .env
│
└── frontend/
    ├── src/
    │   ├── App.jsx
    │   ├── main.jsx
    │   ├── Login.jsx
    │   ├── Register.jsx
    │   ├── Doctors.jsx
    │   ├── BookAppointment.jsx
    │   └── MyAppointments.jsx
    └── package.json
    reate the backend:

mkdir hospital-appointment
cd hospital-appointment
mkdir backend
cd backend
npm init -y
npm install express cors sqlite3 bcryptjs jsonwebtoken dotenv
5. backend/.env
PORT=5000
JWT_SECRET=mysecret123
6. backend/database.js
const sqlite3 = require("sqlite3").verbose();


const db = new sqlite3.Database("hospital.db");


db.serialize(() => {


  db.run(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      email TEXT UNIQUE NOT NULL,
      password TEXT NOT NULL,
      role TEXT DEFAULT 'patient'
    )
  `);


  db.run(`
    CREATE TABLE IF NOT EXISTS doctors (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      specialization TEXT NOT NULL
    )
  `);


  db.run(`
    CREATE TABLE IF NOT EXISTS appointments (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      patientName TEXT NOT NULL,
      patientEmail TEXT NOT NULL,
      doctorId INTEGER NOT NULL,
      date TEXT NOT NULL,
      time TEXT NOT NULL,
      reason TEXT NOT NULL,
      status TEXT DEFAULT 'Pending'
    )
  `);


});


module.exports = db;
7. backend/server.js
require("dotenv").config();
// Add doctor
app.post("/doctors", (req, res) => {


  const {
    name,
    specialization
  } = req.body;


  db.run(
    `INSERT INTO doctors
     (name, specialization)
     VALUES (?, ?)`,
    [name, specialization],
    function () {


      res.json({
        message: "Doctor added",
        id: this.lastID
      });


    }
  );
});




// Book appointment
app.post("/appointments", (req, res) => {


  const {
    patientName,
    patientEmail,
    doctorId,
    date,
    time,
    reason
  } = req.body;


  db.run(
    `INSERT INTO appointments
    (patientName, patientEmail, doctorId, date, time, reason)
    VALUES (?, ?, ?, ?, ?, ?)`,
    [
      patientName,
      patientEmail,
      doctorId,
      date,
      time,
      reason
    ],
    function () {


      res.json({
        message: "Appointment booked",
        id: this.lastID
      });


    }
  );
});




// Get patient appointments
app.get(
  "/appointments/:email",
  (req, res) => {


    db.all(
      `SELECT appointments.*, doctors.name AS doctorName
       FROM appointments
       JOIN doctors
       ON appointments.doctorId = doctors.id
       WHERE patientEmail = ?`,
      [req.params.email],
      (err, appointments) => {


        res.json(appointments);


      }
    );
  }
);




// Start server
app.listen(
  process.env.PORT || 5000,
  () => {
    console.log(
      "Server running on port 5000"
    );
  }
);
8. Start backend
cd backend
node server.js

You should see:

Server running on port 5000
Frontend

Create React application:

cd ..
npm create vite@latest frontend

Select:

React
JavaScript

Then:

cd frontend
npm install
npm install axios react-router-dom
npm run dev
9. frontend/src/App.jsx
import {
  BrowserRouter,
  Routes,
  Route,
  Link
} from "react-router-dom";


import Login from "./Login";
import Register from "./Register";
import Doctors from "./Doctors";
import BookAppointment from "./BookAppointment";
import MyAppointments from "./MyAppointments";


function App() {


  return (
    <BrowserRouter>


      <nav>
        <Link to="/">Login</Link>{" | "}


        <Link to="/register">
          Register
        </Link>{" | "}


        <Link to="/doctors">
          Doctors
        </Link>{" | "}


        <Link to="/book">
          Book Appointment
        </Link>{" | "}


        <Link to="/appointments">
          My Appointments
        </Link>
      </nav>


      <hr />


      <Routes>


        <Route
          path="/"
          element={<Login />}
        />


        <Route
          path="/register"
          element={<Register />}
        />


        <Route
          path="/doctors"
          element={<Doctors />}
        />


        <Route
          path="/book"
          element={<BookAppointment />}
        />


        <Route
          path="/appointments"
          element={<MyAppointments />}
        />


      </Routes>


    </BrowserRouter>
  );
}


export default App;
10. frontend/src/Login.jsx
import { useState } from "react";
import axios from "axios";


function Login() {


  const [email, setEmail] =
    useState("");


  const [password, setPassword] =
    useState("");


  const login = async (e) => {


    e.preventDefault();


    try {


      const response =
        await axios.post(
          "http://localhost:5000/login",
          {
            email,
            password
          }
        );


      localStorage.setItem(
        "token",
        response.data.token
      );


      localStorage.setItem(
        "user",
        JSON.stringify(
          response.data.user
        )
      );


      alert("Login successful");


    } catch (error) {


      alert(
        "Invalid email or password"
      );


    }
  };


  return (
    <div>


      <h1>Hospital Login</h1>


      <form onSubmit={login}>


        <input
          type="email"
          placeholder="Email"
          onChange={(e) =>
            setEmail(e.target.value)
          }
        />


        <br /><br />


        <input
          type="password"
          placeholder="Password"
          onChange={(e) =>
            setPassword(e.target.value)
          }
        />


        <br /><br />


        <button>
          Login
        </button>


      </form>


    </div>
  );
}


export default Login;
11. frontend/src/Register.jsx
import { useState } from "react";
import axios from "axios";


function Register() {


  const [name, setName] =
    useState("");


  const [email, setEmail] =
    useState("");


  const [password, setPassword] =
    useState("");


  const register = async (e) => {


    e.preventDefault();


    try {


      await axios.post(
        "http://localhost:5000/register",
        {
          name,
          email,
          password
        }
      );


      alert(
        "Registration successful"
      );


    } catch (error) {


      alert(
        "Registration failed"
      );


    }
  };


  return (
    <div>


      <h1>Patient Registration</h1>


      <form onSubmit={register}>


        <input
          placeholder="Name"
          onChange={(e) =>
            setName(e.target.value)
          }
        />


        <br /><br />


        <input
          type="email"
          placeholder="Email"
          onChange={(e) =>
            setEmail(e.target.value)
          }
        />


        <br /><br />


        <input
          type="password"
          placeholder="Password"
          onChange={(e) =>
            setPassword(e.target.value)
          }
        />


        <br /><br />


        <button>
          Register
        </button>


      </form>


    </div>
  );
}


export default Register;
12. frontend/src/Doctors.jsx
import { useEffect, useState } from "react";
import axios from "axios";


function Doctors() {


  const [doctors, setDoctors] =
    useState([]);


  useEffect(() => {


    axios
      .get(
        "http://localhost:5000/doctors"
      )
      .then((response) => {
        setDoctors(response.data);
      });


  }, []);


  return (
    <div>


      <h1>Our Doctors</h1>


      {doctors.map((doctor) => (


        <div key={doctor.id}>


          <h3>
            {doctor.name}
          </h3>


          <p>
            Specialization:
            {doctor.specialization}
          </p>


          <hr />


        </div>


      ))}


    </div>
  );
}


export default Doctors;
13. frontend/src/BookAppointment.jsx
import { useEffect, useState } from "react";
        doctorId,
        date,
        time,
        reason
      }
    );


    alert(
      "Appointment booked successfully"
    );
  };


  return (
    <div>


      <h1>Book Appointment</h1>


      <form onSubmit={book}>


        <select
          value={doctorId}
          onChange={(e) =>
            setDoctorId(e.target.value)
          }
        >


          <option value="">
            Select Doctor
          </option>


          {doctors.map((doctor) => (


            <option
              key={doctor.id}
              value={doctor.id}
            >
              {doctor.name}
            </option>


          ))}


        </select>


        <br /><br />


        <input
          type="date"
          onChange={(e) =>
            setDate(e.target.value)
          }
        />


        <br /><br />


        <input
          type="time"
          onChange={(e) =>
            setTime(e.target.value)
          }
        />


        <br /><br />


        <input
          placeholder="Reason"
          onChange={(e) =>
            setReason(e.target.value)
          }
        />


        <br /><br />


        <button>
          Book Appointment
        </button>


      </form>


    </div>
  );
}


export default BookAppointment;
14. frontend/src/MyAppointments.jsx
import { useEffect, useState } from "react";
import axios from "axios";


function MyAppointments() {


  const user = JSON.parse(
    localStorage.getItem("user")
  );


  const [appointments, setAppointments] =
    useState([]);


  useEffect(() => {


    if (!user) return;


    axios
      .get(
        `http://localhost:5000/appointments/${user.email}`
      )
      .then((response) => {
        setAppointments(response.data);
      });


  }, []);


  return (
    <div>


      <h1>My Appointments</h1>


      {appointments.map(
        (appointment) => (


          <div key={appointment.id}>


            <h3>
              Doctor:
              {appointment.doctorName}
            </h3>


            <p>
              Date: {appointment.date}
            </p>


            <p>
              Time: {appointment.time}
            </p>


            <p>
              Reason: {appointment.reason}
            </p>


            <p>
              Status: {appointment.status}
            </p>


            <hr />


          </div>


        )
      )}


    </div>
  );
}


export default MyAppointments;
How the project works
Patient
   ↓
Register
   ↓
Login
   ↓
View Doctors
   ↓
Select Doctor
   ↓
Select Date & Time
   ↓
Book Appointment
   ↓
Appointment Saved in Database
   ↓
View My Appointments
