# To Update the Router
This document provides the steps to update the boiler-plate code from react-router-dom 5 to react-router-dom 6.

1. Change line 12 of `package.json` from 
```"react-router-dom": "^5.2.0",```
to
```"react-router-dom": "^6.2.1",```

1. In your teminal navigate to your nutshell project.  It should be named nutshell-{your team name}.  

    - *Check you are in the correct spot, do an `ls`.  You should see a `package.json` file.  

    - type `npm install` in the terminal.  This will take a while to complete (just like the first time).

1. Replace the code in the following files.
    > Nutshell.js
    ```js
    import React,  {useState} from "react"
    import { Route, Link } from "react-router-dom"
    import { ApplicationViews } from "./ApplicationViews"
    import { NavBar } from "./nav/NavBar"
    import { Login } from "./auth/Login"
    import { Register } from "./auth/Register"
    import "./Nutshell.css"

    export const Nutshell = () => {
      const [isAuthenticated, setIsAuthenticated] = useState(sessionStorage.getItem("nutshell_user") !== null)

      const setAuthUser = (user) => {
          sessionStorage.setItem("nutshell_user", JSON.stringify(user))
          setIsAuthenticated(sessionStorage.getItem("nutshell_user") !== null)
      }

      const clearUser = () => {
          sessionStorage.clear();
          setIsAuthenticated(sessionStorage.getItem("nutshell_user") !== null)
      }

      return (
        <>
          <NavBar clearUser={clearUser} isAuthenticated={isAuthenticated}/>
          <ApplicationViews 
              setAuthUser={setAuthUser}
              isAuthenticated={isAuthenticated}
              setIsAuthenticated={setIsAuthenticated}
          />
        </>
      )
    }
    ```
    > auth/login.js with the below code.
    ```js
    import React, { useState } from "react"
    import { Link, useNavigate } from "react-router-dom";
    import "./Login.css"


    export const Login = ({setAuthUser}) => {
        const [loginUser, setLoginUser] = useState({ email: "" })
        const [existDialog, setExistDialog] = useState(false)

        const navigate = useNavigate()

        const handleInputChange = (event) => {
            const newUser = { ...loginUser }
            newUser[event.target.id] = event.target.value
            setLoginUser(newUser)
        }


        const existingUserCheck = () => {
            // If your json-server URL is different, please change it below!
            return fetch(`http://localhost:8088/users?email=${loginUser.email}`)
                .then(res => res.json())
                .then(user => user.length ? user[0] : false)
        }

        const handleLogin = (e) => {
            e.preventDefault()

            existingUserCheck()
                .then(exists => {
                    if (exists) {
                        // The user id is saved under the key nutshell_user in session Storage. Change below if needed!
                        setAuthUser(exists)
                        navigate("/")
                    } else {
                        setExistDialog(true)
                    }
                })
        }

        return (
            <main className="container--login">
                <dialog className="dialog dialog--auth" open={existDialog}>
                    <div>User does not exist</div>
                    <button className="button--close" onClick={e => setExistDialog(false)}>Close</button>
                </dialog>
                <section>
                    <form className="form--login" onSubmit={handleLogin}>
                        <h1>Nutshell</h1>
                        <h2>Please sign in</h2>
                        <fieldset>
                            <label htmlFor="inputEmail"> Email address </label>
                            <input type="email"
                                id="email"
                                className="form-control"
                                placeholder="Email address"
                                required autoFocus
                                value={loginUser.email}
                                onChange={handleInputChange} />
                        </fieldset>
                        <fieldset>
                            <button type="submit">
                                Sign in
                            </button>
                        </fieldset>
                    </form>
                </section>
                <section className="link--register">
                    <Link to="/register">Register for an account</Link>
                </section>
            </main>
        )
    }
    ```
    > auth/Register.js 

    ```js
    import React, { useState } from "react"
    import { useNavigate } from "react-router-dom";

    import "./Login.css"

    export const Register = () => {

        const [registerUser, setRegisterUser] = useState({ firstName: "", lastName: "", email: "" })
        const [conflictDialog, setConflictDialog] = useState(false)

        const navigate = useNavigate()

        const handleInputChange = (event) => {
            const newUser = { ...registerUser }
            newUser[event.target.id] = event.target.value
            setRegisterUser(newUser)
        }

        const existingUserCheck = () => {
            // If your json-server URL is different, please change it below!
            return fetch(`http://localhost:8088/users?email=${registerUser.email}`)
                .then(res => res.json())
                .then(user => !!user.length)
        }

        const handleRegister = (e) => {
            e.preventDefault()

            existingUserCheck()
                .then((userExists) => {
                    if (!userExists) {
                        // If your json-server URL is different, please change it below!
                        fetch("http://localhost:8088/users", {
                            method: "POST",
                            headers: {
                                "Content-Type": "application/json"
                            },
                            body: JSON.stringify({
                                email: registerUser.email,
                                name: `${registerUser.firstName} ${registerUser.lastName}`
                            })
                        })
                            .then(res => res.json())
                            .then(createdUser => {
                                if (createdUser.hasOwnProperty("id")) {
                                    // The user id is saved under the key nutshell_user in session Storage. Change below if needed!
                                    sessionStorage.setItem("nutshell_user", createdUser.id)
                                    navigate("/")
                                }
                            })
                    }
                    else {
                        setConflictDialog(true)
                    }
                })

        }

        return (
            <main style={{ textAlign: "center" }}>

                <dialog className="dialog dialog--password" open={conflictDialog}>
                    <div>Account with that email address already exists</div>
                    <button className="button--close" onClick={e => setConflictDialog(false)}>Close</button>
                </dialog>

                <form className="form--login" onSubmit={handleRegister}>
                    <h1 className="h3 mb-3 font-weight-normal">Please Register for Application Name</h1>
                    <fieldset>
                        <label htmlFor="firstName"> First Name </label>
                        <input type="text" name="firstName" id="firstName" className="form-control" placeholder="First name" required autoFocus value={registerUser.firstName} onChange={handleInputChange} />
                    </fieldset>
                    <fieldset>
                        <label htmlFor="lastName"> Last Name </label>
                        <input type="text" name="lastName" id="lastName" className="form-control" placeholder="Last name" required value={registerUser.lastName} onChange={handleInputChange} />
                    </fieldset>
                    <fieldset>
                        <label htmlFor="inputEmail"> Email address </label>
                        <input type="email" name="email" id="email" className="form-control" placeholder="Email address" required value={registerUser.email} onChange={handleInputChange} />
                    </fieldset>
                    <fieldset>
                        <button type="submit"> Sign in </button>
                    </fieldset>
                </form>
            </main>
        )
    }
    ```
    > ApplicationViews 
    ```js
    import React from "react"
    import { Route, Routes, Navigate } from "react-router-dom"
    import { Login } from './auth/Login'
    import { Register } from './auth/Register'

    export const ApplicationViews = ({ isAuthenticated, setIsAuthenticated }) => {
      const PrivateRoute = ({ children }) => {
        return isAuthenticated ? children : <Navigate to="/login" />;
      }

      const setAuthUser = (user) => {
        sessionStorage.setItem("nutshell_user", JSON.stringify(user))
        setIsAuthenticated(sessionStorage.getItem("nutshell_user") !== null)
      }

      return (
        <>
          <Routes>
            <Route exact path="/login" element={<Login setAuthUser={setAuthUser} />} />
            <Route exact path="/register" element={<Register />} />
            <Route  path="/" element={
                <PrivateRoute>
                  Add component here
                </PrivateRoute>
            } />
          </Routes>

        </>
      )
    }
    ```
1. If you have not done so, follow the instructions from the README.md to set-up your api.  
    - In the `api` directory, create a copy of the `database.json.example` and remove the `.example` extension.
    - open the new database.json file and add a user with this format.
    ```json
    { "id": 1, "name": "Steve Brownlee", "email": "me@me.com" }
    ```
    - Run `json-server -p 8088 -w database.json` from the `api` directory.

1. start your react app and test that you can login.  You should see a basic nav bar and the words "Add component here".
