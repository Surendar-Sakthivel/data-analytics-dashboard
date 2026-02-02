# Frontend Setup Plan - React + TypeScript

A step-by-step guide to set up the React frontend for the Data Analytics Dashboard.

---

## 🎯 What We'll Build

A React application with:
- TypeScript for type safety
- Tailwind CSS for styling
- React Router for navigation
- Axios for API calls
- Basic authentication flow

---

## 📋 Prerequisites

Before starting, make sure you have:
- ✅ Node.js installed (check: `node --version`)
- ✅ npm installed (check: `npm --version`)
- ✅ Docker databases running (check: `docker-compose ps`)

---

## 🚀 Phase 1: Initialize React App (15 minutes)

### Step 1: Navigate to Project
```bash
cd ~/Desktop/data-analytics-dashboard
```

### Step 2: Create React App with TypeScript
```bash
cd frontend
npx create-react-app . --template typescript
```

**What this does:**
- Creates a React app in the `frontend` folder
- Uses TypeScript instead of JavaScript
- Sets up all build tools automatically

**Expected output:**
```
Success! Created frontend
```

### Step 3: Verify Installation
```bash
npm start
```

**What happens:**
- Opens browser to `http://localhost:3000`
- Shows React logo spinning
- Press `Ctrl+C` to stop

✅ **Checkpoint:** React app is running!

---

## 🎨 Phase 2: Install Dependencies (5 minutes)

### Step 1: Install Core Libraries
```bash
npm install react-router-dom axios
npm install --save-dev @types/react-router-dom
```

**What each does:**
- `react-router-dom` - Navigation between pages
- `axios` - Make API calls to backend
- `@types/*` - TypeScript type definitions

### Step 2: Install Tailwind CSS
```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

**What this does:**
- Installs Tailwind CSS for styling
- Creates `tailwind.config.js` file

### Step 3: Configure Tailwind

Edit `tailwind.config.js`:
```js
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

Edit `src/index.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

✅ **Checkpoint:** All dependencies installed!

---

## 📁 Phase 3: Create Project Structure (10 minutes)

### Step 1: Create Folder Structure
```bash
cd src
mkdir -p components pages services hooks context types utils
```

**Folder purposes:**
- `components/` - Reusable UI components (buttons, forms, etc.)
- `pages/` - Full page components (Login, Dashboard, etc.)
- `services/` - API calls to backend
- `hooks/` - Custom React hooks
- `context/` - Global state management
- `types/` - TypeScript type definitions
- `utils/` - Helper functions

### Step 2: Create Initial Files

**Create `src/types/index.ts`:**
```typescript
export interface User {
  id: number;
  email: string;
  username: string;
}

export interface Dataset {
  id: number;
  name: string;
  size: number;
  uploaded_at: string;
}

export interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}
```

**Create `src/services/api.ts`:**
```typescript
import axios from 'axios';

const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:8000';

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Add token to requests if available
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default api;
```

✅ **Checkpoint:** Project structure created!

---

## 🔐 Phase 4: Build Authentication (20 minutes)

### Step 1: Create Auth Context

**Create `src/context/AuthContext.tsx`:**
```typescript
import React, { createContext, useState, useContext, ReactNode } from 'react';
import { User, AuthContextType } from '../types';
import api from '../services/api';

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    try {
      const response = await api.post('/auth/login', { email, password });
      const { token, user } = response.data;
      localStorage.setItem('token', token);
      setUser(user);
    } catch (error) {
      throw new Error('Login failed');
    }
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, isAuthenticated: !!user }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};
```

### Step 2: Create Login Page

**Create `src/pages/LoginPage.tsx`:**
```typescript
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

const LoginPage: React.FC = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const { login } = useAuth();
  const navigate = useNavigate();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      await login(email, password);
      navigate('/dashboard');
    } catch (err) {
      setError('Invalid credentials');
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100">
      <div className="bg-white p-8 rounded-lg shadow-md w-96">
        <h2 className="text-2xl font-bold mb-6">Login</h2>
        {error && <p className="text-red-500 mb-4">{error}</p>}
        <form onSubmit={handleSubmit}>
          <div className="mb-4">
            <label className="block text-gray-700 mb-2">Email</label>
            <input
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              className="w-full px-3 py-2 border rounded"
              required
            />
          </div>
          <div className="mb-6">
            <label className="block text-gray-700 mb-2">Password</label>
            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              className="w-full px-3 py-2 border rounded"
              required
            />
          </div>
          <button
            type="submit"
            className="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600"
          >
            Login
          </button>
        </form>
      </div>
    </div>
  );
};

export default LoginPage;
```

### Step 3: Create Dashboard Page

**Create `src/pages/DashboardPage.tsx`:**
```typescript
import React from 'react';
import { useAuth } from '../context/AuthContext';
import { useNavigate } from 'react-router-dom';

const DashboardPage: React.FC = () => {
  const { user, logout } = useAuth();
  const navigate = useNavigate();

  const handleLogout = () => {
    logout();
    navigate('/login');
  };

  return (
    <div className="min-h-screen bg-gray-100">
      <nav className="bg-white shadow-md p-4">
        <div className="container mx-auto flex justify-between items-center">
          <h1 className="text-xl font-bold">Data Analytics Dashboard</h1>
          <div className="flex items-center gap-4">
            <span>Welcome, {user?.username || 'User'}</span>
            <button
              onClick={handleLogout}
              className="bg-red-500 text-white px-4 py-2 rounded hover:bg-red-600"
            >
              Logout
            </button>
          </div>
        </div>
      </nav>
      <main className="container mx-auto p-8">
        <h2 className="text-2xl font-bold mb-4">Dashboard</h2>
        <p>Welcome to your analytics dashboard!</p>
      </main>
    </div>
  );
};

export default DashboardPage;
```

✅ **Checkpoint:** Authentication pages created!

---

## 🧭 Phase 5: Set Up Routing (10 minutes)

### Step 1: Update App.tsx

**Edit `src/App.tsx`:**
```typescript
import React from 'react';
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { AuthProvider, useAuth } from './context/AuthContext';
import LoginPage from './pages/LoginPage';
import DashboardPage from './pages/DashboardPage';

const ProtectedRoute: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { isAuthenticated } = useAuth();
  return isAuthenticated ? <>{children}</> : <Navigate to="/login" />;
};

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          <Route path="/login" element={<LoginPage />} />
          <Route
            path="/dashboard"
            element={
              <ProtectedRoute>
                <DashboardPage />
              </ProtectedRoute>
            }
          />
          <Route path="/" element={<Navigate to="/dashboard" />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}

export default App;
```

### Step 2: Create Environment File

**Create `.env` in frontend folder:**
```
REACT_APP_API_URL=http://localhost:8000
```

✅ **Checkpoint:** Routing configured!

---

## ✅ Phase 6: Test the Setup (5 minutes)

### Step 1: Start Development Server
```bash
cd ~/Desktop/data-analytics-dashboard/frontend
npm start
```

### Step 2: Verify Pages Load
- Should open browser to `http://localhost:3000`
- Should redirect to `/login`
- Login page should display with email/password form

### Step 3: Check for Errors
- Open browser console (F12)
- Should see no errors
- Tailwind CSS should be working (page should be styled)

✅ **Checkpoint:** Frontend is working!

---

## 📊 What You'll Have After Setup

### File Structure:
```
frontend/
├── public/
├── src/
│   ├── components/         (empty, ready for components)
│   ├── pages/
│   │   ├── LoginPage.tsx
│   │   └── DashboardPage.tsx
│   ├── services/
│   │   └── api.ts
│   ├── context/
│   │   └── AuthContext.tsx
│   ├── types/
│   │   └── index.ts
│   ├── hooks/              (empty, ready for hooks)
│   ├── utils/              (empty, ready for utilities)
│   ├── App.tsx
│   ├── index.tsx
│   └── index.css
├── .env
├── package.json
├── tailwind.config.js
└── tsconfig.json
```

### Features Working:
- ✅ React with TypeScript
- ✅ Tailwind CSS styling
- ✅ React Router navigation
- ✅ Authentication context
- ✅ Login page UI
- ✅ Dashboard page UI
- ✅ Protected routes
- ✅ API service layer

### Not Yet Working (Need Backend):
- ❌ Actual login (backend not ready)
- ❌ User registration
- ❌ Data fetching

---

## 🚫 Common Issues & Solutions

### Issue: "npm command not found"
**Solution:** Install Node.js from https://nodejs.org/

### Issue: "Port 3000 already in use"
**Solution:**
```bash
# Find what's using port 3000
lsof -i :3000
# Kill that process
kill -9 <PID>
```

### Issue: Tailwind CSS not working
**Solution:**
1. Make sure you added Tailwind directives to `src/index.css`
2. Restart dev server: `Ctrl+C` then `npm start`

### Issue: TypeScript errors
**Solution:**
- Most errors will go away once backend is built
- For now, you can add `// @ts-ignore` above error lines

---

## 📝 Commands Summary

```bash
# Navigate to frontend
cd ~/Desktop/data-analytics-dashboard/frontend

# Install dependencies
npm install

# Start development server
npm start

# Build for production (later)
npm run build

# Run tests (later)
npm test
```

---

## 🎯 Next Steps After Frontend Setup

Once frontend is set up:

1. **Set up FastAPI backend**
   - Create authentication endpoints
   - Connect to PostgreSQL
   - Handle user registration/login

2. **Connect frontend to backend**
   - Test login flow
   - Implement registration

3. **Build core features**
   - File upload page
   - Dataset list page
   - SQL query interface
   - Data visualization

---

## ⏰ Time Estimate

- Phase 1: 15 minutes
- Phase 2: 5 minutes
- Phase 3: 10 minutes
- Phase 4: 20 minutes
- Phase 5: 10 minutes
- Phase 6: 5 minutes

**Total: ~65 minutes** (1 hour)

---

## 💡 Tips for Tomorrow

1. **Take breaks** - Don't rush through setup
2. **Read error messages** - They usually tell you what's wrong
3. **Use browser DevTools** - F12 to see console errors
4. **Commit often** - Save progress after each phase
5. **Ask questions** - If stuck, ask for help!

---

## 🎓 Learning Resources

**React:**
- Official Docs: https://react.dev/learn
- TypeScript with React: https://react.dev/learn/typescript

**Tailwind CSS:**
- Official Docs: https://tailwindcss.com/docs
- Playground: https://play.tailwindcss.com/

**React Router:**
- Official Docs: https://reactrouter.com/

---

## ✅ Checklist for Tomorrow

Before starting:
- [ ] Docker databases are running
- [ ] You're in the project directory
- [ ] Terminal is open
- [ ] You have 1-2 hours of uninterrupted time

During setup:
- [ ] Phase 1: Initialize React app
- [ ] Phase 2: Install dependencies
- [ ] Phase 3: Create folder structure
- [ ] Phase 4: Build authentication
- [ ] Phase 5: Set up routing
- [ ] Phase 6: Test everything

After setup:
- [ ] Commit changes to git
- [ ] Push to GitHub
- [ ] Celebrate! 🎉

---

Good luck! See you tomorrow! 🚀
