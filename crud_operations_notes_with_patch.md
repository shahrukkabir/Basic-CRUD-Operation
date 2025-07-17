
# CRUD Operations Notes

This document explains the implementation of basic **CRUD (Create, Read, Update, Delete)** operations using **Express.js** on the backend and **Fetch API** on the client.

---

## 🔧 1. CREATE - POST

### ✅ CLIENT SIDE

```js
fetch('http://localhost:5000/users', {
    method: 'POST',
    headers: {
        'content-type': 'application/json'
    },
    body: JSON.stringify(user)
})
    .then(res => res.json())
    .then(data => {
        console.log(data);
        if (data.insertedId) {
            alert('Users added successfully');
            form.reset();
        }
    });
```

### ✅ SERVER SIDE

```js
app.post('/users', async (req, res) => {
    const user = req.body;
    console.log('new user', user);
    const result = await userCollection.insertOne(user);
    res.send(result);
});
```

---

## 🔍 2. READ - GET

### ✅ CLIENT SIDE - Load All Users

```jsx
path: '/users',
element: <Users></Users>,
loader: () => fetch('http://localhost:5000/users')

const loadedUsers = useLoaderData();
```

### ✅ SERVER SIDE - Get All Users

```js
app.get('/users', async (req, res) => {
    const cursor = userCollection.find();
    const result = await cursor.toArray();
    res.send(result);
});
```

### ✅ CLIENT SIDE - Load Specific User by ID

```jsx
path: '/update/:id',
element: <Update></Update>,
loader: ({ params }) => fetch(`http://localhost:5000/users/${params.id}`)
```

### ✅ SERVER SIDE - Get Specific User

```js
app.get('/users/:id', async (req, res) => {
    const id = req.params.id;
    const query = { _id: new ObjectId(id) };
    const user = await userCollection.findOne(query);
    res.send(user);
});
```

---

## 🗑️ 3. DELETE

### ✅ CLIENT SIDE

```js
const handleDelete = _id => {
    fetch(`http://localhost:5000/users/${_id}`, {
        method: 'DELETE'
    })
    .then(res => res.json())
    .then(data => {
        if (data.deletedCount > 0) {
            alert('deleted successfully');
            const remaining = users.filter(user => user._id !== _id);
            setUsers(remaining);
        }
    });
};
```

### ✅ SERVER SIDE

```js
app.delete('/users/:id', async (req, res) => {
    const id = req.params.id;
    const query = { _id: new ObjectId(id) };
    const result = await userCollection.deleteOne(query);
    res.send(result);
});
```

---

## ✏️ 4. UPDATE - PUT

### ✅ CLIENT SIDE

```js
const handleUpdate = event => {
    event.preventDefault();
    const form = event.target;
    const name = form.name.value;
    const email = form.email.value;
    const updatedUser = { name, email };

    fetch(`http://localhost:5000/users/${loadedUser._id}`, {
        method: 'PUT',
        headers: {
            'content-type': 'application/json'
        },
        body: JSON.stringify(updatedUser)
    })
    .then(res => res.json())
    .then(data => {
        if (data.modifiedCount > 0) {
            alert('user updated successfully');
        }
    });
};
```

### ✅ SERVER SIDE

```js
app.put('/users/:id', async (req, res) => {
    const id = req.params.id;
    const user = req.body;
    const filter = { _id: new ObjectId(id) };
    const options = { upsert: true };
    const updatedUser = {
        $set: {
            name: user.name,
            email: user.email
        }
    };
    const result = await userCollection.updateOne(filter, updatedUser, options);
    res.send(result);
});
```

---

## 🔁 5. PATCH - Partial Update (e.g., Last Login Time)

### ✅ CLIENT SIDE

```js
const lastSignInTime = result?.user?.metadata?.lastSignInTime;
const loginInfo = { email, lastSignInTime };

fetch(`https://coffee-store-server-delta-sage.vercel.app/users`, {
    method: 'PATCH',
    headers: {
        'content-type': 'application/json'
    },
    body: JSON.stringify(loginInfo)
})
.then(res => res.json())
.then(data => {
    console.log('sign in info updated in db', data);
});
```

### ✅ SERVER SIDE

```js
app.patch('/users', async (req, res) => {
    const email = req.body.email;
    const filter = { email };
    const updatedDoc = {
        $set: {
            lastSignInTime: req.body?.lastSignInTime
        }
    };
    const result = await userCollection.updateOne(filter, updatedDoc);
    res.send(result);
});
```

---

## 🆚 Difference Between PUT and PATCH

| Feature            | PUT                                      | PATCH                                 |
|--------------------|-------------------------------------------|----------------------------------------|
| Full or Partial    | Replaces the entire document              | Updates only specified fields          |
| Use Case           | When updating all fields of a record      | When updating one or few fields only   |
| Idempotent         | Yes                                       | Yes (but behavior may vary slightly)   |
| Example            | Update full name and email together       | Update only `lastLoginTime` field      |
| Backend Operation  | Uses `$set` with full object              | Uses `$set` with partial object        |

---

## 📋 Updated Summary Table

| CRUD Type | HTTP Method | MongoDB Method    | Client Code Example                            | Server Route                 | Description                                 |
|-----------|-------------|-------------------|------------------------------------------------|------------------------------|---------------------------------------------|
| Create    | POST        | `insertOne()`     | `fetch('/users', { method: 'POST' })`          | `app.post('/users')`         | Add a new document                          |
| Read      | GET         | `find()`, `findOne()` | `fetch('/users')`, `fetch('/users/:id')`   | `app.get('/users')`, `app.get('/users/:id')` | Fetch all or one document         |
| Update    | PUT         | `updateOne()`     | `fetch('/users/:id', { method: 'PUT' })`       | `app.put('/users/:id')`      | Replace all fields in a document            |
| Update    | PATCH       | `updateOne()`     | `fetch('/users', { method: 'PATCH' })`         | `app.patch('/users')`        | Update specific field(s) only               |
| Delete    | DELETE      | `deleteOne()`     | `fetch('/users/:id', { method: 'DELETE' })`    | `app.delete('/users/:id')`   | Remove a document                           |
