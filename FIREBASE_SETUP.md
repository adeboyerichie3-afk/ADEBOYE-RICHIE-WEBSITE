# Firebase setup for Reviews

This project uses Firebase Firestore to store public reviews for gallery items.

Files added:
- `reviews.html` — public reviews page and submission form (client-side Firestore integration)
- `gallery.html` — updated to add `data-item` attributes and link images to the reviews page

Important: The Firebase web config is embedded in `reviews.html` so the page can connect to your Firestore project. The `apiKey` and other config fields are safe to include in client-side code. Firestore access is protected by security rules (see below).

Recommended Firestore rules (restrict updates/deletes to admin only):

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /reviews/{reviewId} {
      allow read: if true;
      allow create: if request.resource.data.keys().hasAll(['rating','comment'])
                    && (request.resource.data.rating is int || request.resource.data.rating is float)
                    && request.resource.data.rating >= 1
                    && request.resource.data.rating <= 5
                    && request.resource.data.comment.size() <= 1000
                    && (request.resource.data.name == null || request.resource.data.name.size() <= 100)
                    && (request.resource.data.itemId == null || request.resource.data.itemId.size() <= 200);
      // only admins can update/delete — replace 'your-admin-uid' with your UID later
      allow update, delete: if request.auth != null && request.auth.uid == 'your-admin-uid';
    }
  }
}
```

Anti-spam notes:
- Public write access allows anyone to post. To reduce spam consider:
  - Adding reCAPTCHA and server-side verification (Cloud Function) before creating reviews.
  - Storing reviews in a `pending_reviews` collection and approving them manually in the Firebase console.
  - Periodically reviewing and deleting spam via the Firebase console.

If you want, I can add a moderation UI (requires adding admin auth) or a Cloud Function to verify reCAPTCHA tokens before publishing reviews.

How to test locally:
1. Open `reviews.html` in a browser.
2. The page will display reviews and allow submission.
3. To view reviews for a specific item, open `reviews.html?item=pop1.jpg` (item name must match the `data-item` on images in `gallery.html`).

