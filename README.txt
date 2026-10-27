MSI SMS - সেটআপ গাইড
=====================

ফাইল:
  index.html      -> ইউজার সাইট (Login/Register -> Agent Dashboard -> Client Panel, সব একটার ভেতরে)
  admin.html      -> অ্যাডমিন প্যানেল (আলাদা ফাইল)
  firestore.rules -> Firebase ডাটাবেস সিকিউরিটি রুলস (অবশ্যই বসাতে হবে)

আগে জেনে নাও
-------------
* GitHub Pages / সাধারণ হোস্টিং শুধু ফাইল দেখায়, ডাটাবেস রাখে না।
  রিয়েল-টাইম অ্যাকাউন্টের জন্য বিনামূল্যের Firebase (Google) ব্যবহার করা হয়েছে।
* Firebase কনফিগ না বসালে সাইট "DEMO MODE"-এ চলে: ডাটা শুধু ওই ব্রাউজারেই থাকে,
  অন্য ডিভাইসে দেখা যায় না। শুধু টেস্টের জন্য।

Firebase সেটআপ (একবারই, ~10 মিনিট)
-----------------------------------
1. https://console.firebase.google.com -> Add project
2. Build > Authentication > Get started > Sign-in method > "Email/Password" Enable
3. Build > Firestore Database > Create database (Production mode)
4. Firestore > Rules ট্যাবে firestore.rules ফাইলের সব লেখা পেস্ট করে Publish
5. (করা হয়ে গেছে - msi-sms-panel এর কনফিগ দুই ফাইলে বসানো আছে) Project settings (গিয়ার) > Your apps > Web (</>) > অ্যাপ রেজিস্টার করো
   -> যে firebaseConfig আসবে তার মান index.html ও admin.html দুই ফাইলের উপরের দিকে
      window.MSI_CONFIG = {...} এর ভেতরে বসাও (দুই ফাইলেই একই মান)।
6. Authentication > Settings > Authorized domains এ তোমার সাইটের ডোমেইন (যেমন username.github.io) যোগ করো।

হোস্টিং (GitHub Pages)
----------------------
index.html ও admin.html একটা রিপোজিটরির রুটে আপলোড করো -> Settings > Pages > Deploy from branch.
সাইট: https://username.github.io/repo/        অ্যাডমিন: https://username.github.io/repo/admin.html

প্রথম অ্যাডমিন (দুইভাবে, যেকোনো একটা)
-------------------------------------
ক) Firebase Console থেকে নিজে সেট করা (পাসওয়ার্ড তুমি ঠিক করবে):
   1. Authentication > Users > Add user
      Email:    admin@msi-sms.app      (admin-এর জায়গায় যে username চাও, পরে @msi-sms.app)
      Password: তোমার পছন্দের (কমপক্ষে ৮ অক্ষর)
      তৈরি হলে ওই ইউজারের "User UID" কপি করো।
   2. Firestore Database > Start collection > Collection ID: users
      Document ID: ওই User UID (পেস্ট করো)
      Fields (সব string, createdAt number):
        username  = admin          (ইমেইলের @ এর আগের অংশ, ছোট হাতের)
        name      = Admin
        email     = তোমার আসল ইমেইল
        whatsapp  = (ফাঁকা রাখতে পারো)
        country   = (ফাঁকা রাখতে পারো)
        role      = admin
        status    = active
        createdAt = 1700000000     (number)
   3. admin.html এ username: admin ও ওই পাসওয়ার্ড দিয়ে ঢোকো।
খ) অথবা admin.html এ "Create the first admin" (একবারই কাজ করে)।

ফ্লো
----
1. ইউজার index.html এ "Submit application" দিলে আবেদন pending হয়ে যায়; লগইন করা যায় না
2. অ্যাডমিন প্যানেলের Dashboard-এ "Pending Applications" এ সঙ্গে সঙ্গে আবেদন দেখা যায় (Agents পেজেও)
3. অ্যাডমিন Approve দিলে ইউজার username/পাসওয়ার্ড দিয়ে ঢুকতে পারে; Reject দিলে কখনোই পারে না
4. Active এজেন্ট Dashboard > Create User দিয়ে ক্লায়েন্ট বানায়
5. ক্লায়েন্ট একই লগইন পেজে ঢুকলে Client Panel দেখে
6. Suspend/Block করলে ওই ইউজার সঙ্গে সঙ্গে লগআউট হয়

কী কী আসল (রিয়েল-টাইম) আর কী এখনো শুধু ডিজাইন
-------------------------------------------------
আসল: রেজিস্ট্রেশন, লগইন/লগআউট, রোল (admin/agent/client), approve/suspend/block,
      এজেন্টের ক্লায়েন্ট লিস্ট ও তৈরি, অ্যাডমিনের Agents/Clients/Admins লিস্ট ও তৈরি,
      প্রোফাইল সেভ, পাসওয়ার্ড বদল।
এখনো ফাঁকা: SMS রেঞ্জ, নম্বর, CDR, বিলিং, প্রোভাইডার, পেআউট ইত্যাদি (এগুলোর জন্য আলাদা SMS প্রোভাইডার ব্যাকএন্ড লাগবে)।

নোট
---
* লগইনে username ব্যবহার হয় (ভেতরে username@msi-sms.app নামে Firebase ইমেইল বানানো হয়); আসল ইমেইল প্রোফাইলে সেভ থাকে।
* Forgot password এখনো কাজ করে না (আসল ইমেইল ভেরিফাই করা নেই বলে)।
* একই ব্রাউজারে admin.html ও index.html একই সেশন শেয়ার করে। অ্যাডমিন আর এজেন্ট একসাথে টেস্ট করতে আলাদা ব্রাউজার/প্রোফাইল ব্যবহার করো।
