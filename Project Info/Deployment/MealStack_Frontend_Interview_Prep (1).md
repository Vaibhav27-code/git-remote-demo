# MealStack Frontend - React Interview Preparation Guide

## 📋 TABLE OF CONTENTS
1. [Frontend Overview](#frontend-overview)
2. [React Fundamentals](#react-fundamentals)
3. [State Management with Redux](#state-management-with-redux)
4. [Component Structure](#component-structure)
5. [API Integration](#api-integration)
6. [Routing & Navigation](#routing--navigation)
7. [Common Interview Questions](#common-interview-questions)

---

## 🎯 FRONTEND OVERVIEW

### Technology Stack
```
React 18.x          → UI Library (Component-based)
Redux Toolkit       → State Management
React Router v6     → Client-side Routing
Material-UI (MUI)   → UI Component Library
Axios               → HTTP Client
```

### Project Structure
```
frontend/
├── public/
│   └── index.html
├── src/
│   ├── components/       # Reusable UI components
│   │   ├── Navbar.jsx
│   │   ├── MenuCard.jsx
│   │   ├── OrderCard.jsx
│   │   └── ...
│   ├── pages/           # Page components
│   │   ├── HomePage.jsx
│   │   ├── StudentDashboard.jsx
│   │   ├── AdminDashboard.jsx
│   │   ├── LoginPage.jsx
│   │   └── ...
│   ├── redux/           # State management
│   │   ├── store.js
│   │   ├── slices/
│   │   │   ├── authSlice.js
│   │   │   ├── menuSlice.js
│   │   │   ├── orderSlice.js
│   │   │   └── cartSlice.js
│   │   └── ...
│   ├── services/        # API service layer
│   │   ├── api.js
│   │   ├── authService.js
│   │   ├── orderService.js
│   │   └── ...
│   ├── utils/           # Utility functions
│   │   ├── constants.js
│   │   ├── validators.js
│   │   └── ...
│   ├── App.jsx          # Root component
│   └── index.jsx        # Entry point
├── package.json
└── .env                 # Environment variables
```

---

## ⚛️ REACT FUNDAMENTALS

### 1. What is React?

**Interviewer might ask:** "What is React and why did you use it?"

**Answer:** 
"React is a JavaScript library for building user interfaces, developed by Facebook. I chose React for my project because:

1. **Component-Based Architecture:** I could break the UI into reusable components like MenuCard, OrderCard, etc.

2. **Virtual DOM:** React uses Virtual DOM for efficient updates. When data changes, React updates only the changed parts, not the entire page.

3. **Declarative:** I describe what the UI should look like for each state, and React handles the DOM updates.

4. **Large Ecosystem:** Rich ecosystem with libraries like Redux, React Router, Material-UI.

5. **Performance:** Virtual DOM and reconciliation make it fast even with complex UIs."

---

### 2. Components

**Two Types:**

#### A. Functional Components (Modern Approach)
```jsx
// components/MenuCard.jsx
import React from 'react';
import { Card, CardContent, Typography, Button } from '@mui/material';

function MenuCard({ item, onAddToCart }) {
  // This is a functional component
  // Uses hooks for state and effects
  
  const handleAddClick = () => {
    onAddToCart(item);
  };
  
  return (
    <Card>
      <CardContent>
        <Typography variant="h5">{item.itemName}</Typography>
        <Typography>Price: ₹{item.price}</Typography>
        <Typography>Available: {item.availableQuantity}</Typography>
        <Button 
          onClick={handleAddClick}
          disabled={item.availableQuantity === 0}
        >
          Add to Cart
        </Button>
      </CardContent>
    </Card>
  );
}

export default MenuCard;
```

**Key Points:**
- Function that returns JSX
- Receives data via **props**
- Can use **hooks** (useState, useEffect, etc.)
- Simpler and more concise than class components

#### B. Class Components (Legacy, but important to know)
```jsx
import React, { Component } from 'react';

class MenuCard extends Component {
  constructor(props) {
    super(props);
    this.state = {
      quantity: 1
    };
  }
  
  handleAddClick = () => {
    this.props.onAddToCart(this.props.item);
  };
  
  render() {
    return (
      <div>
        <h3>{this.props.item.itemName}</h3>
        <button onClick={this.handleAddClick}>Add to Cart</button>
      </div>
    );
  }
}

export default MenuCard;
```

**Functional vs Class:**
```
Functional Components          Class Components
- Function syntax              - Class syntax
- Use hooks for state          - this.state
- Simpler, less code           - More boilerplate
- Modern approach              - Legacy
- Recommended by React         - Still valid
```

---

### 3. JSX (JavaScript XML)

**What is JSX?**
```jsx
// JSX (looks like HTML)
const element = <h1>Hello, {name}!</h1>;

// Compiles to JavaScript
const element = React.createElement(
  'h1',
  null,
  'Hello, ',
  name,
  '!'
);
```

**JSX Rules:**
```jsx
// 1. Must return single parent element
// WRONG:
return (
  <h1>Title</h1>
  <p>Paragraph</p>
);

// RIGHT:
return (
  <div>
    <h1>Title</h1>
    <p>Paragraph</p>
  </div>
);

// OR use Fragment:
return (
  <>
    <h1>Title</h1>
    <p>Paragraph</p>
  </>
);

// 2. JavaScript expressions in {}
const price = 100;
return <p>Price: ₹{price * 1.18}</p>;  // Output: Price: ₹118

// 3. className instead of class
return <div className="container">Content</div>;

// 4. camelCase for attributes
return <button onClick={handleClick}>Click</button>;

// 5. Self-closing tags
return <img src="logo.png" alt="Logo" />;
```

---

### 4. Props (Properties)

**Passing data from parent to child:**

```jsx
// Parent Component
function StudentDashboard() {
  const student = {
    name: "John Doe",
    balance: 500
  };
  
  return <WalletCard student={student} />;
}

// Child Component
function WalletCard({ student }) {
  // Destructure props
  return (
    <div>
      <h3>{student.name}</h3>
      <p>Balance: ₹{student.balance}</p>
    </div>
  );
}

// OR without destructuring
function WalletCard(props) {
  return (
    <div>
      <h3>{props.student.name}</h3>
      <p>Balance: ₹{props.student.balance}</p>
    </div>
  );
}
```

**Props are:**
- Read-only (cannot be modified by child)
- Passed from parent to child
- Used for component configuration

**Interview Question:** 
**Q:** "Can you modify props in a component?"
**A:** "No, props are read-only. This is React's unidirectional data flow. If a child needs to update data, it should call a function passed via props by the parent."

---

### 5. State (Component Data)

**useState Hook:**

```jsx
import React, { useState } from 'react';

function OrderForm() {
  // State declaration
  const [quantity, setQuantity] = useState(1);
  const [totalPrice, setTotalPrice] = useState(0);
  
  // State is private to component
  // Can only be updated using setter function
  
  const handleIncrease = () => {
    setQuantity(quantity + 1);  // Update state
  };
  
  const handleDecrease = () => {
    if (quantity > 1) {
      setQuantity(quantity - 1);
    }
  };
  
  // Calculate price when quantity changes
  useEffect(() => {
    setTotalPrice(quantity * 50);
  }, [quantity]);  // Dependency array: run when quantity changes
  
  return (
    <div>
      <button onClick={handleDecrease}>-</button>
      <span>{quantity}</span>
      <button onClick={handleIncrease}>+</button>
      <p>Total: ₹{totalPrice}</p>
    </div>
  );
}
```

**State vs Props:**
```
State                          Props
- Owned by component           - Passed from parent
- Mutable (can change)         - Read-only
- Private                      - Public
- Triggers re-render           - Component re-renders when props change
```

---

### 6. Hooks

**Common Hooks:**

#### A. useState
```jsx
// Single value
const [count, setCount] = useState(0);

// Object
const [form, setForm] = useState({
  email: '',
  password: ''
});

// Array
const [items, setItems] = useState([]);
```

#### B. useEffect
```jsx
import { useEffect } from 'react';

function StudentProfile({ studentId }) {
  const [student, setStudent] = useState(null);
  
  // Runs after every render
  useEffect(() => {
    console.log('Component rendered');
  });
  
  // Runs once on mount (empty dependency array)
  useEffect(() => {
    console.log('Component mounted');
  }, []);
  
  // Runs when studentId changes
  useEffect(() => {
    fetchStudentData(studentId)
      .then(data => setStudent(data));
  }, [studentId]);
  
  // Cleanup function
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Tick');
    }, 1000);
    
    // Cleanup: runs before component unmounts
    return () => {
      clearInterval(timer);
    };
  }, []);
  
  return <div>{student?.name}</div>;
}
```

**useEffect Dependency Array:**
```jsx
useEffect(() => {...});              // Runs after every render
useEffect(() => {...}, []);          // Runs once on mount
useEffect(() => {...}, [value]);     // Runs when value changes
useEffect(() => {...}, [a, b]);      // Runs when a OR b changes
```

#### C. useContext (Context API)
```jsx
// Create context
const AuthContext = createContext();

// Provider (in parent)
function App() {
  const [user, setUser] = useState(null);
  
  return (
    <AuthContext.Provider value={{ user, setUser }}>
      <Dashboard />
    </AuthContext.Provider>
  );
}

// Consumer (in any child)
function Dashboard() {
  const { user, setUser } = useContext(AuthContext);
  
  return <h1>Welcome, {user?.name}</h1>;
}
```

#### D. Custom Hooks
```jsx
// Custom hook for API calls
function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [url]);
  
  return { data, loading, error };
}

// Usage
function StudentList() {
  const { data, loading, error } = useApi('/api/students');
  
  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;
  
  return (
    <ul>
      {data.map(student => <li key={student.id}>{student.name}</li>)}
    </ul>
  );
}
```

---

### 7. Event Handling

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  // Method 1: Inline arrow function
  const handleEmailChange = (e) => {
    setEmail(e.target.value);
  };
  
  // Method 2: Generic handler
  const handleChange = (e) => {
    const { name, value } = e.target;
    if (name === 'email') setEmail(value);
    if (name === 'password') setPassword(value);
  };
  
  // Form submit
  const handleSubmit = (e) => {
    e.preventDefault();  // Prevent page reload
    console.log({ email, password });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        name="email"
        value={email}
        onChange={handleEmailChange}
      />
      <input
        type="password"
        name="password"
        value={password}
        onChange={handleChange}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

**Common Events:**
```jsx
onClick       → Button click
onChange      → Input change
onSubmit      → Form submit
onFocus       → Input focused
onBlur        → Input loses focus
onMouseEnter  → Mouse enters element
onMouseLeave  → Mouse leaves element
onKeyDown     → Key pressed
```

---

### 8. Conditional Rendering

```jsx
function OrderStatus({ status }) {
  // Method 1: if-else
  if (status === 'PENDING') {
    return <p style={{ color: 'orange' }}>Order Pending</p>;
  } else if (status === 'SERVED') {
    return <p style={{ color: 'green' }}>Order Served</p>;
  }
  
  // Method 2: Ternary operator
  return (
    <p style={{ color: status === 'PENDING' ? 'orange' : 'green' }}>
      {status === 'PENDING' ? 'Order Pending' : 'Order Served'}
    </p>
  );
  
  // Method 3: Logical AND (&&)
  return (
    <div>
      {status === 'PENDING' && <p>Please wait...</p>}
      {status === 'SERVED' && <p>Ready for pickup!</p>}
    </div>
  );
}
```

---

### 9. Lists & Keys

```jsx
function MenuList({ items }) {
  return (
    <div>
      {items.map(item => (
        <MenuCard 
          key={item.id}  // IMPORTANT: Unique key for each item
          item={item}
        />
      ))}
    </div>
  );
}
```

**Why Keys?**
- Help React identify which items changed/added/removed
- Improve performance during re-renders
- Should be unique and stable (don't use array index if list can reorder)

**Interview Question:**
**Q:** "Why should we use keys in lists?"
**A:** "Keys help React identify which items in a list have changed, been added, or removed. This optimizes re-rendering by updating only changed items instead of re-rendering the entire list. Keys should be unique and stable - using array indices is okay only if the list never reorders."

---

## 🔄 STATE MANAGEMENT WITH REDUX

### Why Redux?

**Interviewer:** "Why did you use Redux?"

**Answer:** 
"I used Redux for centralized state management because:

1. **Shared State:** Components like Navbar, Dashboard, and Cart all need user/cart data. Redux provides a single source of truth.

2. **Predictable:** State changes only through actions and reducers, making debugging easier.

3. **DevTools:** Redux DevTools let me see every state change and time-travel debug.

4. **Avoid Prop Drilling:** Without Redux, I'd need to pass props through multiple levels. Redux lets any component access state directly.

Example: Cart data is needed in MenuPage (to show cart count) and CartPage (to display items). Instead of lifting state up to App and prop drilling, I store it in Redux."

---

### Redux Flow

```
┌──────────────┐
│  Component   │
│   (View)     │
└──────┬───────┘
       │
       │ 1. Dispatch Action
       ▼
┌──────────────┐
│    Action    │
│ { type, payload }
└──────┬───────┘
       │
       │ 2. Action goes to Reducer
       ▼
┌──────────────┐
│   Reducer    │
│ Pure Function│
│ Old State +  │
│ Action →     │
│ New State    │
└──────┬───────┘
       │
       │ 3. Update Store
       ▼
┌──────────────┐
│    Store     │
│ (Global State)
└──────┬───────┘
       │
       │ 4. Component subscribes to state
       ▼
┌──────────────┐
│  Component   │
│   Re-render  │
└──────────────┘
```

---

### Redux Toolkit Setup

#### 1. Store Configuration

```javascript
// redux/store.js
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './slices/authSlice';
import menuReducer from './slices/menuSlice';
import cartReducer from './slices/cartSlice';
import orderReducer from './slices/orderSlice';

const store = configureStore({
  reducer: {
    auth: authReducer,    // state.auth
    menu: menuReducer,    // state.menu
    cart: cartReducer,    // state.cart
    order: orderReducer   // state.order
  }
});

export default store;
```

#### 2. Slice (Reducer + Actions)

```javascript
// redux/slices/authSlice.js
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  user: null,
  token: localStorage.getItem('token') || null,
  isAuthenticated: false,
  loading: false,
  error: null
};

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    // Reducer functions
    loginStart: (state) => {
      state.loading = true;
      state.error = null;
    },
    
    loginSuccess: (state, action) => {
      // action.payload = { user, token }
      state.user = action.payload.user;
      state.token = action.payload.token;
      state.isAuthenticated = true;
      state.loading = false;
      
      // Save to localStorage
      localStorage.setItem('token', action.payload.token);
    },
    
    loginFailure: (state, action) => {
      state.loading = false;
      state.error = action.payload;
    },
    
    logout: (state) => {
      state.user = null;
      state.token = null;
      state.isAuthenticated = false;
      
      // Remove from localStorage
      localStorage.removeItem('token');
    },
    
    updateProfile: (state, action) => {
      state.user = { ...state.user, ...action.payload };
    }
  }
});

// Export actions
export const { 
  loginStart, 
  loginSuccess, 
  loginFailure, 
  logout,
  updateProfile 
} = authSlice.actions;

// Export reducer
export default authSlice.reducer;
```

**Key Concepts:**

1. **Slice:** Combines reducer logic and actions for a specific feature

2. **Immutable Updates:** Redux Toolkit uses Immer library, allowing "mutating" syntax:
   ```javascript
   // Old way (Redux)
   return { ...state, user: action.payload };
   
   // New way (Redux Toolkit with Immer)
   state.user = action.payload;  // Looks like mutation, but isn't!
   ```

3. **Action Creators:** Automatically generated from reducer names
   ```javascript
   loginSuccess({ user, token });
   // Creates: { type: 'auth/loginSuccess', payload: { user, token } }
   ```

---

#### 3. Cart Slice (Complex Example)

```javascript
// redux/slices/cartSlice.js
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  items: [],  // [{ itemDaily, quantity }]
  totalItems: 0,
  totalPrice: 0
};

const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addToCart: (state, action) => {
      // action.payload = itemDaily object
      const existingItem = state.items.find(
        item => item.itemDaily.id === action.payload.id
      );
      
      if (existingItem) {
        // Item already in cart, increase quantity
        existingItem.quantity += 1;
      } else {
        // New item, add to cart
        state.items.push({
          itemDaily: action.payload,
          quantity: 1
        });
      }
      
      // Recalculate totals
      state.totalItems = state.items.reduce((sum, item) => sum + item.quantity, 0);
      state.totalPrice = state.items.reduce(
        (sum, item) => sum + (item.itemDaily.itemMaster.price * item.quantity), 
        0
      );
    },
    
    removeFromCart: (state, action) => {
      // action.payload = itemDailyId
      state.items = state.items.filter(
        item => item.itemDaily.id !== action.payload
      );
      
      // Recalculate totals
      state.totalItems = state.items.reduce((sum, item) => sum + item.quantity, 0);
      state.totalPrice = state.items.reduce(
        (sum, item) => sum + (item.itemDaily.itemMaster.price * item.quantity), 
        0
      );
    },
    
    updateQuantity: (state, action) => {
      // action.payload = { itemDailyId, quantity }
      const item = state.items.find(
        item => item.itemDaily.id === action.payload.itemDailyId
      );
      
      if (item) {
        item.quantity = action.payload.quantity;
        
        // Recalculate totals
        state.totalItems = state.items.reduce((sum, item) => sum + item.quantity, 0);
        state.totalPrice = state.items.reduce(
          (sum, item) => sum + (item.itemDaily.itemMaster.price * item.quantity), 
          0
        );
      }
    },
    
    clearCart: (state) => {
      state.items = [];
      state.totalItems = 0;
      state.totalPrice = 0;
    }
  }
});

export const { addToCart, removeFromCart, updateQuantity, clearCart } = cartSlice.actions;
export default cartSlice.reducer;
```

---

#### 4. Async Operations with Thunks

```javascript
// redux/slices/menuSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

// Async thunk for fetching menu
export const fetchDailyMenu = createAsyncThunk(
  'menu/fetchDaily',
  async (date, { rejectWithValue }) => {
    try {
      const response = await axios.get(`/api/item-daily?date=${date}`);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response.data);
    }
  }
);

const menuSlice = createSlice({
  name: 'menu',
  initialState: {
    items: [],
    loading: false,
    error: null
  },
  reducers: {
    // Synchronous reducers
  },
  extraReducers: (builder) => {
    builder
      // Pending state
      .addCase(fetchDailyMenu.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      // Success state
      .addCase(fetchDailyMenu.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      // Error state
      .addCase(fetchDailyMenu.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

export default menuSlice.reducer;
```

---

#### 5. Using Redux in Components

```jsx
// pages/MenuPage.jsx
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { fetchDailyMenu } from '../redux/slices/menuSlice';
import { addToCart } from '../redux/slices/cartSlice';

function MenuPage() {
  const dispatch = useDispatch();
  
  // Read from Redux state
  const { items, loading, error } = useSelector(state => state.menu);
  const { totalItems } = useSelector(state => state.cart);
  const { user } = useSelector(state => state.auth);
  
  // Fetch menu on component mount
  useEffect(() => {
    const today = new Date().toISOString().split('T')[0];
    dispatch(fetchDailyMenu(today));
  }, [dispatch]);
  
  // Handle add to cart
  const handleAddToCart = (item) => {
    dispatch(addToCart(item));
  };
  
  if (loading) return <p>Loading menu...</p>;
  if (error) return <p>Error: {error}</p>;
  
  return (
    <div>
      <h1>Today's Menu</h1>
      <p>Cart Items: {totalItems}</p>
      
      {items.map(item => (
        <div key={item.id}>
          <h3>{item.itemMaster.itemName}</h3>
          <p>Price: ₹{item.itemMaster.price}</p>
          <p>Available: {item.availableQuantity}</p>
          <button 
            onClick={() => handleAddToCart(item)}
            disabled={item.availableQuantity === 0}
          >
            Add to Cart
          </button>
        </div>
      ))}
    </div>
  );
}

export default MenuPage;
```

**Key Hooks:**
- `useSelector`: Read data from Redux state
- `useDispatch`: Dispatch actions to update state

---

## 🌐 API INTEGRATION

### Axios Setup

```javascript
// services/api.js
import axios from 'axios';

// Create axios instance
const api = axios.create({
  baseURL: 'http://localhost:8080/api',
  headers: {
    'Content-Type': 'application/json'
  }
});

// Request interceptor: Add JWT token to every request
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor: Handle errors globally
api.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    if (error.response?.status === 401) {
      // Unauthorized: Clear token and redirect to login
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

---

### Service Layer

```javascript
// services/authService.js
import api from './api';

export const authService = {
  // Register student
  register: async (data) => {
    const response = await api.post('/students/register', data);
    return response.data;
  },
  
  // Login
  login: async (email, password) => {
    const response = await api.post('/students/login', { email, password });
    return response.data;
  },
  
  // Get profile
  getProfile: async (studentId) => {
    const response = await api.get(`/students/profile/${studentId}`);
    return response.data;
  },
  
  // Update profile
  updateProfile: async (studentId, data) => {
    const response = await api.put(`/students/profile/${studentId}`, data);
    return response.data;
  },
  
  // Logout
  logout: () => {
    localStorage.removeItem('token');
  }
};
```

```javascript
// services/orderService.js
import api from './api';

export const orderService = {
  // Place order
  placeOrder: async (orderData) => {
    const response = await api.post('/orders/place', orderData);
    return response.data;
  },
  
  // Get student orders
  getStudentOrders: async (studentId) => {
    const response = await api.get(`/orders/student/${studentId}`);
    return response.data;
  },
  
  // Get order history
  getOrderHistory: async (studentId, status) => {
    const url = status 
      ? `/orders/history/${studentId}?status=${status}`
      : `/orders/history/${studentId}`;
    const response = await api.get(url);
    return response.data;
  }
};
```

---

### API Integration in Components

```jsx
// pages/LoginPage.jsx
import React, { useState } from 'react';
import { useDispatch } from 'react-redux';
import { useNavigate } from 'react-router-dom';
import { loginStart, loginSuccess, loginFailure } from '../redux/slices/authSlice';
import { authService } from '../services/authService';

function LoginPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const dispatch = useDispatch();
  const navigate = useNavigate();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Start loading
    dispatch(loginStart());
    
    try {
      // Call API
      const response = await authService.login(email, password);
      
      // Success: Update Redux state
      dispatch(loginSuccess({
        user: response.user,
        token: response.token
      }));
      
      // Navigate to dashboard
      navigate('/dashboard');
      
    } catch (error) {
      // Error: Update Redux state
      dispatch(loginFailure(error.response?.data?.message || 'Login failed'));
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
        required
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
        required
      />
      <button type="submit">Login</button>
    </form>
  );
}

export default LoginPage;
```

---

## 🚦 ROUTING & NAVIGATION

### React Router Setup

```jsx
// App.jsx
import React from 'react';
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { useSelector } from 'react-redux';

// Pages
import HomePage from './pages/HomePage';
import LoginPage from './pages/LoginPage';
import RegisterPage from './pages/RegisterPage';
import StudentDashboard from './pages/StudentDashboard';
import AdminDashboard from './pages/AdminDashboard';
import MenuPage from './pages/MenuPage';
import CartPage from './pages/CartPage';
import OrdersPage from './pages/OrdersPage';

// Protected Route Component
function ProtectedRoute({ children, requiredRole }) {
  const { isAuthenticated, user } = useSelector(state => state.auth);
  
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  if (requiredRole && user.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }
  
  return children;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Public routes */}
        <Route path="/" element={<HomePage />} />
        <Route path="/login" element={<LoginPage />} />
        <Route path="/register" element={<RegisterPage />} />
        
        {/* Student routes */}
        <Route 
          path="/dashboard" 
          element={
            <ProtectedRoute requiredRole="STUDENT">
              <StudentDashboard />
            </ProtectedRoute>
          } 
        />
        <Route 
          path="/menu" 
          element={
            <ProtectedRoute requiredRole="STUDENT">
              <MenuPage />
            </ProtectedRoute>
          } 
        />
        <Route 
          path="/cart" 
          element={
            <ProtectedRoute requiredRole="STUDENT">
              <CartPage />
            </ProtectedRoute>
          } 
        />
        <Route 
          path="/orders" 
          element={
            <ProtectedRoute requiredRole="STUDENT">
              <OrdersPage />
            </ProtectedRoute>
          } 
        />
        
        {/* Admin routes */}
        <Route 
          path="/admin" 
          element={
            <ProtectedRoute requiredRole="ADMIN">
              <AdminDashboard />
            </ProtectedRoute>
          } 
        />
        
        {/* 404 route */}
        <Route path="*" element={<h1>404 - Page Not Found</h1>} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

**Navigation:**
```jsx
import { useNavigate, Link } from 'react-router-dom';

function Navbar() {
  const navigate = useNavigate();
  
  const handleLogout = () => {
    // Clear auth state
    // ...
    navigate('/login');
  };
  
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/menu">Menu</Link>
      <Link to="/orders">Orders</Link>
      <button onClick={handleLogout}>Logout</button>
    </nav>
  );
}
```

**Key Components:**
- `BrowserRouter`: Enables routing
- `Routes`: Container for all routes
- `Route`: Define path and component
- `Navigate`: Programmatic redirect
- `Link`: Navigation link (like <a> but for SPA)
- `useNavigate`: Hook for programmatic navigation

---

## 🎨 MATERIAL-UI INTEGRATION

```jsx
import { 
  Card, 
  CardContent, 
  Typography, 
  Button,
  Grid,
  TextField,
  Dialog
} from '@mui/material';

function MenuCard({ item, onAddToCart }) {
  return (
    <Card sx={{ maxWidth: 345, margin: 2 }}>
      <CardContent>
        <Typography variant="h5" component="div">
          {item.itemMaster.itemName}
        </Typography>
        
        <Typography variant="body2" color="text.secondary">
          {item.itemMaster.category}
        </Typography>
        
        <Typography variant="h6" color="primary">
          ₹{item.itemMaster.price}
        </Typography>
        
        <Typography 
          variant="body2" 
          color={item.availableQuantity > 0 ? 'success.main' : 'error.main'}
        >
          {item.availableQuantity > 0 
            ? `Available: ${item.availableQuantity}` 
            : 'Out of Stock'
          }
        </Typography>
        
        <Button 
          variant="contained" 
          color="primary"
          onClick={() => onAddToCart(item)}
          disabled={item.availableQuantity === 0}
          sx={{ marginTop: 2 }}
        >
          Add to Cart
        </Button>
      </CardContent>
    </Card>
  );
}
```

---

## 🎤 COMMON INTERVIEW QUESTIONS

### Q1: What is Virtual DOM and how does React use it?

**Answer:**
"The Virtual DOM is a lightweight JavaScript representation of the actual DOM. React uses it for efficient updates:

1. **Initial Render:** React creates a Virtual DOM tree

2. **State Change:** When state changes, React creates a new Virtual DOM tree

3. **Diffing:** React compares (diffs) the new Virtual DOM with the previous one

4. **Reconciliation:** React calculates the minimum changes needed

5. **Update:** React updates only the changed parts in the actual DOM

**Example:**
```jsx
// State changes: count goes from 0 to 1
<div>
  <h1>Count: {count}</h1>
  <p>Other content</p>
</div>

// React only updates the text node "0" → "1"
// Doesn't re-create the entire <div>
```

This is much faster than updating the entire DOM on every change."

---

### Q2: Explain component lifecycle in React.

**Answer:**
"In functional components with hooks, the lifecycle is:

**1. Mount (Component Created):**
```jsx
useEffect(() => {
  console.log('Component mounted');
  // Fetch data, setup subscriptions
}, []);  // Empty array: runs once
```

**2. Update (State/Props Change):**
```jsx
useEffect(() => {
  console.log('Component updated');
  // Runs after every render
});

useEffect(() => {
  console.log('studentId changed');
  // Runs when studentId changes
}, [studentId]);
```

**3. Unmount (Component Removed):**
```jsx
useEffect(() => {
  const timer = setInterval(...);
  
  return () => {
    clearInterval(timer);  // Cleanup
    console.log('Component unmounted');
  };
}, []);
```

In my project, I use this for:
- Fetching menu data on mount
- Clearing timers on unmount
- Refetching data when filters change"

---

### Q3: What is prop drilling and how do you solve it?

**Answer:**
"Prop drilling is passing props through multiple levels of components that don't need the data.

**Problem:**
```jsx
<App user={user}>
  <Dashboard user={user}>
    <Sidebar user={user}>
      <UserProfile user={user} />  // Only this needs user!
    </Sidebar>
  </Dashboard>
</App>
```

**Solutions:**

1. **Context API:**
```jsx
const UserContext = createContext();

function App() {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={user}>
      <Dashboard />  // No props!
    </UserContext.Provider>
  );
}

function UserProfile() {
  const user = useContext(UserContext);  // Direct access!
  return <div>{user.name}</div>;
}
```

2. **Redux:** (What I used)
   - Store user in Redux state
   - Any component can access via useSelector

In my project, I used Redux to avoid prop drilling for user, cart, and menu data."

---

### Q4: Controlled vs Uncontrolled components?

**Answer:**
"**Controlled Components:** React controls the input value via state

```jsx
function ControlledForm() {
  const [email, setEmail] = useState('');
  
  return (
    <input
      value={email}  // React controls this
      onChange={(e) => setEmail(e.target.value)}
    />
  );
}
```

**Uncontrolled Components:** DOM controls the input value

```jsx
function UncontrolledForm() {
  const emailRef = useRef();
  
  const handleSubmit = () => {
    console.log(emailRef.current.value);  // Read from DOM
  };
  
  return <input ref={emailRef} />;  // DOM controls this
}
```

**I used controlled components** because:
- Easier validation
- Better React integration
- Single source of truth (state)
- Can easily clear/modify values"

---

### Q5: What are React keys and why are they important?

**Answer:**
"Keys help React identify which items in a list have changed, been added, or removed.

**Bad:**
```jsx
{items.map((item, index) => (
  <div key={index}>{item.name}</div>  // DON'T use index!
))}
```

**Good:**
```jsx
{items.map(item => (
  <div key={item.id}>{item.name}</div>  // Unique, stable ID
))}
```

**Why not index?**
If list reorders:
```
Initial: [A, B, C] → keys: [0, 1, 2]
Reorder: [C, A, B] → keys: [0, 1, 2] (same keys, different items!)
```
React thinks nothing changed, causes bugs.

**In my project:** I used item.id as key for menu items, orders list."

---

### Q6: How does Redux differ from Context API?

**Answer:**
"Both manage global state, but have different use cases:

**Context API:**
- Built into React
- Good for simple state (theme, auth)
- Less boilerplate
- Re-renders all consumers on any state change

**Redux:**
- External library
- Better for complex state
- More boilerplate
- Fine-grained subscriptions (components only re-render when their slice changes)
- DevTools, middleware
- Better performance for large apps

**I chose Redux because:**
1. Complex state (auth, cart, menu, orders)
2. Many components need different parts of state
3. Need Redux DevTools for debugging
4. Cart logic (add/remove/update) benefits from Redux's predictable updates"

---

This completes the frontend preparation guide! Would you like me to create the complete interview questions document next?
