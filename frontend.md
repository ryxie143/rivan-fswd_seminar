# FRONTEND INSTALLATION/SETUP

## Task 1: Project Setup

```bash
npm create vite@latest frontend -- --template react
# Choose react
# Choose javascript
cd frontend
npm install axios react-router-dom jwt-decode
```

## Task 2: Project Structure Setup

1. Inside the frontend directory, delete CSS files in the src folder (App.css and Index.css)
2. Open src/app.jsx and replace all code with:

```jsx
import react from "react"

function App() {
  return (
    <>
    
    </>
  )        
}

export default App
```

3. Go to main.jsx and remove `import './index.css'`

4. Create the following folder directories structure inside src:
   - `pages/`
   - `styles/`
   - `components/`

5. Create these files at the src level:
   - `constants.js`
   - `api.js`

6. Create an environment variable file `.env` in the frontend root

## Task 3: Configuration Files

1. In constants.js, add:

```javascript
export const ACCESS_TOKEN = "access";
export const REFRESH_TOKEN = "refresh"
```

2. In api.js, add:

```javascript
import axios from "axios";
import { ACCESS_TOKEN } from "./constants";

const api = axios.create({
    baseURL: import.meta.env.VITE_API_URL
})
```

3. In .env file, add:

```
VITE_API_URL="http://localhost:8000"
```

4. Add this in api.js:
   
```
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem(ACCESS_TOKEN);
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

export default api;
```

## Task 4: Writing Protected Routes

1. Create new file in `components/ProtectedRoute.jsx`:

```jsx
import { Navigate } from "react-router-dom";
import { jwtDecode } from "jwt-decode";
import api from "../api";
import { REFRESH_TOKEN, ACCESS_TOKEN } from "../constants";
import { useState, useEffect } from "react";


function ProtectedRoute({ children }) {
    const [isAuthorized, setIsAuthorized] = useState(null);

    useEffect(() => {
        auth().catch(() => setIsAuthorized(false))
    }, [])

    const refreshToken = async () => {
        const refreshToken = localStorage.getItem(REFRESH_TOKEN);
        try {
            const res = await api.post("/api/token/refresh/", {
                refresh: refreshToken,
            });
            if (res.status === 200) {
                localStorage.setItem(ACCESS_TOKEN, res.data.access)
                setIsAuthorized(true)
            } else {
                setIsAuthorized(false)
            }
        } catch (error) {
            console.log(error);
            setIsAuthorized(false);
        }
    };

    const auth = async () => {
        const token = localStorage.getItem(ACCESS_TOKEN);
        if (!token) {
            setIsAuthorized(false);
            return;
        }
        const decoded = jwtDecode(token);
        const tokenExpiration = decoded.exp;
        const now = Date.now() / 1000;

        if (tokenExpiration < now) {
            await refreshToken();
        } else {
            setIsAuthorized(true);
        }
    };

    if (isAuthorized === null) {
        return <div>Loading...</div>;
    }

    return isAuthorized ? children : <Navigate to="/login" />;
}

export default ProtectedRoute;
```

## Task 5: Navigation & Pages

1. Create a file in pages folder:
- `Login.jsx`
- `Register.jsx`
- `Home.jsx`
- `NotFound.jsx`

2. Configure `Home.jsx`:
```jsx
function Home() {
    return <div>Home</div>
}

export default Home
```

3. Configure `Login.jsx`:
```jsx
function Login() {
    return <div>Login</div>
}

export default Login
```

4. Configure `NotFound.jsx`:
```jsx
function NotFound() {
    return <div>NotFound</div>
}

export default NotFound
```

5. Configure `Register.jsx`:
```jsx
function Register() {
    return <div>Register</div>
}

export default Register
```

6. Add this in Apps.jsx
```jsx
import react from "react"
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom"
import Login from "./pages/Login"
import Register from "./pages/Register"
import Home from "./pages/Home"
import NotFound from "./pages/NotFound"
import ProtectedRoute from "./components/ProtectedRoute"

function Logout() {
  localStorage.clear()
  return <Navigate to="/login" />
}

function RegisterAndLogout() {
  localStorage.clear()
  return <Register />
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route
          path="/"
          element={
            <ProtectedRoute>
              <Home />
            </ProtectedRoute>
          }
        />
        <Route path="/login" element={<Login />} />
        <Route path="/logout" element={<Logout />} />
        <Route path="/register" element={<RegisterAndLogout />} />
        <Route path="*" element={<NotFound />}></Route>
      </Routes>
    </BrowserRouter>
  )
}

export default App


7. In your terminal copy and paste this:

```bash
npm install
npm run dev
```

3. Add this in your `NotFound.jsx`:
```jsx
function NotFound() {
    return <div>
        <h1>404 Not Found</h1>
        <p>The page you're looking for doesn't exist</p>
    </div>
}

export default NotFound
```

## Task 6: Making a Generic Form

1. Add a new file in the components folder and name it `Form.jsx`
```jsx
import { useState } from "react";
import api from "../api";
import { useNavigate } from "react-router-dom";
import { ACCESS_TOKEN, REFRESH_TOKEN } from "../constants";
import "../styles/Form.css"


function Form({ route, method }) {
    const [username, setUsername] = useState("");
    const [password, setPassword] = useState("");
    const [loading, setLoading] = useState(false);
    const navigate = useNavigate();

    const name = method === "login" ? "Login" : "Register";


    const handleSubmit = async (e) => {
        setLoading(true);
        e.preventDefault();

        try {
            const res = await api.post(route, { username, password })
            if (method === "login") {
                localStorage.setItem(ACCESS_TOKEN, res.data.access);
                localStorage.setItem(REFRESH_TOKEN, res.data.refresh);
                navigate("/")
            } else {
                navigate("/login")
            }
        } catch (error) {
            alert(error)
        } finally {
            setLoading(false)
        }
    };

    return (
        <form onSubmit={handleSubmit} className="form-container">
            <h1>{name}</h1>
            <input
                className="form-input"
                type="text"
                value={username}
                onChange={(e) => setUsername(e.target.value)}
                placeholder="Username"
            />
            <input
                className="form-input"
                type="password"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                placeholder="Password"
            />
            <button className="form-button" type="submit">
                {name}
            </button>
        </form>
    );
}

export default Form
```

## Task 7: Adding form styles

1. Add new file in style folder and name it `Form.css`
```css
.form-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 50px auto;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  max-width: 400px;
}

.form-input {
  width: 90%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.form-button {
  width: 95%;
  padding: 10px;
  margin: 20px 0;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
}

.form-button:hover {
  background-color: #0056b3;
}
```

2. Inside `form.jsx` add:
```jsx
import "../styles/Form.css"
```

## Task 8: Connecting the login on the register form

1. Edit the `Register.jsx`
```jsx
import Form from "../components/Form"

function Register() {
    return <Form route="/api/user/register/" method="register" />
}

export default Register

2. Edit the `Login.jsx`
```jsx
import Form from "../components/Form"

function Login() {
    return <Form route="/api/token/" method="login" />
}

export default Login
```

3. Try to run again using the
```bash
npm run dev
```
on the terminal, and control + click the link after

4. In the terminal enter this:
```bash
cd ..
cd backend
python manage.py runserver
```
copy the link shown on the terminal and paste it on the link in .env file

## Task 9: Building the homepage

1. Add this on `Home.jsx`
```jsx
import { useState, useEffect } from "react";
import api from "../api";
import Note from "../components/Note"
import "../styles/Home.css"

function Home() {
    const [notes, setNotes] = useState([]);
    const [content, setContent] = useState("");
    const [title, setTitle] = useState("");

    useEffect(() => {
        getNotes();
    }, []);

    const getNotes = () => {
        api
            .get("/api/notes/")
            .then((res) => res.data)
            .then((data) => {
                setNotes(data);
                console.log(data);
            })
            .catch((err) => alert(err));
    };

    const deleteNote = (id) => {
        api
            .delete(`/api/notes/delete/${id}/`)
            .then((res) => {
                if (res.status === 204) alert("Note deleted!");
                else alert("Failed to delete note.");
                getNotes();
            })
            .catch((error) => alert(error));
    };

    const createNote = (e) => {
        e.preventDefault();
        api
            .post("/api/notes/", { content, title })
            .then((res) => {
                if (res.status === 201) alert("Note created!");
                else alert("Failed to make note.");
                getNotes();
            })
            .catch((err) => alert(err));
    };

    return (
        <div>
            <div>
                <h2>Notes</h2>
                {notes.map((note) => (
                    <Note note={note} onDelete={deleteNote} key={note.id} />
                ))}
            </div>
            <h2>Create a Note</h2>
            <form onSubmit={createNote}>
                <label htmlFor="title">Title:</label>
                <br />
                <input
                    type="text"
                    id="title"
                    name="title"
                    required
                    onChange={(e) => setTitle(e.target.value)}
                    value={title}
                />
                <label htmlFor="content">Content:</label>
                <br />
                <textarea
                    id="content"
                    name="content"
                    required
                    value={content}
                    onChange={(e) => setContent(e.target.value)}
                ></textarea>
                <br />
                <input type="submit" value="Submit"></input>
            </form>
        </div>
    );
}

export default Home;
```

## Task 10: Building the Note Component

1. Add a new file in components folder and name it `Note.jsx`
```jsx
import React from "react";

function Note({ note, onDelete }) {
    const formattedDate = new Date(note.created_at).toLocaleDateString("en-US")

    return (
        <div className="note-container">
            <p className="note-title">{note.title}</p>
            <p className="note-content">{note.content}</p>
            <p className="note-date">{formattedDate}</p>
            <button className="delete-button" onClick={() => onDelete(note.id)}>
                Delete
            </button>
        </div>
    );
}

export default Note
```

2. In `Home.jsx` add this line:
```jsx
import Note from "../components/Note"
```

## Task 11: Frontend finishing touches

1. Add a new file inside the styles folder and name it
- `Note.css`
- `LoadingIndicator.css`
- `Home.css`

2. Configure `Home.css`
```css
/* Container for the whole page */
div {
  font-family: Arial, sans-serif;
}

/* Styles for the notes section */
.notes-section {
  margin-bottom: 2rem;
}

.notes-section h2 {
  color: #333;
  font-size: 24px;
}

/* Styles for individual notes - assuming your Note component has some container element */
.note {
  background-color: #f9f9f9;
  border-left: 5px solid #007bff;
  margin: 10px 0;
  padding: 10px 15px;
  border-radius: 5px;
}

/* Styles for the form section */
form {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  max-width: 500px;
  margin: auto;
}

form h2 {
  color: #333;
  font-size: 24px;
  margin-bottom: 20px;
}

form label {
  font-weight: bold;
  margin-top: 10px;
}

form input,
form textarea {
  width: 100%;
  padding: 8px;
  margin: 8px 0 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

form input[type="submit"] {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

form input[type="submit"]:hover {
  background-color: #0056b3;
}
```

2. Configure `LoadingIndicator.css`
```css
.loader-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.loader {
  border: 5px solid #f3f3f3; /* Light grey */
  border-top: 5px solid #3498db; /* Blue */
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
```

3. Configure `Note.css`
```css
.note-container {
  padding: 10px;
  margin: 20px 0;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.note-title {
  color: #333;
}

.note-content {
  color: #666;
}

.note-date {
  color: #999;
  font-size: 0.8rem;
}

.delete-button {
  background-color: #f44336; /* Red */
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.delete-button:hover {
  background-color: #d32f2f; /* Darker red */
}
```

4. Add this line on `Home.jsx` 
```jsx
import "../styles/Home.css"
```
and on `Note.jsx`
```jsx
import "../styles/Note.css"
```

5. Add a new file inside components folder and name it LoadingIndicator.jsx
```jsx
import "../styles/LoadingIndicator.css"

const LoadingIndicator = () => {
    return <div className="loading-container">
        <div className="loader"></div>
    </div>
}

export default LoadingIndicator
```

6. Insert this on `Form.jsx` above the button code
```jsx
import LoadingIndicator from "./LoadingIndicator";

{loading && <LoadingIndicator />}
```
