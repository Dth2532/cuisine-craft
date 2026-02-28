# MealCraft - Firebase Setup Guide

## Overview
MealCraft uses Firebase for authentication and data storage. Follow these steps to set up your own Firebase project.

---

## Step 1: Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **"Create a project"**
3. Enter a project name (e.g., "mealcraft")
4. Enable/disable Google Analytics (optional)
5. Click **"Create project"**

---

## Step 2: Enable Authentication

1. In Firebase Console, go to **Build > Authentication**
2. Click **"Get started"**
3. Go to **Sign-in method** tab
4. Enable **Email/Password**
5. Click **Save**

---

## Step 3: Create Firestore Database

1. Go to **Build > Firestore Database**
2. Click **"Create database"**
3. Select **"Start in production mode"**
4. Choose a location closest to your users
5. Click **Enable**

---

## Step 4: Set Up Security Rules

Go to **Firestore Database > Rules** and paste these rules:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can only access their own data
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
      
      // User's recipes subcollection
      match /recipes/{recipeId} {
        allow read, write: if request.auth != null && request.auth.uid == userId;
      }
    }
    
    // Shared meal plans - anyone can read, only owner can write
    match /sharedPlans/{planId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow update, delete: if request.auth != null && request.auth.uid == resource.data.userId;
    }
  }
}
```

Click **Publish** to save the rules.

---

## Step 5: Get Your Firebase Config

1. Go to **Project Settings** (gear icon)
2. Scroll down to **Your apps**
3. Click the **Web** icon (`</>`)
4. Register your app with a nickname
5. Copy the `firebaseConfig` object

It looks like this:
```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

---

## Step 6: Update the App

Open `index.html` and find the `firebaseConfig` section (around line 1380):

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  ...
};
```

Replace with your actual config values from Step 5.

---

## Step 7: Deploy (Optional)

### Option A: Firebase Hosting
```bash
npm install -g firebase-tools
firebase login
firebase init hosting
firebase deploy
```

### Option B: Other Platforms
- **Netlify**: Drag and drop your folder
- **Vercel**: Connect your GitHub repo
- **GitHub Pages**: Push to a `gh-pages` branch

---

## Firestore Data Structure

```
/users/{userId}
├── name: string
├── email: string
├── createdAt: timestamp
├── favorites: array[recipeId]
├── mealPlan: map
│   └── "2026-03-01": {
│       ├── breakfast: { recipeId: "..." }
│       ├── lunch: { recipeId: "..." }
│       └── dinner: { recipeId: "..." }
│   }
└── /recipes/{recipeId}  (subcollection)
    ├── name: string
    ├── emoji: string
    ├── category: string
    ├── description: string
    ├── time: number
    ├── servings: number
    ├── ingredients: array[string]
    ├── instructions: array[string]
    └── createdAt: timestamp

/sharedPlans/{planId}
├── userId: string
├── userName: string
├── date: string
├── meals: map
└── createdAt: timestamp
```

---

## Troubleshooting

### "Permission denied" errors
- Check that your Firestore rules are published correctly
- Ensure the user is authenticated

### Authentication not working
- Verify Email/Password is enabled in Firebase Auth
- Check that your `apiKey` is correct

### Data not saving
- Check browser console for errors
- Verify Firestore database was created

---

## Cost Considerations

Firebase has a generous free tier:
- **Authentication**: 50,000 monthly active users
- **Firestore**: 1 GiB storage, 50K reads/day, 20K writes/day

For a personal meal planning app, you'll likely stay well within free limits.

---

## Need Help?

- [Firebase Documentation](https://firebase.google.com/docs)
- [Firestore Guides](https://firebase.google.com/docs/firestore)
- [Firebase Auth Guides](https://firebase.google.com/docs/auth)
