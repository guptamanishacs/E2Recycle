# E2Recycle Project Code Explanation

## Project Overview
E2Recycle is a comprehensive e-waste management platform that allows users to submit recycling requests, tracks commission payments, and provides admin oversight. The project follows a full-stack architecture with React frontend and Node.js/Express backend.

---

## Backend Code Structure

### 1. Server Configuration (server.js)

```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const dotenv = require('dotenv');
const path = require('path');

// Load environment variables with explicit path
const envPath = path.join(__dirname, '.env');
console.log('Loading .env from:', envPath);
dotenv.config({ path: envPath });
```

**Explanation:**
- Sets up the main Express server
- Loads environment variables from .env file (admin credentials, database URL)
- Configures CORS for frontend-backend communication
- Establishes MongoDB connection using Mongoose

**Key Features:**
- Environment variable debugging for admin login issues
- MongoDB connection with error handling
- CORS configuration for cross-origin requests
- Route mounting for different API endpoints

### 2. Database Models

#### User Model (models/User.js)
```javascript
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true
  },
  email: {
    type: String,
    required: true,
    unique: true
  },
  password: {
    type: String,
    required: true
  },
  userType: {
    type: String,
    enum: ['individual', 'business', 'educational', 'recycler'],
    required: true
  }
});
```

**Explanation:**
- Defines user data structure for different user types
- Includes validation for required fields and unique constraints
- Supports multiple user categories (individuals, businesses, educational institutions, recyclers)
- Password field for authentication (hashed using bcryptjs)

#### RecyclingRequest Model (models/RecyclingRequest.js)
```javascript
const recyclingRequestSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  productName: {
    type: String,
    required: true
  },
  productType: {
    type: String,
    required: true
  },
  quantity: {
    type: Number,
    required: true
  },
  estimatedPrice: {
    type: Number,
    required: true
  },
  status: {
    type: String,
    enum: ['pending', 'approved', 'rejected', 'accepted', 'completed'],
    default: 'pending'
  }
});
```

**Explanation:**
- Core model for recycling requests submitted by users
- Links to User model through userId reference
- Tracks product details (name, type, quantity, estimated price)
- Status workflow: pending → approved → accepted → completed
- Includes location, contact details, and admin approval system

#### Transaction Model (models/Transaction.js)
```javascript
const transactionSchema = new mongoose.Schema({
  recyclingRequestId: {
    type: mongoose.Schema.Types.ObjectId,
    required: true,
    ref: 'RecyclingRequest'
  },
  recyclerId: {
    type: mongoose.Schema.Types.ObjectId,
    required: true,
    ref: 'User'
  },
  orderAmount: {
    type: Number,
    required: true
  },
  commissionRate: {
    type: Number,
    default: 8 // 8% commission
  },
  commissionAmount: {
    type: Number,
    required: true
  },
  status: {
    type: String,
    enum: ['pending', 'paid', 'confirmed', 'disputed'],
    default: 'pending'
  }
});
```

**Explanation:**
- Manages the 8% commission system for completed transactions
- Links recycling requests to recyclers who accepted them
- Calculates commission automatically (8% of order amount)
- Tracks payment status from pending to confirmed
- Includes admin confirmation workflow

### 3. API Routes

#### Authentication Routes (routes/auth.js)
```javascript
// User Registration
router.post('/register', async (req, res) => {
  try {
    const { username, email, password, userType } = req.body;
    
    // Check if user exists
    const existingUser = await User.findOne({
      $or: [{ email }, { username }]
    });
    
    if (existingUser) {
      return res.status(400).json({
        message: 'User with this email or username already exists'
      });
    }

    // Hash password
    const saltRounds = 10;
    const hashedPassword = await bcrypt.hash(password, saltRounds);

    // Create user
    const user = new User({
      username,
      email,
      password: hashedPassword,
      userType
    });

    await user.save();
    res.status(201).json({ message: 'User registered successfully' });
  } catch (error) {
    res.status(500).json({ message: 'Server error during registration' });
  }
});
```

**Explanation:**
- Handles user registration with password hashing
- Checks for duplicate users by email/username
- Uses bcryptjs for secure password storage
- Supports different user types (individual, business, educational, recycler)

```javascript
// User Login
router.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;

    // Check for admin login
    if (email === process.env.ADMIN_EMAIL && password === process.env.ADMIN_PASSWORD) {
      const adminToken = jwt.sign(
        { id: 'admin' },
        process.env.JWT_SECRET || 'your-secret-key',
        { expiresIn: '24h' }
      );
      return res.json({
        token: adminToken,
        user: { id: 'admin', email: process.env.ADMIN_EMAIL, userType: 'admin' }
      });
    }

    // Regular user login
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(400).json({ message: 'Invalid credentials' });
    }

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) {
      return res.status(400).json({ message: 'Invalid credentials' });
    }

    const token = jwt.sign(
      { id: user._id },
      process.env.JWT_SECRET || 'your-secret-key',
      { expiresIn: '24h' }
    );

    res.json({
      token,
      user: {
        id: user._id,
        username: user.username,
        email: user.email,
        userType: user.userType
      }
    });
  } catch (error) {
    res.status(500).json({ message: 'Server error during login' });
  }
});
```

**Explanation:**
- Dual authentication system: admin and regular users
- Admin login uses environment variables (e2recycle@gmail.com/admin123)
- Regular users use database authentication with bcrypt
- JWT token generation for session management
- Returns user information and authentication token

#### Transaction Routes (routes/transactions.js)
```javascript
// Get recycler's pending commission payments
router.get('/recycler/pending', auth, async (req, res) => {
  try {
    const recyclerId = req.user.id;

    const pendingTransactions = await Transaction.find({
      recyclerId,
      status: { $in: ['pending', 'paid'] }
    })
    .populate('recyclingRequestId', 'productName productType quantity uniqueCode')
    .sort({ createdAt: -1 });

    res.json({
      success: true,
      transactions: pendingTransactions
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Server error while fetching transactions'
    });
  }
});
```

**Explanation:**
- Fetches commission payments that recyclers need to pay
- Uses authentication middleware to identify the recycler
- Populates related recycling request data
- Returns transactions with pending or paid status

```javascript
// Submit commission payment
router.put('/:id/pay', auth, async (req, res) => {
  try {
    const { id } = req.params;
    const { paymentMethod, paymentReference } = req.body;
    const recyclerId = req.user.id;

    const transaction = await Transaction.findOne({
      _id: id,
      recyclerId,
      status: 'pending'
    });

    if (!transaction) {
      return res.status(404).json({
        success: false,
        message: 'Transaction not found or already processed'
      });
    }

    // Update transaction with payment info
    transaction.status = 'paid';
    transaction.paymentMethod = paymentMethod;
    transaction.paymentReference = paymentReference;
    transaction.paymentDate = new Date();

    await transaction.save();

    res.json({
      success: true,
      message: 'Payment submitted successfully. Waiting for admin confirmation.',
      transaction
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Server error while processing payment'
    });
  }
});
```

**Explanation:**
- Allows recyclers to submit commission payments
- Updates transaction status from pending to paid
- Records payment method and reference number
- Timestamps the payment submission

```javascript
// Confirm commission payment (by admin)
router.put('/:id/confirm', auth, async (req, res) => {
  try {
    const { id } = req.params;
    const { confirmed, adminNotes } = req.body;
    const adminId = req.user.id;

    const transaction = await Transaction.findOne({
      _id: id,
      status: 'paid'
    });

    if (!transaction) {
      return res.status(404).json({
        success: false,
        message: 'Transaction not found or not ready for confirmation'
      });
    }

    if (confirmed) {
      transaction.status = 'confirmed';
      if (adminId === 'admin') {
        transaction.confirmedBy = 'admin';
      } else {
        transaction.confirmedBy = adminId;
      }
      transaction.confirmedAt = new Date();
    } else {
      transaction.status = 'disputed';
    }
    
    if (adminNotes) {
      transaction.adminNotes = adminNotes;
    }

    await transaction.save();

    res.json({
      success: true,
      message: confirmed ? 'Payment confirmed successfully' : 'Payment marked as disputed',
      transaction
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Server error while confirming payment'
    });
  }
});
```

**Explanation:**
- Admin-only route for confirming recycler payments
- Can confirm or dispute payments
- Handles admin ID properly (string "admin" vs ObjectId)
- Records confirmation timestamp and admin notes

### 4. Authentication Middleware (middleware/auth.js)
```javascript
const auth = async (req, res, next) => {
  try {
    const token = req.header('Authorization');
    
    if (!token) {
      return res.status(401).json({ message: 'No token, authorization denied' });
    }

    // Remove Bearer from string
    const tokenValue = token.replace('Bearer ', '');
    
    // Check if token is valid format
    if (!tokenValue || tokenValue === 'null' || tokenValue === 'undefined') {
      return res.status(401).json({ message: 'Invalid token format' });
    }
    
    const decoded = jwt.verify(tokenValue, process.env.JWT_SECRET || 'your-secret-key');
    req.user = decoded;
    
    next();
  } catch (error) {
    res.status(401).json({ message: 'Token is not valid' });
  }
};
```

**Explanation:**
- Protects routes requiring authentication
- Extracts JWT token from Authorization header
- Verifies token using JWT_SECRET
- Adds user information to request object
- Handles invalid token formats and expired tokens

---

## Frontend Code Structure

### 1. Main Application (App.js)
```javascript
import React from 'react';
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import LandingPage from './components/LandingPage';
import LoginPage from './components/LoginPage';
import RegisterPage from './components/RegisterPage';
import Dashboard from './components/Dashboard';
import AdminDashboard from './components/AdminDashboard';

function App() {
  return (
    <Router>
      <div className="App">
        <Routes>
          <Route path="/" element={<LandingPage />} />
          <Route path="/login" element={<LoginPage />} />
          <Route path="/register" element={<RegisterPage />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/admin" element={<AdminDashboard />} />
          <Route path="*" element={<Navigate to="/" replace />} />
        </Routes>
      </div>
    </Router>
  );
}
```

**Explanation:**
- Sets up routing for the entire application
- Defines main routes: landing, login, register, dashboard, admin
- Uses React Router for navigation
- Includes catch-all route for invalid URLs

### 2. Landing Page Components

#### Hero Section (components/HeroSection.js)
```javascript
const HeroSection = () => {
  const sentences = useMemo(() => [
    "Transform your electronic waste into environmental impact",
    "Recycle responsibly, protect our planet's future",
    "Turn old devices into sustainable solutions",
    "Join the e-waste revolution for a greener tomorrow"
  ], []);

  const [currentSentenceIndex, setCurrentSentenceIndex] = useState(0);
  const [currentText, setCurrentText] = useState('');
  const [isTyping, setIsTyping] = useState(true);
  const [charIndex, setCharIndex] = useState(0);

  useEffect(() => {
    const currentSentence = sentences[currentSentenceIndex];
    
    if (isTyping) {
      // Typing effect
      if (charIndex < currentSentence.length) {
        const timer = setTimeout(() => {
          setCurrentText(currentSentence.substring(0, charIndex + 1));
          setCharIndex(charIndex + 1);
        }, 100);
        return () => clearTimeout(timer);
      } else {
        const timer = setTimeout(() => {
          setIsTyping(false);
        }, 2000);
        return () => clearTimeout(timer);
      }
    } else {
      // Deleting effect
      if (charIndex > 0) {
        const timer = setTimeout(() => {
          setCurrentText(currentSentence.substring(0, charIndex - 1));
          setCharIndex(charIndex - 1);
        }, 50);
        return () => clearTimeout(timer);
      } else {
        setCurrentSentenceIndex((prevIndex) => 
          prevIndex === sentences.length - 1 ? 0 : prevIndex + 1
        );
        setIsTyping(true);
      }
    }
  }, [charIndex, isTyping, currentSentenceIndex, sentences]);
```

**Explanation:**
- Creates animated typing effect for hero text
- Cycles through different environmental messages
- Uses React hooks (useState, useEffect, useMemo)
- Implements typing and deleting animation with timers
- Background image integration with contact information overlay

### 3. Authentication Components

#### Login Page (components/LoginPage.js)
```javascript
const handleSubmit = async (e) => {
  e.preventDefault();
  setLoading(true);
  setError('');

  try {
    const response = await fetch('http://localhost:5000/api/auth/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(formData),
    });

    const data = await response.json();

    if (response.ok) {
      // Store token and user data
      localStorage.setItem('token', data.token);
      localStorage.setItem('user', JSON.stringify(data.user));

      // Redirect based on user type
      if (data.user.userType === 'admin') {
        navigate('/admin');
      } else {
        navigate('/dashboard');
      }
    } else {
      setError(data.message || 'Login failed');
    }
  } catch (error) {
    setError('Network error. Please try again.');
  } finally {
    setLoading(false);
  }
};
```

**Explanation:**
- Handles user login with email/password
- Sends POST request to backend authentication endpoint
- Stores JWT token and user data in localStorage
- Redirects users based on type (admin → admin dashboard, others → user dashboard)
- Includes error handling and loading states

#### Registration Page (components/RegisterPage.js)
```javascript
const [registrationData, setRegistrationData] = useState({
  username: '',
  email: '',
  password: '',
  confirmPassword: '',
  userType: 'individual',
  // Type-specific fields
  phone: '',
  address: '',
  businessName: '',
  contactPerson: '',
  institutionName: '',
  department: '',
  companyName: '',
  recyclerType: '',
  certifications: '',
  operatingAreas: ''
});
```

**Explanation:**
- Multi-step registration supporting different user types
- Dynamic form fields based on selected user type
- Includes validation for required fields
- Password confirmation checking
- Submits data to backend registration endpoint

### 4. Dashboard Components

#### User Dashboard (components/Dashboard.js)
```javascript
const Dashboard = () => {
  const [user, setUser] = useState(null);
  const [activeView, setActiveView] = useState('overview');

  useEffect(() => {
    const userData = localStorage.getItem('user');
    if (userData) {
      setUser(JSON.parse(userData));
    }
  }, []);

  const renderContent = () => {
    switch (activeView) {
      case 'overview':
        return <div>Dashboard Overview</div>;
      case 'requests':
        return <RecyclingRequestForm />;
      case 'status':
        return user?.userType === 'recycler' ? <IncomingRequests /> : <AcceptedRequests />;
      case 'commission':
        return user?.userType === 'recycler' ? <CommissionPayments /> : null;
      default:
        return <div>Dashboard Overview</div>;
    }
  };
```

**Explanation:**
- Main dashboard with navigation sidebar
- Different views based on user type and selected menu
- Loads user data from localStorage
- Dynamic content rendering based on activeView state
- Supports recycler-specific features (commission payments, incoming requests)

#### Recycling Request Form (components/RecyclingRequestForm.js)
```javascript
const handleSubmit = async (e) => {
  e.preventDefault();
  setLoading(true);
  setError('');

  try {
    const token = localStorage.getItem('token');
    const response = await fetch('http://localhost:5000/api/recycling-requests', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`,
      },
      body: JSON.stringify(formData),
    });

    const data = await response.json();

    if (response.ok) {
      setSuccess('Recycling request submitted successfully!');
      setFormData({
        productName: '',
        productType: '',
        quantity: 1,
        description: '',
        estimatedPrice: '',
        condition: '',
        purchaseDate: '',
        location: '',
        preferredDate: '',
        phoneNumber: '',
        comments: ''
      });
    } else {
      setError(data.message || 'Failed to submit request');
    }
  } catch (error) {
    setError('Network error. Please try again.');
  } finally {
    setLoading(false);
  }
};
```

**Explanation:**
- Form for submitting new recycling requests
- Includes product details, location, contact information
- Uses authentication token for API requests
- Form validation and error handling
- Success/error feedback to users

### 5. Commission System Components

#### Commission Payments (components/CommissionPayments.js)
```javascript
const loadPendingTransactions = async () => {
  try {
    const token = localStorage.getItem('token');
    const response = await fetch('http://localhost:5000/api/transactions/recycler/pending', {
      headers: {
        'Authorization': `Bearer ${token}`,
      },
    });

    const data = await response.json();
    if (data.success) {
      setPendingTransactions(data.transactions);
    }
  } catch (error) {
    console.error('Error loading transactions:', error);
  }
};

const handlePaymentSubmit = async (transactionId, paymentData) => {
  try {
    const token = localStorage.getItem('token');
    const response = await fetch(`http://localhost:5000/api/transactions/${transactionId}/pay`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`,
      },
      body: JSON.stringify(paymentData),
    });

    const data = await response.json();
    if (data.success) {
      alert('Payment submitted successfully!');
      loadPendingTransactions();
      setShowPaymentModal(false);
    } else {
      alert('Payment submission failed: ' + data.message);
    }
  } catch (error) {
    alert('Error submitting payment: ' + error.message);
  }
};
```

**Explanation:**
- Shows recyclers their pending commission payments (8% of completed orders)
- Displays admin bank details for payment
- Payment submission with reference number
- Modal interface for payment process
- Updates transaction status after payment

#### Admin Transactions (components/AdminTransactions.js)
```javascript
const handleConfirmPayment = async (transactionId, confirmed, adminNotes = '') => {
  try {
    setProcessing(transactionId);
    
    const response = await fetch(`http://localhost:5000/api/transactions/${transactionId}/confirm`, {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        confirmed,
        adminNotes
      }),
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || 'Failed to update transaction');
    }

    alert(confirmed ? 'Payment confirmed successfully!' : 'Payment marked as disputed.');
    loadTransactions();
  } catch (error) {
    console.error('Error updating transaction:', error);
    alert('Failed to update transaction: ' + error.message);
  } finally {
    setProcessing(null);
  }
};
```

**Explanation:**
- Admin interface for confirming recycler payments
- View all transactions with detailed information
- Confirm or dispute payment submissions
- Add admin notes to transactions
- Status filtering (pending, paid, confirmed, disputed)

### 6. Request Management Components

#### Incoming Requests (components/IncomingRequests.js)
```javascript
const handleAcceptRequest = async (requestId) => {
  try {
    const token = localStorage.getItem('token');
    const response = await fetch(`http://localhost:5000/api/recycling-requests/${requestId}/accept`, {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json',
      },
    });

    if (response.ok) {
      alert('Request accepted successfully!');
      loadRequests();
    } else {
      const errorData = await response.json();
      if (errorData.pendingPayments > 0) {
        alert(`You have ${errorData.pendingPayments} pending commission payment(s). Please complete payment to accept new requests.`);
      } else {
        alert('Failed to accept request');
      }
    }
  } catch (error) {
    alert('Error accepting request: ' + error.message);
  }
};
```

**Explanation:**
- Shows approved recycling requests to recyclers
- Recyclers can accept requests for processing
- Blocks new request acceptance if commission payments are pending
- Hides secret codes from recyclers until payment confirmed
- Creates commission transactions when requests are accepted

### 7. Key Features Implementation

#### PDF Generation (using jsPDF + html2canvas)
```javascript
const generatePDF = async () => {
  try {
    setGeneratingPDF(true);
    
    const element = document.getElementById('requests-table');
    const canvas = await html2canvas(element, {
      scale: 2,
      logging: false,
      useCORS: true
    });
    
    const imgData = canvas.toDataURL('image/png');
    const pdf = new jsPDF('l', 'mm', 'a4');
    
    const imgWidth = 280;
    const pageHeight = 210;
    const imgHeight = (canvas.height * imgWidth) / canvas.width;
    let heightLeft = imgHeight;
    
    let position = 10;
    
    pdf.addImage(imgData, 'PNG', 10, position, imgWidth, imgHeight);
    heightLeft -= pageHeight;
    
    while (heightLeft >= 0) {
      position = heightLeft - imgHeight + 10;
      pdf.addPage();
      pdf.addImage(imgData, 'PNG', 10, position, imgWidth, imgHeight);
      heightLeft -= pageHeight;
    }
    
    pdf.save('recycling-requests.pdf');
  } catch (error) {
    console.error('Error generating PDF:', error);
    alert('Error generating PDF');
  } finally {
    setGeneratingPDF(false);
  }
};
```

**Explanation:**
- Converts HTML tables to PDF documents
- Uses html2canvas to create image from DOM elements
- jsPDF handles PDF generation and download
- Supports multi-page PDFs for large datasets
- Used for exporting recycling requests and transaction reports

#### Authentication Check
```javascript
useEffect(() => {
  const token = localStorage.getItem('token');
  const userData = localStorage.getItem('user');
  
  if (!token || !userData) {
    navigate('/login');
    return;
  }
  
  try {
    const user = JSON.parse(userData);
    if (user.userType !== 'admin') {
      navigate('/dashboard');
      return;
    }
    setUser(user);
  } catch (error) {
    navigate('/login');
  }
}, [navigate]);
```

**Explanation:**
- Checks authentication status on component mount
- Redirects unauthenticated users to login
- Validates user type for admin-only pages
- Handles invalid or corrupted localStorage data

---

## Key System Workflows

### 1. User Registration & Login Flow
```
User Registration → Password Hashing → Database Storage → 
Login → Password Verification → JWT Token Generation → 
Dashboard Access Based on User Type
```

### 2. Recycling Request Workflow
```
User Submits Request → Admin Reviews → Admin Approves → 
Recycler Accepts → Commission Transaction Created → 
Request Completed → Recycler Pays Commission → Admin Confirms Payment
```

### 3. Commission Payment System
```
Request Completion → 8% Commission Calculated → Transaction Created → 
Recycler Pays Commission → Admin Receives Payment → Admin Confirms → 
Recycler Can Accept New Requests
```

### 4. Admin Management
```
Admin Login (e2recycle@gmail.com) → View All Transactions → 
Review Payment Submissions → Confirm/Dispute Payments → 
Manage User Requests → Approve/Reject Submissions
```

## Security Features

1. **Password Security**: bcrypt hashing with salt rounds
2. **JWT Authentication**: Stateless session management
3. **Protected Routes**: Authentication middleware on sensitive endpoints
4. **Input Validation**: Server-side validation using express-validator
5. **CORS Protection**: Controlled cross-origin resource sharing
6. **Environment Variables**: Secure credential storage
7. **Admin Authentication**: Separate admin login system

## Database Design

- **Users Collection**: Stores all user types with role-based access
- **RecyclingRequests Collection**: Manages the complete request lifecycle
- **Transactions Collection**: Tracks 8% commission payments and confirmations
- **Relationships**: MongoDB references linking users, requests, and transactions

This comprehensive code structure creates a secure, scalable e-waste management platform with robust user authentication, commission tracking, and administrative oversight capabilities.