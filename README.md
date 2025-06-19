// server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

// MongoDB connection
mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true, 
  useUnifiedTopology: true
}).then(() => console.log("MongoDB Connected"))
  .catch((err) => console.log(err));

// User schema
const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  password: String,
  referralCode: String,
  referredBy: String,
  referrals: { type: Number, default: 0 },
  balance: { type: Number, default: 0 },
});

const User = mongoose.model("User", userSchema);

const JWT_SECRET = process.env.JWT_SECRET || 'secret_key';

// Signup
app.post("/signup", async (req, res) => {
  const { name, email, password, referredBy } = req.body;
  const existingUser = await User.findOne({ email });
  if (existingUser) return res.status(400).json({ message: "User already exists" });

  const hashedPassword = await bcrypt.hash(password, 10);
  const referralCode = Math.random().toString(36).substring(2, 8);

  const newUser = new User({
    name,
    email,
    password: hashedPassword,
    referralCode,
    referredBy: referredBy || null,
  });

  await newUser.save();

  // Update referrer if exists
  if (referredBy) {
    const refUser = await User.findOne({ referralCode: referredBy });
    if (refUser) {
      refUser.referrals += 1;
      refUser.balance += 100;  // Add referral reward
      await refUser.save();
    }
  }

  res.json({ message: "Signup successful" });
});

// Login
app.post("/login", async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  if (!user) return res.status(400).json({ message: "User not found" });

  const isPasswordCorrect = await bcrypt.compare(password, user.password);
  if (!isPasswordCorrect) return res.status(400).json({ message: "Invalid password" });

  const token = jwt.sign({ id: user._id }, JWT_SECRET, { expiresIn: '1d' });
  res.json({ token, referralCode: user.referralCode, balance: user.balance, referrals: user.referrals });
});

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Referral Reward Website</title>
<style>
  body { font-family: Arial, sans-serif; margin: 20px; }
  header, main, footer { max-width: 600px; margin: auto; }
  .sidebar { float: right; width: 300px; margin-left: 20px; }
  .ad-container { margin: 20px 0; text-align: center; }
  .fixed-bottom-ad {
    position: fixed; bottom: 0; width: 100%; background: #fff; z-index: 999;
    box-shadow: 0 -2px 5px rgba(0,0,0,0.1);
  }
  .clearfix::after { content: ""; clear: both; display: table; }
</style>
</head>
<body>

<header>
  <h1>Referral Reward Website</h1>
  <!-- Header Ad -->
  <div class="ad-container">
    <script type="text/javascript" src="//pl26953656.profitableratecpm.com/16/c8/e1/16c8e1d709e258fd49a4d252feb92bfd.js"></script>
  </div>
</header>

<main class="clearfix">
  <div>
    <h2>Welcome, User!</h2>
    <p>Your referral code: <strong id="referralCode">ABC123</strong></p>
    <p>Total Referrals: <strong id="totalReferrals">0</strong></p>
    <p>Wallet Balance: <strong id="walletBalance">৳0</strong></p>
  </div>

  <!-- Sidebar Ad -->
  <aside class="sidebar">
    <div class="ad-container">
      <script type="text/javascript" src="//pl26953656.profitableratecpm.com/16/c8/e1/16c8e1d709e258fd49a4d252feb92bfd.js"></script>
    </div>
  </aside>
</main>

<section>
  <h3>Withdraw Section</h3>
  <p>You can withdraw your balance via Bkash, Nagad, or PayPal.</p>
  <!-- Withdraw Page Ad -->
  <div class="ad-container">
    <script type="text/javascript" src="//pl26953656.profitableratecpm.com/16/c8/e1/16c8e1d709e258fd49a4d252feb92bfd.js"></script>
  </div>
  <button onclick="alert('Withdrawal functionality coming soon')">Withdraw Now</button>
</section>

<section>
  <h3>Referral Page</h3>
  <p>Share your referral link and earn ৳100 per referral.</p>
  <input type="text" readonly value="https://yourwebsite.com/signup?ref=ABC123" style="width: 100%; padding: 8px;" />
  <!-- Referral Page Ad -->
  <div class="ad-container">
    <script type="text/javascript" src="//pl26953656.profitableratecpm.com/16/c8/e1/16c8e1d709e258fd49a4d252feb92bfd.js"></script>
  </div>
</section>

<!-- Fixed bottom ad for mobile -->
<div class="fixed-bottom-ad">
  <script type="text/javascript" src="//pl26953656.profitableratecpm.com/16/c8/e1/16c8e1d709e258fd49a4d252feb92bfd.js"></script>
</div>

<script>
  // Dummy data to simulate frontend dynamic data
  document.getElementById('referralCode').innerText = "xyz789";
  document.getElementById('totalReferrals').innerText = "5";
  document.getElementById('walletBalance').innerText = "৳500";
</script>

</body>
</html>MONGO_URI=your_mongodb_connection_string_here
JWT_SECRET=your_jwt_secret_key_here
PORT=5000
