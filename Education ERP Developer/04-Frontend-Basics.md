---
title: Frontend Basics
tags: [javascript, jquery, react, vue, frontend]
created: 2026-02-03
---

# Frontend Basics

## 🎯 JavaScript Essentials

### ES6+ Features

```javascript
// Destructuring
const { name, email } = student;
const [first, ...rest] = items;

// Spread operator
const newStudent = { ...student, class: 'Class 10' };
const allItems = [...items1, ...items2];

// Arrow functions
const getTotal = (items) => items.reduce((sum, i) => sum + i.amount, 0);

// Template literals
const message = `Welcome ${student.name}, your fee is ₹${fee.amount}`;

// Optional chaining & nullish coalescing
const guardianName = student?.guardian?.name ?? 'N/A';

// Async/await
async function fetchStudents() {
  try {
    const response = await fetch('/api/students');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Failed to fetch:', error);
  }
}

// Promise.all for parallel requests
const [students, classes, fees] = await Promise.all([
  fetch('/api/students').then(r => r.json()),
  fetch('/api/classes').then(r => r.json()),
  fetch('/api/fees').then(r => r.json())
]);
```

### DOM Manipulation

```javascript
// Query selectors
const form = document.querySelector('#studentForm');
const rows = document.querySelectorAll('.student-row');

// Event handling
form.addEventListener('submit', async (e) => {
  e.preventDefault();
  const formData = new FormData(form);
  await submitStudent(Object.fromEntries(formData));
});

// Event delegation
document.querySelector('#studentTable').addEventListener('click', (e) => {
  if (e.target.classList.contains('delete-btn')) {
    const studentId = e.target.dataset.id;
    deleteStudent(studentId);
  }
});

// Dynamic content
function renderStudents(students) {
  const tbody = document.querySelector('#studentTable tbody');
  tbody.innerHTML = students.map(s => `
    <tr>
      <td>${s.name}</td>
      <td>${s.class}</td>
      <td>
        <button class="delete-btn" data-id="${s.id}">Delete</button>
      </td>
    </tr>
  `).join('');
}
```

## 📦 jQuery (Legacy ERP Systems)

```javascript
// Document ready
$(document).ready(function() {
  loadStudents();
});

// AJAX requests
function loadStudents() {
  $.ajax({
    url: '/api/students',
    method: 'GET',
    success: function(data) {
      renderTable(data);
    },
    error: function(xhr) {
      alert('Error: ' + xhr.responseJSON.message);
    }
  });
}

// Form submission
$('#studentForm').on('submit', function(e) {
  e.preventDefault();
  const formData = $(this).serialize();
  
  $.post('/api/students', formData)
    .done(function(response) {
      showNotification('Student added successfully');
      loadStudents();
    })
    .fail(function(xhr) {
      showErrors(xhr.responseJSON.errors);
    });
});

// Event delegation
$('#studentTable').on('click', '.delete-btn', function() {
  const id = $(this).data('id');
  if (confirm('Delete this student?')) {
    $.ajax({
      url: `/api/students/${id}`,
      method: 'DELETE',
      success: loadStudents
    });
  }
});

// DOM manipulation
function renderTable(students) {
  const rows = students.map(s => `
    <tr>
      <td>${s.name}</td>
      <td>${s.email}</td>
      <td>
        <button class="btn edit-btn" data-id="${s.id}">Edit</button>
        <button class="btn delete-btn" data-id="${s.id}">Delete</button>
      </td>
    </tr>
  `).join('');
  
  $('#studentTable tbody').html(rows);
}
```

## ⚛️ React Basics

### Component Structure

```jsx
// Functional component with hooks
import { useState, useEffect } from 'react';

function StudentList() {
  const [students, setStudents] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchStudents();
  }, []);

  async function fetchStudents() {
    try {
      const response = await fetch('/api/students');
      const data = await response.json();
      setStudents(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Email</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        {students.map(student => (
          <StudentRow key={student.id} student={student} onDelete={fetchStudents} />
        ))}
      </tbody>
    </table>
  );
}

function StudentRow({ student, onDelete }) {
  async function handleDelete() {
    if (window.confirm('Delete this student?')) {
      await fetch(`/api/students/${student.id}`, { method: 'DELETE' });
      onDelete();
    }
  }

  return (
    <tr>
      <td>{student.name}</td>
      <td>{student.email}</td>
      <td>
        <button onClick={handleDelete}>Delete</button>
      </td>
    </tr>
  );
}
```

### Form Handling

```jsx
function StudentForm({ onSubmit }) {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    classId: ''
  });
  const [errors, setErrors] = useState({});

  function handleChange(e) {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  }

  async function handleSubmit(e) {
    e.preventDefault();
    try {
      const response = await fetch('/api/students', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      if (!response.ok) {
        const data = await response.json();
        setErrors(data.errors);
        return;
      }
      
      onSubmit();
      setFormData({ name: '', email: '', classId: '' });
    } catch (err) {
      setErrors({ general: err.message });
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="name"
          value={formData.name}
          onChange={handleChange}
          placeholder="Name"
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>
      <div>
        <input
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      <button type="submit">Add Student</button>
    </form>
  );
}
```

## 💚 Vue.js Basics

### Component Structure

```vue
<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-else-if="error">Error: {{ error }}</div>
    <table v-else>
      <thead>
        <tr>
          <th>Name</th>
          <th>Email</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="student in students" :key="student.id">
          <td>{{ student.name }}</td>
          <td>{{ student.email }}</td>
          <td>
            <button @click="deleteStudent(student.id)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      students: [],
      loading: true,
      error: null
    };
  },
  
  async mounted() {
    await this.fetchStudents();
  },
  
  methods: {
    async fetchStudents() {
      try {
        const response = await fetch('/api/students');
        this.students = await response.json();
      } catch (err) {
        this.error = err.message;
      } finally {
        this.loading = false;
      }
    },
    
    async deleteStudent(id) {
      if (confirm('Delete this student?')) {
        await fetch(`/api/students/${id}`, { method: 'DELETE' });
        await this.fetchStudents();
      }
    }
  }
};
</script>
```

### Vue 3 Composition API

```vue
<script setup>
import { ref, onMounted } from 'vue';

const students = ref([]);
const loading = ref(true);

async function fetchStudents() {
  try {
    const response = await fetch('/api/students');
    students.value = await response.json();
  } finally {
    loading.value = false;
  }
}

onMounted(fetchStudents);
</script>

<template>
  <div v-if="loading">Loading...</div>
  <ul v-else>
    <li v-for="student in students" :key="student.id">
      {{ student.name }}
    </li>
  </ul>
</template>
```

## ❓ Interview Q&A

> [!question]- What is the difference between `==` and `===`?
> - `==` (loose equality): Compares values after type coercion
> - `===` (strict equality): Compares both value and type
> ```javascript
> '5' == 5   // true (coerced)
> '5' === 5  // false (different types)
> null == undefined  // true
> null === undefined // false
> ```
> Always use `===` to avoid unexpected behavior.

> [!question]- Explain closures in JavaScript
> A closure is a function that has access to variables from its outer scope even after the outer function has returned.
> ```javascript
> function createCounter() {
>   let count = 0; // Enclosed variable
>   return {
>     increment: () => ++count,
>     getCount: () => count
>   };
> }
> 
> const counter = createCounter();
> counter.increment(); // 1
> counter.increment(); // 2
> counter.getCount();  // 2
> ```

> [!question]- What is event bubbling and capturing?
> ```
> Capturing (top-down)     Bubbling (bottom-up)
>        │                        ▲
>        ▼                        │
>   ┌─────────┐              ┌─────────┐
>   │ document│              │ document│
>   │ ┌─────┐ │              │ ┌─────┐ │
>   │ │ div │ │              │ │ div │ │
>   │ │┌───┐│ │              │ │┌───┐│ │
>   │ ││btn││ │              │ ││btn││ │
>   │ │└───┘│ │              │ │└───┘│ │
>   │ └─────┘ │              │ └─────┘ │
>   └─────────┘              └─────────┘
> ```
> - Default is bubbling
> - Use `e.stopPropagation()` to stop
> - Use `{ capture: true }` for capturing phase

> [!question]- What are React hooks rules?
> 1. Only call hooks at the top level (not in loops/conditions)
> 2. Only call hooks from React functions
> 3. Custom hooks must start with "use"
> ```javascript
> // Wrong
> if (condition) {
>   const [state, setState] = useState(); // ❌
> }
> 
> // Correct
> const [state, setState] = useState();
> if (condition) {
>   // use state here
> }
> ```

> [!question]- Explain Virtual DOM
> ```
> Real DOM Update:          Virtual DOM Update:
> ┌─────────────┐           ┌─────────────┐
> │ Change data │           │ Change data │
> └──────┬──────┘           └──────┬──────┘
>        │                         │
>        ▼                         ▼
> ┌─────────────┐           ┌─────────────┐
> │ Re-render   │           │ New Virtual │
> │ entire DOM  │           │    DOM      │
> └─────────────┘           └──────┬──────┘
>                                  │ Diff
>                                  ▼
>                          ┌─────────────┐
>                          │ Patch only  │
>                          │  changes    │
>                          └─────────────┘
> ```
> Virtual DOM is faster because it minimizes actual DOM operations.

> [!question]- What is the difference between Vue's `v-if` and `v-show`?
> - `v-if`: Conditionally renders element (adds/removes from DOM)
> - `v-show`: Always renders, toggles CSS `display` property
> 
> Use `v-show` for frequent toggles, `v-if` for conditions that rarely change.
