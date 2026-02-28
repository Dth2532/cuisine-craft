# Passwordless Email Link Authentication Setup

This guide shows you how to enable "magic link" passwordless login where users click a link in their email instead of entering a password.

---

## Step 1: Enable Email Link Sign-In in Firebase Console

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Select your project
3. Navigate to **Build > Authentication > Sign-in method**
4. Click on **Email/Password**
5. Enable **Email/Password** (required as the base provider)
6. Enable **Email link (passwordless sign-in)** toggle
7. Click **Save**

---

## Step 2: Add Your Domain to Authorized Domains

1. Still in **Authentication**, go to the **Settings** tab
2. Click on **Authorized domains**
3. Add your domain(s):
   - `localhost` (for development)
   - Your production domain (e.g., `mealcraft.example.com`)

---

## Step 3: Configure Action URL (Dynamic Links)

For email links to work, Firebase needs to know where to redirect users:

1. Go to **Authentication > Settings > Email Templates**
2. Click on **Email address verification** 
3. Note the **Action URL** - this is where users are redirected
4. You can customize this URL in the code (see below)

---

## Step 4: Update Your Code

Replace the authentication section in your `index.html` with the passwordless version below.

### Updated HTML (Auth Forms Section)

Replace the existing auth forms with:

```html
<!-- Auth Screen -->
<div class="auth-screen" id="authScreen">
  <div class="auth-container">
    <div class="auth-logo">
      <div class="auth-logo-icon">🍳</div>
      <div class="auth-logo-text">MealCraft</div>
    </div>

    <div class="auth-error" id="authError"></div>
    <div class="auth-success" id="authSuccess"></div>

    <!-- Email Entry Form -->
    <form class="auth-form active" id="emailForm">
      <h3 style="text-align: center; margin-bottom: 8px; font-family: 'Playfair Display', serif;">Welcome!</h3>
      <p style="text-align: center; color: var(--text-secondary); margin-bottom: 24px;">Enter your email to sign in or create an account</p>
      
      <div class="form-group">
        <label class="form-label">Email Address</label>
        <input type="email" class="form-input" id="authEmail" placeholder="you@example.com" required>
      </div>
      
      <button type="submit" class="auth-btn">
        <span>✉️ Send Magic Link</span>
      </button>
      
      <p style="text-align: center; color: var(--text-muted); margin-top: 20px; font-size: 0.85rem;">
        We'll send you a link to sign in instantly — no password needed!
      </p>
    </form>

    <!-- Email Sent Confirmation -->
    <div class="auth-form" id="emailSentForm">
      <div style="text-align: center;">
        <div style="font-size: 64px; margin-bottom: 20px;">📬</div>
        <h3 style="font-family: 'Playfair Display', serif; margin-bottom: 8px;">Check Your Email</h3>
        <p style="color: var(--text-secondary); margin-bottom: 24px;">
          We sent a magic link to<br>
          <strong id="sentToEmail">your email</strong>
        </p>
        <p style="color: var(--text-muted); font-size: 0.9rem; margin-bottom: 24px;">
          Click the link in the email to sign in. The link expires in 1 hour.
        </p>
        <button type="button" class="auth-btn" style="background: var(--bg-elevated);" onclick="showEmailForm()">
          ← Use Different Email
        </button>
      </div>
    </div>

    <!-- Complete Sign In (for new users) -->
    <form class="auth-form" id="completeSignInForm">
      <h3 style="text-align: center; margin-bottom: 8px; font-family: 'Playfair Display', serif;">Almost There!</h3>
      <p style="text-align: center; color: var(--text-secondary); margin-bottom: 24px;">Enter your name to complete setup</p>
      
      <div class="form-group">
        <label class="form-label">Your Name</label>
        <input type="text" class="form-input" id="newUserName" placeholder="John Doe" required>
      </div>
      
      <button type="submit" class="auth-btn">
        <span>🚀 Complete Sign Up</span>
      </button>
    </form>
  </div>
</div>
```

### Updated CSS (Add These Styles)

```css
.auth-success {
  background: rgba(46, 204, 113, 0.15);
  border: 1px solid var(--accent-success);
  color: var(--accent-success);
  padding: 12px 16px;
  border-radius: var(--radius-md);
  margin-bottom: 20px;
  font-size: 0.9rem;
  display: none;
}

.auth-success.show {
  display: block;
}
```

### Updated JavaScript (Replace Auth Section)

```javascript
// =====================
// Firebase Configuration
// =====================
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};

firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.firestore();

// Action code settings for email link
const actionCodeSettings = {
  // URL to redirect to after clicking the email link
  // IMPORTANT: This must be whitelisted in Firebase Console
  url: window.location.origin + window.location.pathname,
  handleCodeInApp: true,
};

// =====================
// Passwordless Auth Functions
// =====================

// Send magic link email
document.getElementById('emailForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  const email = document.getElementById('authEmail').value;
  
  showLoading();
  try {
    await auth.sendSignInLinkToEmail(email, actionCodeSettings);
    
    // Save email locally to complete sign-in when user clicks the link
    window.localStorage.setItem('emailForSignIn', email);
    
    // Show confirmation
    document.getElementById('sentToEmail').textContent = email;
    showForm('emailSentForm');
    
    showAuthSuccess('Magic link sent! Check your inbox.');
  } catch (error) {
    showAuthError(getAuthErrorMessage(error.code));
  }
  hideLoading();
});

// Complete sign-in when user arrives via email link
async function completeSignInWithEmailLink() {
  // Check if this is a sign-in link
  if (!auth.isSignInWithEmailLink(window.location.href)) {
    return false;
  }
  
  showLoading();
  
  // Get the email from localStorage
  let email = window.localStorage.getItem('emailForSignIn');
  
  // If email is missing (e.g., user opened link on different device), prompt for it
  if (!email) {
    email = window.prompt('Please enter your email to confirm sign-in:');
    if (!email) {
      hideLoading();
      showAuthError('Email is required to complete sign-in.');
      return false;
    }
  }
  
  try {
    const result = await auth.signInWithEmailLink(email, window.location.href);
    
    // Clear the email from storage
    window.localStorage.removeItem('emailForSignIn');
    
    // Clear the URL to remove the sign-in link parameters
    window.history.replaceState({}, document.title, window.location.pathname);
    
    // Check if this is a new user
    if (result.additionalUserInfo?.isNewUser) {
      hideLoading();
      showForm('completeSignInForm');
      return true;
    }
    
    // Existing user - they're now signed in
    hideLoading();
    return true;
    
  } catch (error) {
    hideLoading();
    showAuthError(getAuthErrorMessage(error.code));
    return false;
  }
}

// Complete sign-up for new users (set their name)
document.getElementById('completeSignInForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  const name = document.getElementById('newUserName').value;
  
  showLoading();
  try {
    const user = auth.currentUser;
    
    // Update profile with name
    await user.updateProfile({ displayName: name });
    
    // Create user document in Firestore
    await db.collection('users').doc(user.uid).set({
      name: name,
      email: user.email,
      createdAt: firebase.firestore.FieldValue.serverTimestamp(),
      favorites: [],
      mealPlan: {}
    });
    
    // Refresh auth state
    await user.reload();
    
    // Now the onAuthStateChanged will handle the rest
    hideLoading();
  } catch (error) {
    showAuthError('Error completing sign-up. Please try again.');
    hideLoading();
  }
});

// Helper functions
function showForm(formId) {
  document.querySelectorAll('.auth-form').forEach(f => f.classList.remove('active'));
  document.getElementById(formId).classList.add('active');
  document.getElementById('authError').classList.remove('show');
  document.getElementById('authSuccess').classList.remove('show');
}

function showEmailForm() {
  showForm('emailForm');
}

function showAuthError(message) {
  const errorEl = document.getElementById('authError');
  errorEl.textContent = message;
  errorEl.classList.add('show');
  document.getElementById('authSuccess').classList.remove('show');
}

function showAuthSuccess(message) {
  const successEl = document.getElementById('authSuccess');
  successEl.textContent = message;
  successEl.classList.add('show');
  document.getElementById('authError').classList.remove('show');
}

function getAuthErrorMessage(code) {
  const messages = {
    'auth/invalid-email': 'Please enter a valid email address.',
    'auth/invalid-action-code': 'This link is invalid or has expired. Please request a new one.',
    'auth/expired-action-code': 'This link has expired. Please request a new one.',
    'auth/user-disabled': 'This account has been disabled.',
    'auth/too-many-requests': 'Too many attempts. Please try again later.',
  };
  return messages[code] || 'An error occurred. Please try again.';
}

// Check for email link on page load
window.addEventListener('DOMContentLoaded', () => {
  completeSignInWithEmailLink();
});

// Auth State Observer
auth.onAuthStateChanged(async (user) => {
  if (user) {
    // Check if user has completed profile setup
    if (!user.displayName) {
      authScreen.style.display = 'flex';
      showForm('completeSignInForm');
      return;
    }
    
    state.user = user;
    authScreen.style.display = 'none';
    appContainer.classList.add('active');
    bottomNav.style.display = 'block';
    
    // Update UI with user info
    document.getElementById('userAvatar').textContent = user.displayName[0].toUpperCase();
    document.getElementById('userName').textContent = user.displayName;
    document.getElementById('userEmail').textContent = user.email;
    
    // Load user data
    await loadUserData();
    checkForSharedPlan();
  } else {
    state.user = null;
    authScreen.style.display = 'flex';
    appContainer.classList.remove('active');
    bottomNav.style.display = 'none';
    showForm('emailForm');
  }
});

// Logout
document.getElementById('logoutBtn').addEventListener('click', async () => {
  await auth.signOut();
});
```

---

## How It Works

### User Flow

1. **New User**:
   - Enters email → clicks "Send Magic Link"
   - Receives email with sign-in link
   - Clicks link → redirected back to app
   - Enters their name → account created!

2. **Returning User**:
   - Enters email → clicks "Send Magic Link"
   - Clicks link in email → immediately signed in

3. **Different Device/Browser**:
   - If user opens link on different device, they'll be prompted to re-enter their email for security

### Security Benefits

- No passwords to remember or leak
- Links expire after 1 hour
- Single-use links (can't be reused)
- Email verification built-in

---

## Email Template Customization

To customize the email users receive:

1. Go to **Firebase Console > Authentication > Templates**
2. Click on **Email address sign-in**
3. Customize:
   - Subject line
   - Sender name
   - Email body text

Example custom subject: `Your MealCraft Magic Link 🍳`

---

## Testing Locally

1. Make sure `localhost` is in your authorized domains
2. Use a real email address (Firebase sends actual emails)
3. Check spam folder if you don't see the email

---

## Troubleshooting

### "The link is invalid" error
- The link may have expired (1 hour limit)
- The link was already used (single-use)
- The URL isn't whitelisted in authorized domains

### Email not arriving
- Check spam/junk folder
- Verify the email address is correct
- Check Firebase Console for quota limits

### "Cross-device sign-in" prompt appearing
- This is normal when opening the link on a different device/browser
- User just needs to re-enter their email for security

---

## Production Checklist

- [ ] Add your production domain to authorized domains
- [ ] Customize email template with your branding
- [ ] Update `actionCodeSettings.url` to your production URL
- [ ] Test the full flow on production domain
