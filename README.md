# ⚛️ React Custom Global Store (No Redux, No Context)

A lightweight, high-performance state management solution built entirely with **React Custom Hooks**. This project demonstrates how to implement a **Global Store** with the **Observer Pattern**, offering a Redux-like experience with zero dependencies and minimal boilerplate.

**Why this exists?** To solve the "Prop Drilling" problem without the complexity of Redux or the unnecessary re-renders of the Context API.

## Key Features

*   **Zero External Dependencies:** Built using native React Hooks (`useState`, `useEffect`).
*   **State Slices Support:** Organize your data into modular slices (e.g., `products`, `counter`, `auth`) just like Redux.
*   **High Performance:**
    *   Components only re-render when they listen to the store.
    *   Supports `shouldListen: false` for components that only dispatch actions (no re-renders!).
*   **Simplified Dispatch:** No complex Action Creators or Types. Just `dispatch('ACTION_ID', payload)`.
*   **Logic Decoupling:** Business logic is separated from UI components.

---

## Architecture: How It Works

This solution uses a custom implementation of the **Observer Pattern**:

1.  **Global Registry:** Variables defined outside the hook (`let globalState`) act as a single source of truth across the entire app.
2.  **Listeners:** Components subscribe to changes via `useEffect`. When state updates, the store iterates through all registered listeners (`useState` modifiers) and triggers re-renders.
3.  **State Merging:** The `initStore` function allows you to merge multiple "Slices" of state into one global object dynamically.

---

## 📂 Project Structure

The project is organized to separate the store logic from the UI.

```text
src/
├── hooks-store/          # The State Management Core
│   ├── store.js          # The Global Hook Engine (Observer Pattern implementation)
│   ├── products-store.js # Slice: Manages products & favorites logic
│   └── counter-store.js  # Slice: Manages counter logic (demo purposes)
├── components/           # Reusable UI Components
│   ├── Products/         # Product-related components (ProductItem, etc.)
│   ├── Favorites/        # Favorites-related components
│   └── Nav/              # Navigation bar
├── containers/           # Pages / Smart Components
│   ├── Products.js       # "All Products" Page (Consumes store)
│   ├── Favorites.js      # "Favorites" Page (Consumes store)
│   └── Counter.js        # Counter Demo (Consumes store)
├── context/              # (Legacy/Comparison) Context API alternative
├── App.js                # Main Layout & Routing
└── index.js              # Entry Point & Store Initialization
```

---

## How to run the project

1. **Clone the repository:**
   ```bash
   git clone https://github.com/ibrahimabdullaziz/react-custom-hooks-store.git
   ```
2. **Install dependencies:**
   ```bash
   npm install
   ```
3. **Start the development server:**
   ```bash
   npm start
   ```

---

## Usage Guide

### 1. The Core Hook (`hooks-store/store.js`)
This is the reusable engine. You don't often touch this file, but it's important to understand it.

```javascript
/* structure of store.js */
let globalState = {}; // The single source of truth
let listeners = [];   // Array of functions to call on update (setState)
let actions = {};     // Dictionary of action functions

export const useStore = (shouldListen = true) => { ... } // Custom Hook
export const initStore = (userActions, initialState) => { ... } // Setup function
```

### 2. Creating a State Slice
Create a new file (e.g., `auth-store.js`) to manage a specific part of your data.

```javascript
import { initStore } from './store';

const configureStore = () => {
  const actions = {
    // Action: (current_state, payload) => new_state_slice
    LOGIN: (state, userData) => ({ isLoggedIn: true, user: userData }),
    LOGOUT: (state) => ({ isLoggedIn: false, user: null }),
  };

  const initialState = {
    isLoggedIn: false,
    user: null
  };

  initStore(actions, initialState);
};

export default configureStore;
```

### 3. Initializing the Store
In your application entry point (usually `index.js`), run the configuration functions **once**.

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import configureStore from './hooks-store/products-store'; // Import your slices
import configureAuthStore from './hooks-store/auth-store';

// Initialize Store Slices
configureStore();
configureAuthStore();

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

### 4. Consuming State (Reading Data)
Use the `useStore` hook in any component. It returns the global state and a dispatch function.

```javascript
import React, { useContext } from 'react';
import { useStore } from '../hooks-store/store';

const Products = props => {
  const state = useStore()[0];
  return (
    <ul className="products-list">
      {state.products.map(prod => (
        <ProductItem
          key={prod.id}
          id={prod.id}
          title={prod.title}
          description={prod.description}
          isFav={prod.isFavorite}
        />
      ))}
    </ul>
  );
};
```

### 5. Dispatching Actions (Optimized)
If a component **only** triggers actions and doesn't need to render based on data changes, pass `false` to `useStore`. This **prevents unnecessary re-renders**, a significant performance advantage over standard Context API.

```javascript
import React from 'react';
import { useStore } from '../../hooks-store/store';

const ProductItem = React.memo(props => {
  const dispatch = useStore(false)[1];

  const toggleFavHandler = () => {
    // toggleFav(props.id);
    dispatch('TOGGLE_FAV', props.id);
  };

  return (
    <Card style={{ marginBottom: '1rem' }}>
      <div className="product-item">
        <h2 className={props.isFav ? 'is-fav' : ''}>{props.title}</h2>
        <p>{props.description}</p>
        <button
          className={!props.isFav ? 'button-outline' : ''}
          onClick={toggleFavHandler}
        >
          {props.isFav ? 'Un-Favorite' : 'Favorite'}
        </button>
      </div>
    </Card>
  );
});
```

---

##  Comparison

| Feature | 🔴 Redux | 🔵 Context API | 🟢 This Custom Store |
| :--- | :--- | :--- | :--- |
| **Setup & Boilerplate** | High (Reducers, Types, Providers) | Medium (Providers per context) | **Very Low** (Hooks only) |
| **Performance** | High (Selectors) | Low (Renders whole provider tree) | **High** (Targeted re-renders) |
| **Global State** | Yes | Yes | **Yes** |
| **Async Logic** | Middleware (Thunk/Saga) | Manual | **Manual** (Just call async then dispatch) |
| **DevTools** | Excellent | Average | **Console / N/A** |
| **Bundle Size** | Heavy | Zero (Built-in) | **Super Light** (~1kb) |

---

## Author
**Ibrahim Abdullaziz**
