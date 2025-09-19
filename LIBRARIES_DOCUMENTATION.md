# E2Recycle Project Libraries & Their Functions

## Backend Libraries (Node.js/Express)

### 1. **express** (v4.18.2)
**Purpose**: Web application framework for Node.js
**Work in Project**:
- Creates HTTP server and handles routing
- Manages middleware stack for request processing
- Provides RESTful API endpoints for frontend communication
- Handles HTTP methods (GET, POST, PUT, DELETE)
- Request/response object management
```javascript
const express = require('express');
const app = express();
app.get('/api/users', (req, res) => { /* handle request */ });
```

### 2. **mongoose** (v7.5.0)
**Purpose**: MongoDB object modeling library
**Work in Project**:
- Connects to MongoDB database
- Defines data schemas for Users, RecyclingRequests, Transactions
- Provides data validation and type casting
- Query building and database operations
- Population of referenced documents
```javascript
const mongoose = require('mongoose');
const userSchema = new mongoose.Schema({
  username: { type: String, required: true }
});
```

### 3. **bcryptjs** (v2.4.3)
**Purpose**: Password hashing library
**Work in Project**:
- Hashes user passwords before storing in database
- Compares plain text passwords with hashed versions during login
- Provides salt rounds for enhanced security
- Prevents storing plain text passwords
```javascript
const bcrypt = require('bcryptjs');
const hashedPassword = await bcrypt.hash(password, 10);
const isValid = await bcrypt.compare(password, hashedPassword);
```

### 4. **jsonwebtoken** (v9.0.2)
**Purpose**: JSON Web Token implementation
**Work in Project**:
- Generates JWT tokens for user authentication
- Creates admin tokens for administrative access
- Verifies tokens on protected routes
- Manages session state without server storage
- Token expiration handling
```javascript
const jwt = require('jsonwebtoken');
const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '24h' });
const decoded = jwt.verify(token, process.env.JWT_SECRET);
```

### 5. **cors** (v2.8.5)
**Purpose**: Cross-Origin Resource Sharing middleware
**Work in Project**:
- Enables frontend (React) to communicate with backend API
- Handles preflight OPTIONS requests
- Configures allowed origins, methods, and headers
- Manages cross-domain security policies
```javascript
const cors = require('cors');
app.use(cors({
  origin: 'http://localhost:3000',
  credentials: true
}));
```

### 6. **dotenv** (v16.3.1)
**Purpose**: Environment variable loader
**Work in Project**:
- Loads configuration from .env file
- Manages sensitive data (database URLs, admin credentials)
- Provides environment-specific settings
- Keeps secrets out of source code
```javascript
const dotenv = require('dotenv');
dotenv.config();
const adminEmail = process.env.ADMIN_EMAIL; // e2recycle@gmail.com
```

### 7. **express-validator** (v7.0.1)
**Purpose**: Server-side validation middleware
**Work in Project**:
- Validates incoming request data
- Sanitizes user input
- Provides validation error handling
- Ensures data integrity before database operations
```javascript
const { body, validationResult } = require('express-validator');
body('email').isEmail().normalizeEmail(),
body('password').isLength({ min: 6 })
```

### 8. **uuid** (v13.0.0)
**Purpose**: Unique identifier generation
**Work in Project**:
- Generates unique codes for recycling requests
- Creates secret codes for recycler access
- Provides unique transaction references
- File naming and identification purposes
```javascript
const { v4: uuidv4 } = require('uuid');
const uniqueCode = uuidv4(); // Generates: f47ac10b-58cc-4372-a567-0e02b2c3d479
```

### 9. **nodemon** (v3.0.1) [Development Only]
**Purpose**: Development server with auto-restart
**Work in Project**:
- Monitors file changes during development
- Automatically restarts server when code changes
- Improves development workflow
- Only used in development environment
```bash
npm run dev  # Uses nodemon to start server with auto-restart
```

---

## Frontend Libraries (React)

### 1. **react** (v18.2.0)
**Purpose**: JavaScript library for building user interfaces
**Work in Project**:
- Creates component-based UI architecture
- Manages component state with hooks (useState, useEffect)
- Handles user interactions and events
- Provides virtual DOM for efficient rendering
- Component lifecycle management
```javascript
import React, { useState, useEffect } from 'react';
const [user, setUser] = useState(null);
useEffect(() => { /* side effects */ }, []);
```

### 2. **react-dom** (v18.2.0)
**Purpose**: React rendering library for browser DOM
**Work in Project**:
- Renders React components to the browser DOM
- Handles browser-specific operations
- Manages event system and DOM updates
- Provides entry point for React applications
```javascript
import ReactDOM from 'react-dom/client';
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

### 3. **react-router-dom** (v6.26.2)
**Purpose**: Declarative routing for React applications
**Work in Project**:
- Manages navigation between different pages
- Provides protected routes for authenticated users
- Handles URL routing and browser history
- Role-based route access (user/admin dashboards)
- Dynamic route parameters
```javascript
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
<Routes>
  <Route path="/login" element={<LoginPage />} />
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/admin" element={<AdminDashboard />} />
</Routes>
```

### 4. **react-scripts** (v5.0.1)
**Purpose**: Create React App build toolchain
**Work in Project**:
- Provides development server with hot reloading
- Builds optimized production bundles
- Includes Webpack, Babel, ESLint configuration
- Testing framework setup with Jest
- Development and production build scripts
```bash
npm start    # Development server
npm run build # Production build
npm test     # Run tests
```

### 5. **jspdf** (v3.0.2)
**Purpose**: PDF generation library for client-side
**Work in Project**:
- Generates PDF reports from recycling request data
- Creates commission payment receipts
- Exports transaction summaries for admin
- Client-side PDF creation without server processing
```javascript
import jsPDF from 'jspdf';
const pdf = new jsPDF();
pdf.text('E2Recycle Report', 20, 20);
pdf.save('recycling-report.pdf');
```

### 6. **html2canvas** (v1.4.1)
**Purpose**: Screenshot library for HTML elements
**Work in Project**:
- Converts HTML tables to canvas images
- Captures dashboard screenshots for PDF inclusion
- Works with jsPDF to create visual reports
- Preserves styling and layout in exports
```javascript
import html2canvas from 'html2canvas';
const canvas = await html2canvas(document.getElementById('table'));
const imgData = canvas.toDataURL('image/png');
```

### 7. **uuid** (v13.0.0)
**Purpose**: Unique identifier generation (same as backend)
**Work in Project**:
- Generates client-side unique identifiers
- Creates temporary IDs for form components
- Tracking form submissions and states
- Component key generation for React lists
```javascript
import { v4 as uuidv4 } from 'uuid';
const componentId = uuidv4();
```

### 8. **web-vitals** (v2.1.4)
**Purpose**: Web performance metrics library
**Work in Project**:
- Monitors Core Web Vitals (LCP, FID, CLS)
- Tracks page load performance
- Provides user experience metrics
- Performance optimization insights
```javascript
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';
getCLS(console.log);
getFID(console.log);
```

---

## Testing Libraries

### 1. **@testing-library/react** (v16.3.0)
**Purpose**: React component testing utilities
**Work in Project**:
- Unit testing for React components
- Simulates user interactions
- Tests component rendering and behavior
- Integration testing for component workflows
```javascript
import { render, screen } from '@testing-library/react';
render(<LoginPage />);
expect(screen.getByText('Login')).toBeInTheDocument();
```

### 2. **@testing-library/jest-dom** (v6.8.0)
**Purpose**: Custom Jest matchers for DOM testing
**Work in Project**:
- Provides additional assertion methods
- Better test readability and error messages
- DOM-specific testing utilities
- Enhanced Jest testing capabilities
```javascript
expect(element).toBeInTheDocument();
expect(element).toHaveClass('active');
expect(element).toBeVisible();
```

### 3. **@testing-library/user-event** (v13.5.0)
**Purpose**: User interaction simulation for testing
**Work in Project**:
- Simulates realistic user interactions
- Tests clicking, typing, form submissions
- More accurate than fireEvent for user behavior
- Async user interaction testing
```javascript
import userEvent from '@testing-library/user-event';
await userEvent.type(screen.getByLabelText('Email'), 'test@example.com');
await userEvent.click(screen.getByRole('button', { name: 'Login' }));
```

### 4. **@testing-library/dom** (v10.4.1)
**Purpose**: DOM testing utilities (framework agnostic)
**Work in Project**:
- Provides core DOM querying functionality
- Base library for other testing-library packages
- Cross-browser DOM testing support
- Foundation for React testing utilities

---

## Library Integration Examples

### Authentication Flow
```javascript
// Backend: bcryptjs + jsonwebtoken
const hashedPassword = await bcrypt.hash(password, 10);
const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);

// Frontend: react-router-dom
if (authenticated) {
  navigate('/dashboard');
} else {
  navigate('/login');
}
```

### Database Operations
```javascript
// Backend: mongoose
const user = await User.findOne({ email });
const request = await RecyclingRequest.create({
  userId: user._id,
  productName: 'iPhone 12'
});
```

### PDF Report Generation
```javascript
// Frontend: html2canvas + jspdf
const canvas = await html2canvas(tableElement);
const pdf = new jsPDF();
pdf.addImage(canvas.toDataURL(), 'PNG', 10, 10);
pdf.save('report.pdf');
```

### Environment Configuration
```javascript
// Backend: dotenv
process.env.ADMIN_EMAIL;      // e2recycle@gmail.com
process.env.ADMIN_PASSWORD;   // admin123
process.env.MONGODB_URI;      // Database connection
```

### Form Validation
```javascript
// Backend: express-validator
body('email').isEmail(),
body('password').isLength({ min: 6 })

// Frontend: React state management
const [errors, setErrors] = useState({});
if (!email) errors.email = 'Email is required';
```

---

## Development vs Production Libraries

### Development Only
- **nodemon**: Auto-restart server during development
- **@testing-library/***: Testing frameworks
- **react-scripts**: Development server and build tools

### Production Libraries
- **express, mongoose, bcryptjs**: Core backend functionality
- **react, react-dom**: Core frontend framework
- **jspdf, html2canvas**: Report generation
- **jsonwebtoken**: Authentication in production

### Universal Libraries
- **uuid**: Both frontend and backend ID generation
- **cors**: Production CORS configuration
- **dotenv**: Environment management

This comprehensive library documentation shows how each dependency contributes to the E2Recycle platform's functionality, from user authentication to PDF generation and everything in between!