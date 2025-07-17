
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
// Route
path: '/users',
element: <Users></Users>,
loader: () => fetch('http://localhost:5000/users')

// Inside Users component
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

---

### ✅ CLIENT SIDE - Load Specific User by ID

```jsx
// Route
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

```jsx
<button onClick={() => handleDelete(user._id)}>X</button>

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

## 📋 Summary Table

| Operation    | HTTP Method | Client Example                        | Server Route                 |
|--------------|-------------|----------------------------------------|------------------------------|
| **Create**   | POST        | `fetch('/users', { method: 'POST' })` | `app.post('/users')`         |
| **Read All** | GET         | `fetch('/users')`                     | `app.get('/users')`          |
| **Read One** | GET         | `fetch('/users/:id')`                 | `app.get('/users/:id')`      |
| **Update**   | PUT         | `fetch('/users/:id', { method: 'PUT' })` | `app.put('/users/:id')`   |
| **Delete**   | DELETE      | `fetch('/users/:id', { method: 'DELETE' })` | `app.delete('/users/:id')` |

---
# MongoDB CRUD Methods Summary

| CRUD Type | MongoDB Method | Purpose/Usage                            | Example Code Snippet                                 | Description |
|-----------|----------------|------------------------------------------|------------------------------------------------------|-------------|
| Create    | `insertOne()`  | Insert a single document into collection | `userCollection.insertOne(user)`                     | Adds one new user to the database |
| Read      | `find()`       | Retrieve all documents (cursor)          | `userCollection.find()`                              | Returns a cursor to iterate all users |
| Read      | `toArray()`    | Convert cursor to array                  | `cursor.toArray()`                                   | Used after `find()` to get actual data |
| Read      | `findOne()`    | Retrieve one document by query           | `userCollection.findOne({ _id: new ObjectId(id) })`  | Fetches a specific user by ID |
| Update    | `updateOne()`  | Update a single document                 | `userCollection.updateOne(filter, update, options)`  | Modifies fields in one document |
| Update    | `filter`       | Used to match the document               | `const filter = { _id: new ObjectId(id) }`           | Selects which document to update |
| Update    | `options`      | Set options like `upsert: true`          | `const options = { upsert: true }`                   | Insert if no match found |
| Update    | `$set`         | Define fields to update                  | `$set: { name: user.name, email: user.email }`       | Only updates specified fields |
| Delete    | `deleteOne()`  | Delete a single document                 | `userCollection.deleteOne({ _id: new ObjectId(id) })`| Removes one document by ID |

