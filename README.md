# QuizQuest

A live classroom quiz game that runs in the browser. A teacher hosts a quiz and shares a 4-digit room code. Students join on their own phones and answer against a timer. Faster correct answers earn more points, and a leaderboard and podium show the results.

The whole app is one HTML file (`quizquest.html`) with no build step. Rooms and accounts are stored in Firebase Realtime Database so phones on different networks can join the same room.

## Features

**Teachers**
- Create an account with the Teacher role
- Create, edit and delete quizzes (title, time per question, 2 to 4 options per question)
- Edit or remove individual questions inside a quiz
- Host a quiz and share a room code, a link, or a QR code (with a full-screen mode for projectors)
- Change the time per question in the lobby
- Add or remove 10 seconds during a question
- See how many students have answered, then show results
- Resume or close open rooms from the home screen
- After a quiz, see how every student answered: a student-by-question table, plus a breakdown per question showing how many (and which) students picked each option

**Students**
- Create an account with the Student role
- Join by typing the room code, or by opening the link or scanning the QR code (the code is filled in)
- Answer on a phone-friendly screen
- See points, rank and the correct answer after each question
- Review all of their answers after the quiz: what they picked, the correct answer, points per question and time taken

**All users**
- Profile page to change name and password (email and role are read-only)

## How scoring works

- A wrong or missing answer scores 0.
- A correct answer scores 500 points plus up to 500 more depending on speed. Answering instantly gives about 1,000, and answering at the last second gives about 500.
- If every student has answered, the question ends early.

## Setup

### 1. Create a Firebase Realtime Database

1. Go to the [Firebase console](https://console.firebase.google.com) and create a project.
2. Open **Build → Realtime Database** and create a database.
3. Copy the database URL. It looks like `https://your-project-default-rtdb.firebaseio.com`.
4. Open the **Rules** tab and publish rules that allow the app to read and write. For a classroom demo:

   ```json
   {
     "rules": {
       ".read": true,
       ".write": true
     }
   }
   ```

   These rules are open to anyone who has the URL. See [Security notes](#security-notes) before using them with real data.

### 2. Point the app at your database

In `quizquest.html`, find this line and set your URL (no trailing slash):

```js
const FB_URL='https://your-project-default-rtdb.firebaseio.com';
```

If `FB_URL` is empty, the app runs in local mode. Rooms and accounts then work only across tabs of one browser, which is useful for testing.

### 3. Host the file

Phones need a web address to open the app, and the QR code needs one too. Any static host works, for example GitHub Pages, Netlify, Vercel or Firebase Hosting. Rename the file to `index.html` if your host expects that.

Opening the file directly from your computer works for testing on one device, but QR codes and links won't work for phones.

## Using it

1. The teacher signs up as **Teacher** and signs in.
2. The teacher chooses **Host or manage quizzes**, picks a quiz and clicks **Host**.
3. The teacher shares the 4-digit code, link or QR code.
4. Students sign up as **Student**, enter the code and click **Join room**.
5. The teacher clicks **Start quiz** once students have joined.
6. After each question the teacher clicks **Next question**, and at the end **Show final results**.
7. The teacher clicks **Close room** when finished.

## Where data is stored

| Data | Location |
| --- | --- |
| Accounts | Firebase, under `qq/qq-users` |
| Live rooms (players, answers, scores) | Firebase, under `qq/qq-room-<code>` |
| Quizzes | The teacher's own browser (`localStorage`) |
| Who is signed in | The browser tab (`sessionStorage`) |
| Last finished quiz (for answer review) | The user's own browser (`localStorage`, one copy per account) |

Because quizzes live in the teacher's browser, they are not shared between devices, and clearing browser data removes them. A room keeps its own copy of the quiz, so editing or deleting a quiz never disturbs a game in progress.

## Resetting data

- **Remove all accounts:** in the Firebase console, open **Realtime Database → Data**, expand `qq` and delete `qq-users`.
- **Remove old rooms:** delete the `qq-room-…` entries in the same place.
- **Reset starter quizzes:** clear the site's data in the teacher's browser.

## Security notes

This is a classroom-grade app, not a hardened one.

- Login is checked in the browser. Passwords are hashed with SHA-256 without a salt before being stored.
- With open database rules, anyone who knows the database URL can read or change the data, including account records.
- Teacher and student roles are chosen at sign-up and enforced only in the browser.
- Student names identify players inside a room. Two students with the same name would share a score.

For real security, move accounts and the password check to a server or use Firebase Authentication with proper database rules.

## Known limitations

- Quizzes are per browser, not synced.
- There is no way to change an account's email.
- There is no password reset by email. A teacher or admin has to delete the account record in Firebase.
- Question types are multiple choice and true/false only.
- Fonts and the QR code library load from Google Fonts and jsDelivr, so the app needs internet access. To make the QR code work offline, add an inlined copy of `qrcodejs` to the first script block.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| "Cannot reach the database" | The `FB_URL` value, and that the database rules are published |
| Badge says "Offline mode" | `FB_URL` is empty, so only tabs in one browser share data |
| QR code doesn't appear | The page is opened from a file or you are offline. Host it on a web address |
| "No room with that code" | The code is wrong, or the teacher closed the room |
| Student screen goes blank after the teacher closes the room | Expected. Students see "This room has closed" |

## Tech

Plain HTML, CSS and JavaScript. Firebase Realtime Database through its REST and Server-Sent Events API (no SDK). Fonts: Bricolage Grotesque and DM Sans. QR codes: `qrcodejs`.
